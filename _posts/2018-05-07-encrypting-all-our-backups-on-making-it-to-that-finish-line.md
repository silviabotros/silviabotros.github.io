---
layout: post
title: " Encrypting All Our Backups: On Making It To That Finish Line"
date: 2018-05-07 20:21 -0700
---

_Note: This is a repost of a blog post I wrote for Sendgrid's blog in September of 2017_

### Encrypting All Our Backups: On Making It To That Finish Line

A year ago, Sendgrid was working hard towards SOC2 certification. Everyone was involved. There were stories on nearly every delivery team board with a SOC2 tag as we were all looking to be certified by the end of the third quarter. As you can imagine, being the person in charge of databases, there was definitely some work to do for that part of the stack.

On my task list for this business-wide endeavor was making sure that our backups were encrypted. Since my area of familiarity is DBA tools and knowing that Percona's xtrabackup already has support for encryption, it was predictable that I would go to that as the first attempt at this task.

A few important things were in my sights in testing this approach:
Obviously, the backup needed to be encrypted
The overhead to creating the backup needed to be known and acceptable
The overhead to decrypting the backup at recovery time needed to be known and acceptable

That meant I needed first to be able to track how long my backups take.

Sendgrid uses graphite for its infrastructure metrics and while the vast majority are sent via sensu, graphite is easy enough to send metrics directly via bash lines– very convenient since the backup scripts are in bash. Note that sending metrics at graphite directly is not super scalable but since these backups run at most once an hour, that was fine for my needs.

That part turned out to be relatively easy.

```
START=$(date +%s)
# do a bunch of backup related things
END=$(date  +%s)
runtime=$(($END-$START))
echo "databases.backups.$db_prefix.full  $runtime `date +%s`" | nc -w 1 $graphite_url 2003
```

To explain what happened in that last line, I send graphite the path of the metric I am sending (make sure that is unique), the metric value, then the current time in epoch format. Netcat is what I decided to use for simplicity, and I give it a timeout of 1 second because, otherwise, it will never exit. the `graphite url` is our DNS endpoint for graphite in the data center.

Now that I had a baseline to compare to, we were ready to start testing encryption methods.

Following [the detailed documentation](https://www.percona.com/doc/percona-xtrabackup/LATEST/innobackupex/encrypted_backups_innobackupex.html) from Percona on how to do this I started out by making a key. If you read that documentation page carefully, you may realize something.

This key is to be passed to the backup tool directly, and it is the same key that can decrypt the snapshot. That is called symmetric encryption and it is, by nature of that same key in both direction, less secure than asymmetric encryption. I decided to continue testing to see if simplicity still makes this a viable approach.
Tests with very small DBs, a few hundred MBs, were successful. The tool works as expected and documented but that was more of a functional test and the real question was "what is the size of the penalty of encryption on our larger DBs?" The more legacy instances at SendGrid had grown to sizes from 1-2 TB to a single 18 TB beast. What I was going to use for the small instances had to also be acceptably operational on the larger ones.

This is where testing and benchmarks got interesting.

My first test subject of a considerable size is a database we have that is 1 TB on disk. Very quickly I encountered an unexpected issue. With minimal encryption settings (1 thread, default chunk sizes) I saw the backups fail with this error:

```
xtrabackup: error: log block numbers mismatch:
xtrabackup: error: expected log block no. 243782669, but got no. 245879813 from the log file.
xtrabackup: error: it looks like InnoDB log has wrapped around before xtrabackup could process all records due to either log c
opying being too slow, or  log files being too small.
xtrabackup: Error: xtrabackup_copy_logfile() failed.
```

At the time these databases used 512MB as the transaction log file size, and this is a fairly busy cluster so those files were rotating almost every minute. Normally this would be noticeable in the DB performance but it was mostly masked by the wonder of solid state drives. Seems like not setting any parallel encryption threads (read: use one) means we spend so much time encrypting `.ibd` files that the innodb redo log rolls from under us making the backup break.

Let’s try this again with a number of encryption threads. As a first attempt, I tried with 50 threads. The trick here is to find the sweet spot of fast encryption without competing over CPU. I also increased the size of the `ib_logfiles` to 1 GB each.

This was a more successful test that I was happy to let brew overnight. For the first few nights, things seemed good. It was time to make a backup that doesn’t grow too much, but box load average during the backup process was definitely showing the added steps.

However, when I moved on to testing restores, I found that the restore process of the same backups, after adding encryption, had increased from 60 to 280 minutes. That meant a severe penalty to our promised recovery time in case of disaster and we needed to bring that back to a more reasonable timeframe.

This is where teamwork and simpler solutions to problems shined. One of our InfoSec team members decided to see if this solution can be simplified. So he did some more testing and came back with something simpler and more secure. I had not yet learned about [GPG2](https://linux.die.net/man/1/gpg2) and so this became a learning exercise for me as well.

The good thing about gpg2 is that it supports asymmetric encryption. The way that works is that we create a key pair where there is a private and public parts. The public part is used to encrypt any stream or file you decide to feed gpg2 and the private secret can be used to decrypt.

The change to our backup scripts to add encryption distilled to this. Some arguments are removed to make this easier to read:

```
innobackupex  $backup_path 2>>$log_file | pigz | /usr/local/bin/gpg2 --no-default-keyring --keyring /var/lib/mysql/backup_keyring.gpg --trust-model always --encrypt --digest-algo sha512 --cipher-algo aes256 --compress-level 0 --recipient REDACTED --recipient REDACTED | ssh $remote_user@$remote_server "cat > $remote_path/$date_path/FS_$backup_file" 2>>$local_err

```

On the other end, when restoring a backup, we simply have to make sure a secret key that is acceptable is in the host's keyring and then use this command:

```
/usr/bin/gpg2 --decrypt $backupfile.gpg > backup_file.xbs
```

Since I was new to gpg2 as well, this became a learning opportunity for me. Colin, our awesome InfoSec team member, continued to test backups and restores using gpg2 until he confirmed that the using gpg2 had multiple advantages to using xtrabackup's built in compression

* It was asymmetric
* Its secret management, for decryption, is relatively easily rotated (More on this below)
* It is tool agnostic, which means any other kind of backup that isn't using xtrabackup could use the same method
* It was using multiple cores on decryption which gave us better restore times

One place where I still see room for improvement is how we handle the secret that can decrypt these files. At the moment we have them in our enterprise password management solution, but getting it from there then using it to test backups is a manual process. Next in our plan is to implement [Vault by Hashicorp](https://www.vaultproject.io) and use that to seamlessly, on the designated hosts, to pull the secret key for decryption, then remove it from the local ring so it is easily available for automated tests and still protected.

Ultimately, we got all of our database backups to comply with our SOC2 needs in time without sacrificing backup performance or disaster recovery SLA. I had lots of fun working on this assignment with our InfoSec team and came out of it learning new tools. Plus it is always nice when the simplest solution ends up being the most suitable for one's task.

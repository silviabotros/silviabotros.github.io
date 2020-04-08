---
layout: post
title: High Performance MySQL
date: '2020-04-08 00:00:00'
tags:
- announcements
---
There are things that one never expects to get to do. I have talked about this on Twitter before but I still remember being a new immigrant who took a chance and quit college to come to the US, start over and merely dreamed of working tech.

I have also talked about how I started in tech after college, how I never planned to be a DBA and it was all a 'happy' accident and a necessity of the time that showed me something I didn't know I was passionate about doing well. Back then, the de-facto book on how to run MySQL in a production setting besides Oracle's own docs was a book called "High Performance MySQL, 2nd edition". MySQL 5.1 was brand new. InnoDB was becoming a more mature engine, people were starting to look at MySQL as a much more serious datastore technology to use in companies looking to rely on open source software to build their business.

That book helped teach me so many things back then. What is a buffer pool? How does one optimize it? How does a query run? How do I understand a query plan? What datatypes cause more disk activity and how do I optimize all these buffers? 

Since then there has been a third edition that added lots of new content about 5.5 (which was then the 'GA' version). 

The third edition came out in 2012 and since then, there has been 3 major releases of MySQL and so many improvements, new frameworks and tools in the MySQL community. It is time for a 4th edition that covers not just new core features in MySQL but a new look at _how_ to run MySQL in a high performance, high demand environment. Not just as islands of database instances, but as a database platform that serves the business and the delivery teams. How to manage compliance needs. How to manage schema changes. How to scale not just the database instances but also how to choose the right access layer for your application layer. When to recognize that you need to go polyglot and branch out of relational databases altogether.

I will be working on creating the new version of the book that got me hooked on scaling databases all those years ago. Planned for release in the fall of next year. I hope it does for many what the second edition did for me years ago. This book was the start of an amazing journey for me and I am thrilled to help bring the next edition of it to up and coming database engineers. 
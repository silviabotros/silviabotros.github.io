---
layout: post
title: How to choose a data store for the new shiny thing
date: 2018-05-11 21:43 -0700
---
_Note: This is post first appeared as part of the SysAdvent series of December of 2017_

Databases can be hard. You know what's harder? Choosing one in the first place. This is challenging whether you are in a new company that is still finding its product/market fit or in a company that has found its audience and is simply expanding the product offering. When building a new thing, one of the very first parts of that design process is what data stores should we use and should that be a single or a plural? Should we use relational stores or do we need to pick a key value store? What about time options? Should we also sprinkle in some distributed log replay? So. Many. Options...

I will try in this article to describe a process that will hopefully guide that decision and, where applicable, explain how the size and maturity of your organization can impact this decision.

## Baseline requirements

Data is the lifeblood of any product. Even if we’re planning in the design to use more bleeding edge technology to store the application state (because MySQL or Postgres aren’t “cool” anymore), whatever we choose is still a data store and hence requires that we apply rigor when making our selection. The important thing to remember is that nothing is for free. All data stores come with compromises and if you are not being explicit about what compromises you are taking as a business risk, you will be taking unknown risk that will show itself at the worst possible time.

Your product manager is unlikely to know or even need to care what you use for your data store but they will drive the needs that shrink the option list. Sometimes even that needs some nudging by the development team, though. Here is a list of things you need to ask the product team to help drive your options:

* Growth rate - How is the data itself or the access to it expected to change over time?
* How will the billing team use this new data?
* How will the ETL team use this data?
* What accuracy/consistency requirements are expected for this new feature?
 * What time span for that consistency is acceptable? is post processing correction acceptable?

## Find the context that is not said

The choice of the data store is not a choice reserved for the DBA or the Ops team or even just the engineer writing the code. For an already mature organization with a known addressable market, the requirements that feed this decision need to come from across the organization. If the requirements from the product team fit a dozen data stores, how do you determine requirements not explicitly called out? You need to surface unspoken requirements as soon as possible because it is the road to failed expectations down the line. A lot of implied things can make you fail in this 'too many choices' trap. This includes but is not limited to:

* Incomplete feature lists
* Performance requirements that are not explicitly listed
* Consistency needs that are assumed
* Growth rate that is not specified
* Billing or ETL query needs that aren't yet available/known

These are all possible ways that can leave an engineering team spinning their wheels too long vetting a long list of data store choices simply because the explicit criteria they are working with are too permissive or incomplete.

For more ‘greenfield’ products, as i mentioned before, your goal is flexibility. So a more general purpose, known quality, data store will help you to get closer to a deliverable, with the knowledge that down the line, you may need to move to a datastore that is more amenable to your new scale.

## Make your list

It is time to filter potential solutions by your list of requirements. The resulting list needs to be not more than a handful of possible data stores. If the list of potential databases you can use is more than that then your requirements are too permissive and you need to go back and find out more information.

For younger, less mature companies, data store requirements is the area of the most unknowns. You are possibly building a new thing that no one offers just yet and so things like total addressable market size and growth rate may be relatively unknown and hard to quantify. In this case what you need is to not constrain yourself too early in the lifetime of your new company by using a one trick pony datastore. Yes, at some point your data will grow in new and unexpected ways but what you need right now is flexibility as you try to find your market niche and learn what the growth of your data will look like and what specific scalability features will become crucial to your growth.

If you are a larger company with a growing number of paying customers, your task here is to shrink the option list to preferably data stores you already have and maintain. When you already have a lot of paying customers, the risk of adding new data stores that your team is not familiar with becomes higher and, depending on the context of the data, simply unacceptable. Another thing to keep in mind is what tooling already exists for data stores and what would adopting a new one mean as far as up front work your team has to do. Configuration management, backup scripts, data recovery scripts, new monitoring checks, new dashboards to build and get familiar with. The list of operational cost of a new data store, regardless of risk, is not trivial.
## Choose your poison

So here is a badly kept secret that DBAs hold on to. Databases are all terrible at something. There is even a whole theorem about that. Not just databases in the traditional sense, but any tech that stores state will be horrible in a way unique to how you use it. That is just a fact of life that you better internalize now. No, I am not saying you should avoid using any of these technologies, I am saying keep your expectations sane and know that YOU and only you and your team ultimately own delivering on the promises you make.

What does this mean in non abstract terms? Once you have a solid idea what data stores are going to be part of what you are building, you should start by knowing the weaknesses of these data stores. These weaknesses include but are not limited to:

* Does this datastore work well under scan queries?
* Does this datastore rely on a gossip protocol for data replication? if so, how does it handle network partitions? How much data is involved in that gossip?
* Does this datastore have a single point of failure?
* How mature are the drivers in the community to talk to it or do you need to roll your own?
* This list can be *huge*

Thinking through the weaknesses of the potential solutions still on your list should knock more options off the list. This is now reality meeting the lofty promises of tech.

## Spreadsheet and Bake off!

Once your list of choices is down to a small handful, it is time to put them all in a spreadsheet and start digging a little deeper. You need a pros column and a cons column and at this point, you will need to spend some time in each database documentation to find out nitty gritty details on how to do certain tasks. If this is data you expect to have a large growth rate, you need to know which of these options is easier to scale out. If this is a feature that does a lot of fuzzy search, you need to know which datastore can handle scans or searching through a large number of rows better and with what design. The target at this stage is to whittle down the list to ideally 2 or 3 options via documentation alone because if this new feature is critical enough to the company success, you will have to benchmark all three.

Why benchmark you say? Because no 2 companies use the same datastore the same way. Because sometimes documentation implies caveats that only gets exposed in other people's war stories. Because no one owns the stability, the reliability and the predictability of this datastore but you.

Design your benchmark in advance. Ideally, you set up a full instance of the datastores in your list with production level specifications and produce test data that is not too small to make load testing useless. Make sure to not only benchmark for 'normal load' but also to test out some failure scenarios. The hope is that through the benchmark, you can find out any caveats that are severe enough to cause you to revisit the option list now instead of later when all the code is written and you are now at the fire drill phase with a lot of time and effort committed to the choice you made.

## Document your choice

No matter what you do, you must document and broadcast internally the method by which you reached your choice and the alternatives that were investigated on the route to that decision. Presuming there is an overarching architecture blueprint of how this new feature will be created and all its components, you make sure to create a section dedicated to the datastore powering this new feature with links to all the benchmarks done to reach the decision the team came to. This is not just for the benefit of future new hires but also for your team's benefit in the present. A document that people can asynchronously read and develop opinions on provides a way to keep the decision process transparent, grow a sense of best intent among the team members and can bring in criticism from perspectives you didn't foresee.


## Wrap up

These steps are not only going to lead to data-informed decisions when growing the business offering, but will also lead to a robust infrastructure and a more disciplined approach to when and where you use an ever growing field of technologies to provide value to your paying customers.

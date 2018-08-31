---
layout: post
title: Thoughts on the new State of Devops report
date: ‘2018-08-31 10:46:00’
---
This week, the latest [State of DevOps][1] report by the team at [DORA][2] was released. There is a lot of conversations in various social media platforms on what makes an engineering team ‘successful’ and how to increase feature velocity in tech companies and the DORA report does not just rely on anecdotes but surveys the field using rigorous scientific methods and statistical constructs to back or debunk common practices with real data. Because of the strong scientific backbone of their findings, I have been following the report for the last few years. If you have not, the book [Accelerate][3] is a great way to learn the findings of the previous four years. 

### Database management as a part of team performance
One of the new things in this year's report is expanding behaviors by high performing teams to more than just ‘development’ practices. This year expands the view on high performing engineering organizations to how these organizations handle database management. 

> We discovered that good communication and comprehensive configuration management that includes the database matter. Teams that do well at continuous delivery store database changes as scripts in version control and manage these changes in the same way as production application changes.
I have [written in the past][4] about breaking DBA team barriers, about the DBA stereotype becoming stale and not really helpful to overall organization goals but I admit my opinions had been till now both anecdotal and strongly biased by my experiences in my own career. In my conversations with other organizations it seemed the more common practice was still to either assign a team (or worse, one person) the task of care and feeding of the databases powering the product and to keep the database changes needed for feature releases as last and completely manual because 'databases are scary'. 

So when the work by DORA was finally released I was delighted to see that this year included data and analysis on the correlation between team performance and how to handle database change management. And it seemed my gut feeling on where we should be going as an industry was actually supported by the data. High performing teams maintained as much as they can about their databases in version control just as they do with feature code. Schema definitions, scripts that do administrative tasks and even code that applies configuration management to the databases were all in version control in high performing and elite teams. 

This practice provides a track record of changes and a way to apply peer review to what is being changed. It also allows for treating the database as part of the larger deploy process. It becomes easier to at least provide visibility to team into when and at what stage is their requested change which allows for less communication overhead when planning deploy timelines. With everything but the data itself in version control, teams can also start applying automation and provide feature teams with autonomy over their data-stores which is also greatly helps team overall velocity. 

### The role of outsourcing
> Analysis shows that low-performing teams are 3.9 times more likely to use functional outsourcing (overall) than elite performance teams, and 3.2 times more likely to use outsourcing of any of the following functions: application development, IT operations work, or testing and QA. This suggests that outsourcing by function is rarely adopted by elite performers.
The report also makes an assertive point about the effort to outsource development by lower performing teams. It even goes as much as calling them ‘misguided performers ’. This, to me, is directed at efforts to just ‘throw money at’ team velocity without much thought into how to use such an engagement. If you are part of a decision on whether to outsource or not, it is very important from the outset to have some internal agreement on things such as
* What is the expected benefit 
* What is the definition of success 
* How to measure said success, what agreed upon metrics are we using later to reevaluate 

Now, I have nothing against consultants. I have worked with some of the best consultant DBAs in the field and have also learned a lot from them. My team has also in the past employed contractors from so called “DevOps for hire” companies. But I have also seen consultant engagements go very very terribly in both cost and morale hits to the in house engineers and the crucial difference has been how we incorporate these consultants into the team just as one would a new full time employee. Hiring consultants to become a new silo away from the rest of the engineering organization to solve problems in a vacuum never, ever, works and somehow executives in many companies still think that is a sane thing to do with the company resources. The Accelerate book goes into more detail into how to use external resource successfully towards team velocity and this year’s report solidifies this with more recent data showing low performing teams more likely to fall in the outsourcing trap as a supposed ‘quick fix’.
 
### Cloud infrastructure
> How you implement cloud matters
A few years ago, a lot of companies still saw using platforms like AWS or Azure as a way to have more outages and spend a lot more money on operating costs of running their product. Now we seem to have swung the other way since these platforms have become a lot more stable but there still seems to be a presumption that using AWS will magically increase feature release velocity but that it is also a quick thing a company can just swing in a quarter. 

It is important to internalize some important facts about running infrastructure in cloud platforms
* Companies moving to a cloud platform these days look at it as a way to increase feature velocity. That is clearly stated as a finding from the report data and it is an improvement in the field’s understanding of what advantages running in the cloud provides
* Now that we established _why_ you would move your infrastructure, you cannot, *cannot* forklift what you already built to a cloud provider and magically expect velocity or even cheaper costs. In fact, it will most definitely cause the opposite of both without deliberate thought into your architecture design. You have to approach a move from on premise to cloud to include some thought into re-architecture of what you are moving. Yes including databases. 
* Use the advantages in the cloud platform you are moving to. Use managed services wherever possible to refocus your engineers’ time to only focus on what makes your product better. 

Does this mean that you will no longer need ops people or DBEs? Absolutely not. But the roles of operations engineers and database engineers, like everything else, change over time. This is something that needs to be a separate and longer post but if your team transforms the role of a DBE away from manually running databases and towards helping scale your data architecture, you will find that using managed datastores will actually help free those engineers to provide better value for your business and make up for more than the cost of Aurora or DynamoDB.

There are so many other gems that make one go “aha!” in the report on what high performing engineering teams actually *do* vs what they *think* makes them successful. It is pretty great seeing these reports annually being supported by solid science and I very much appreciate the work [Dr. Nicole Forsgren][5], [Jez Humble][6] and [Gene Kim][7] put into it year after year. 


[1]:	https://cloudplatformonline.com/2018-state-of-devops.html
[2]:	https://devops-research.com
[3]:	https://www.amazon.com/Accelerate-Software-Performing-Technology-Organizations-ebook/dp/B07B9F83WM/ref=sr_1_1?ie=UTF8&qid=1535649522&sr=8-1&keywords=accelerate
[4]:	http://sysadvent.blogspot.com/2016/12/day-2-dbas-priesthood-no-more.html
[5]:	https://twitter.com/nicolefv
[6]:	https://twitter.com/jezhumble
[7]:	https://twitter.com/RealGeneKim

---
layout: blog_post
title: Okta Software Engineering Design Principles
author: jon_todd
tags: [software_engineering, architecture, design_principles]
---

Okta has been an agile development shop since the beginning. One important
aspect of being agile is enabling a mix of bottom-up and top-down decision
making where high level vision and strategy is clearly communicated to enable
teams to autonomously deliver value while also feeding back learnings from the
trenches to inform the high level goals.[^the-knowledge-creating-company] This
document collects much of the tacit engineering design principles we've used to
guide development at Okta and continues to evolve as we experiment and learn.

## 1. Create User Value 

First and foremost, writing software is about creating value for users. This
seems pretty straight forward, but as systems evolve and become more complex we
start to introduce more abstraction and layering which brings us further away
from the concrete problem we're trying to solve. It's important to keep in mind
the reason you're writing software in the first place and use the understanding
of your audience to inform priority.

At Okta, our entire company is aligned on this principle because our #1 core
value is [customer
success](https://www.okta.com/customers/focus-on-customer-success.html).  In
practice this means there's almost always a bunch of customers eager to beta a
new feature we're working on. We collaborate closely with customers while we're
building features and are able to get their feedback as we iterate and get
changes out the door in weekly sprints.

![xkcd - pass the salt](http://imgs.xkcd.com/comics/the_general_problem.png)

## 2. Keep it Simple

>   Everything should be made as simple as possible, but no simpler — Albert
>   Einstein 

This is a truism that's been around for ages and it goes hand in hand with the
first principle.  If it doesn’t add value to your users now, [you ain’t gonna
need it - YAGNI](http://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)!

We've all encountered overly complex code where it's nearly impossible to reason
about what it does. Part of this confusion comes from the fact that it's
generally [easier to read code than to write
it](http://www.joelonsoftware.com/articles/fog0000000069.html) but beyond 
that, there's clearly fundamental qualities of some code which makes it
more intuitive than other code. There's a lot of prior are on this topic but a
great place to start is the book [Clean
Code](http://books.google.com/books?id=dwSfGQAACAAJ) by Robert C. Martin aka
Uncle Bob. The book breaks down the various qualities of code which make it
intuitive and provides a framework for reasoning about the quality of code. 

Here are some guiding principles about writing clean code we use in practice
which are also covered in the book.

* Clean code makes intent clear, use comments when code isn't expressive enough
* Clean code can be read and enhanced by others (or the author after a few
  years)
* Clean code provides one way, rather than many, to do a particular task
* Clean code is idomatic
* Clean code is broken into pieces which each do one thing and do it well

At the end of the day you can read all the books you want on writing clean code
but, like any craft, there's no substitute for experience. Every engineer is
constantly honing their skills and at Okta, we rely heavily on code reviews and pair
programming to help hone each other's skills. 

![wtfs per minute](/assets/img/2015-05-08-software-engineering-design-principles-code_quality_wtfs_per_minute.jpg)


## 3. Know Thy-Service With Data 

In the world of "big data" this point needs little explanation. We collect a
massive amount of operational data about our systems to:

* Monitor health
* Monitor performance
* Debug issues
* Audit security
* Make decisions

With every new feature we add, developers are responsible for ensuring that
their designs provide visibility into these dimensions. In order to make this as
simple as possible we've invested in the following: 

* Runtime logging control filtering by level, class, tenant, user 
* Creation of dashboards and alerts is self-service 
* Every developer has access to metrics and anonymous unstructured data
* Request ID generated at edge is passed along at every layer of stack for
  correlation
* Automation platform for common operational tasks like taking threaddumps

Some of the technologies we use to gain visibility are: PagerDuty, RedShift,
Zabbix, ThousandEyes, Boundary, Pingdom, App Dynamics, Splunk, ELK, S3.

## 4. Make Failure Cheap 

Every software system will experience failures and all code has bugs. While we
constantly work at having fewer, it's unrealistic to assume they won't occur. So
in addition to investing in prevention, we invest in making failure cheap.

When it comes to the cost of failure, it becomes significantly more expensive to
fix a failure the further out on the development time line going from
requirements gathering through development &amp; testing out to running in
production.[^agile-cost-curve] 

![cost curve of development](/assets/img/2015-05-08-software-engineering-design-principles-agile-cost-curve.png)

One fundamental we take from both Agile and XP is to invest in pushing failure
as early in the development time line as possible. We mitigate failures from
poor requirements gathering by iterating quickly with the customer as described in
Principle 1. Once we get to design and development we make failure cheap through: 

* Design reviews with stake holders ahead of writing code 
* TDD - developers write all tests for their code, test isn't a separate phase
  from development 
* Keeping master stable - check-in to master is gated by passing all unit,
  functional and UI tests 
* Developers can trigger CI on any topic branch, CI is massively parallelized
  over a cloud of fast machines

Since our testing phase is done during development the next phase is production
deployments. At this phase we reduce the cost of failure by:

* Hiding beta features behind flags in the code
* Incremental rollout first to test accounts and then in batches of customers
* Automated deployment process
* Code and infrastructure is forward and backward compatible allowing
  rollback
* Health check and automatically remove down nodes
* Return a degraded / read-only response over nothing at all

> An escalator can never break; it can only becomr stairs -- Mitch Hedberg

## 5. Automate Everything

This one is pretty self explanatory. If you perform a task routinely, it should
be automated. These are automation principles we follow:

* Automate every aspect of the deployment including long running db migrations
* All artifacts are immutable and versioned
* All code modules get depenendcies automatically from central artifact server
* Creation of base images and provisioning of new hardware is automated
* All forms of testing are automated 
* Development environment setup is automated

Tools we use:

* AWS - Automated provisioning of hardware 
* Chef - Configuration managment
* Ansible - Automated deployment orchestration
* Jenkins - Continuous integration
* Gearman - To get Jenkins to scale
* Docker - Containerizing services

## 6. With Performance, Less is More

We find especially with performance, there are typically huge wins to be had in
up front design decisions which may come at very little to no cost. Our design
mantras for performance are:

1. Don't do it
2. Do it, but don't do it again
3. Do it less
4. Do it later
5. Do it when they're not looking
6. Do it concurrently
7. Do it cheaper

In practice we implement a number of strategies to limit risk to poorly
performing code:

* Major new features and performance tunings live behind feature flags allowing
  slow rollout and tuning in real life environment
* Chunk everything that scales on order of N. When N is controlled by customer
  enforce limits and design for infinity.
* Slow query and frequent query monitoring to detect poor access patterns

<img style="max-width:300px" src="/assets/img/2015-05-08-software-engineering-design-principles-more_is_less.jpg" alt="if less is more, does that mean more is less?">

### Reference

[^the-knowledge-creating-company]: Ikujiro Nonaka, and Hirotaka Takeuchi. The Knowledge Creating Company. Oxford University Press, 1995. Print. https://books.google.com/books/about/The_Knowledge_creating_Company.html?id=B-qxrPaU1-MC
[^agile-cost-curve]: Scott Ambler. Examining the Agile Cost of Change Curve. Website. http://www.agilemodeling.com/essays/costOfChange.htm

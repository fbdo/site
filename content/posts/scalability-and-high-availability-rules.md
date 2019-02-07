+++ 
draft = false
date = 2019-01-27T20:53:16+01:00
title = "Scalability and high availability rules"
slug = "" 
tags = ["scalability", "high availability", "cloud"]
categories = ["availability","scalability"]
+++

# Objectives
This page describes a set of rules and a methodology that, once applied to a system, can help you evaluate how well your system is regarding
scalability and availability, can help you map your weaknesses and plan actions to improve them.
In this context, scalability, availability and performance measured by low latency and high throughput are highly intertwined, as a high available
system need to scale to be available regardless the load, a system with a high throughput scale more easily than one with a low throughput, and
as a consequence is more available.

# Vocabulary

## Scalability

Citing the Wikipedia:

> Scalability is the capability of a system, network, or process to handle a growing amount of work, or its potential to be enlarged
to accommodate that growth. For example, a system is considered scalable if it is capable of increasing its total output under
an increased load when resources (typically hardware) are added.

## Availability
Citing the Wikipedia:

> The degree to which a system, subsystem or equipment is in a specified operable and committable state at the start of a
mission, when the mission is called for at an unknown, i.e. a random, time. Simply put, availability is the proportion of time a
system is in a functioning condition. This is often described as a mission capable rate. Mathematically, this is expressed as
100% minus unavailability.

The unavailability U of a system can be mathematically expressed as:

U = MTTR/(MTTR+MTTF)

Where:

MTTR is the Mean Time To Recover

MTTF is the Mean Time To Fail.

As an example, let's suppose you have an application running in one machine in your datacenter, and your hardware provider told you the MTTF of this machine is 5 years. To recover from this failure, your application takes 30s to come alive again, counting the time to detect the failure, the time to instantiate a new virtual machine in another machine in your datacenter, and the time you application takes to startup (this is your MTTR).

So your availability A is:

A = 1 - U = 1 - (30/(30 + 5*365*24*60*60)) = 1 - (30/(30 + 157680000)) = 0,999999809741284

Or in another words, you can expect approximately 6 hours of outage per year for your application.

An interesting fact that affects the availability is the probability one machine used by your system will fail is inversely proportional to the number of machine you use. So, in the example above, given a datacenter with 1000 machines (a relatively small datacenter) where each one has a MTTF of 5 years, you will have a machine failing more or less every 2 days.

# Rules Review and Priorization
Below you can see some rules/best practices to achieve infinite scalability. Every rule not obeyed implies in a risk, and this risk can be defined as:

Risk = Probability * Impact

Where:

Impact = %Impact * (Downtime + %DataLoss + ResponseTimeImpact)

%Impact = %CustomersImpacted or %FunctionalityImpacted

By those terms it's possible to define a prioritization table where for each rule, the priority is:

Priority = f(Risk Reduction, Cost of Solution)

The Risk Reduction and Cost of Solution are very specific for a organization/department/team. To make easier to evaluate a system, it's simpler to give discrete values as High, Medium and Low for both variables, ariving to the following commutative table:

<center>

| Risk/Cost  | Low           | Medium   | High     |
| ---------- |:-------------:| --------:| --------:|
| Low        | Do it         | Evaluate | Don't do |
| Medium     | Do it         | Do it    | Evaluate |
| High       | Do it         | Do it    | Evaluate |

</center>

# Rules

Bellow you can find a list of rules that once applied consistently to your project, Your project will enjoy scalability and high availability. 

We can split the process of activing high availabiliity in 3 dimensions: a horizontal scaling, splitting by function, service or resource and lookup/formulaic splits, arriving in the following cube representing the 3 variables relationship regarding the ideal near infinite scale:

<center>

![Scalability cube](/images/scalability-and-high-availability-rules/scalability_cube.png)

</center>

The rules are organized in main sections, and every rules has a number, a short description and a resulting benefit/priority to the project. High priority rules normally are the ones with low cost and high risk, while low priority rules are the ones with high cost and low risk. The classification given by this document should be used as a starting point and adapted according to the project context.

The non-complaince to one of the rules normally means the project is accepting a risk in terms of scalability, and as such it's needed a analysis cost/benefit. For real-time/critical systems the risk imposed by a low priority rule can be unnaceptable.

## Reduce the Equation Rule

### Don't overengineer the solution (High-2)

Don't build a solution for a problem bigger than the requirement.

Don't do unnecessary work. Execute something only once.

Make it simple to other architects to understand.

Test: present the solution to a team of pairs

### Design scale into the solution (Medium-3)

D-I-D process:

Design for 20x capacity

Implement for 3x capacity

Deploy for 1.5x capacity

Design have a low cost, design for 20x to infinity.

Implement considering 3x to 20x the capacity, consider the greatest bottleneck.

### Simplify the solution 3 times over (Medium-3)

Simplify scope using the Pareto principle. 80% of the revenue is achieved by 20% of the work.

Simplify design by thinking about cost effectiveness and scalability.

Do the same work faster and easier. Consider to join services together
instead of multiple network trips to do the same job.

Simplify implementation by leveraging the experience of other, reuse open source, libraries from other departments, buy existing
solutions or check for descriptions of a similar solution.

### Reduce DNS lookups (Medium-3)

Large number of user requests affects the user experience. DNS lookups contribute to slow down the application.

### Reduce number of objects travelling the network (Medium-3)
Check where your network is slower/faster and aggregate calls appropriately. The datacenter network is usually faster, so your protocol can be chatier here, but the last net hop to the user is usually slower.


## Distribute your Work

### Design to clone or replicate things (High-2)

Duplicate services/databases to spread the load.

Clone services and implement load balancers.

Database normalization and ACID properties make them difficult to split.

High read to write ratio allow read-only copies.

A layer of caching is a recommended first step.

### Design to split different things (a.k.a domain-based microservices) (Medium-3)

Split systems in nouns or verbs or both.

Separation using verbs: signup, login, search, browse, view, add-to-cart, purchase.

Separation using nouns: product catalog, product inventory, user account information, marketing information.

### Design to split similar things (a.k.a sharding) (Medium-3)

Identify a characteristic of your data and split or partition both data and services based on that attribute.

It's possible to use some business related sharding, like per geographic location, by company size, by revenue. Or a simple modulus or
hash function by user id.

When data for the same group of users is grouped together, bigger the chance for a cache hit.#3 Design to Scale Out Horizontally

### Design your solution to scale out, not just up (High-2)

Allows for fast scale of transactions at the cost of duplicated data and functionality.

Don't get caught in the trap of expecting to scale up to find out that you've run out of a faster and larger system to purchase.

### Use commodity systems (High-2)

Many smaller systems don't suffer from inefficient sheduling algorithms, memory bus access speeds, structural harzards, data harzards.

Stay way from very large systems.

Allows for fast, cost-effective growth.

### Scale out your data centers (Medium-3)

Forget about hot-cold datacenters, they have more or less the same cost of operation and there's a higher risk of failure when you finally need it.

Slice data between your datacenters to keep high availability. Spare some machine resource to distribute the load in case of failure.

E.g. given three datacenters:

### Design to leverage the cloud (Low-4)

Provision of hardware in the cloud takes few minutes.

Design to leverage virtualization in all sites and grow in the cloud to meet unexpected spiky demand.

BUT be carefull to not depend on specific cloud services to avoid a supplier lock-in.

## Use the Right Tool

### Use the right database, and use it properly (High-2)

Don't use a SQL database where a noSQL database would work better.

Use the filesystem when possible.

Relationships give extra flexibility and guarantees but also extra costs to scale.

### Reduce the numbers of firewalls (High-2)

Use firewalls where they reduce risks, but be aware they cause issues with scalability and availability.

Don't use for low value content.

### Actively use log files (Medium-3)

Use the application log to diagnose and prevent problems

Implement a correlation ID to correlate logs from different services

Put a process in place to monitor log files and force people to take action on issues

Aggregate them, use log servers on out-of-band network

Archive and purge as the value decreases

Log in an asynchronous fashion

### Use Non-blocking I/O as much as possible (High-2)

Serial, blocking I/O were the standard but are too expensive in high throuput systems

Most of the Java Web servers use blocking I/O

All the JDBC drivers use blocking I/O

New options are reactive frameworks

## Don't Duplicate your Work
    
### Don't check your own work (Low-4)

Don't validate data you just wrote

Keep data locally if you will immeadiately need it

### Stop redirecting traffic (Medium-3)

Traffic redirections on the client side cause unecessary resource consumption.

Use server side redirection features instead

### Relax temporal constraints (Very High-1)

Relax your temporal constraints, it's hard to keep them and they are hardly necessary.

## Use Cache Aggressively

### Leverage Content Delivery Networks (Medium-3)

Wherever possible move static, semi-static or content that are not so dynamic to CDNs.

### Use Expire Headers (Medium-3)

Use the expire headers to inform the client or the CDN when a content should be considered stale.

### Leverage application caches (Medium-3)

Split the application to increase the cache hits by business terms, e.g. 80% of the requests occur against 20% of your inventory.

Premium users on their on servers

### Leverage object caches (Very High-1)

On the application layer use caches, normally before your datastore

Monitor the cache hit ratio, it should be around 80%.

### Put object caches on their own "tier" (High-2)

You will safe CPU and memory from the application layer

## Learn from your Mistakes

### Learn aggressively (High-2)

Do A/B testing

Employ a postmortem process:

Phase 1: Timeline

Phase 2: Issue identification

Phase 3: State actions, use SMART principle (every action should be specific, measurable, attainable, realistic and timely).

Every action need to have a single owner, even if executed by a group

Hypothesize failure

No failure should stop in "server A died", we need to find why the monitoring didn't show the problem earlier, why the reaction was slow, why it took so long to recover, why the database is not split to reduce impact, etc...

Keep a log of failures and review them periodically to identify patterns.

### Don't rely on QAs to find mistakes (High-2)

The QA know-how should be used to define the quality process and to keep memory of mistakes and recurring problems, help the
engineering team to find root causes in the process

Use TDD aggressively. Define and monitor a desired code coverage.

### Failing to design for rollbacks is designing for failure (Very High-1)

Ensure all releases have the ability to rollback, practice it on QA

The cost is low, the risk is high

Only additive database changes

Database changes should be scripted and testes, including rollback scripts

Use restricted SQL queries, don't use SELECT * and add column names to UPDATE statements

Avoid semantic changes of data

The application should have a way to activate features for a limited number of users

## Database Rules

### Remove business intelligence from transaction processing (High-2)

Don't use stored procedures

Easier to migrate to NoSQL down the road

Easier to scale

Business process should not be tied to the product

### Be aware of costly relationships (Medium-3)

Relationships defines how costly is to retrieve data and how difficult would be to split the database

New queries should be analyzed by DBAs

To scale we may reduce normal forms

A query join can be replaced by a view, a materialized view, a summary table to preprocess the query, or use the application to do the
join

### Use the right type of database lock (Very High-1)

Be aware of implicit locks and explicit ones

Types of locks: implicit, explicit, row, page, extent, table, database
Monitor your database to check if the correct lock is being deployed
Employ read-only database nodes, denormalize and split databases

### Don't use multiphase commits (High-2)
Avoid 2 and 3 phase commits, they slowdown your application proportionally to the number of nodes on your cluster

### Don't use SELECT FOR UPDATE (High-2)
It adds locks and can slowdown transactions

Some databases bave by default cursors FOR UPDATE, check your documentation

### Don't select everything with SELECT * (Very High-1)

More probabble to break things if the table structure changes, and it will transfer unnecessary data

## Design for Fault Tolerance and Graceful Failure

### Design using fault-isolative "swim lanes" (Medium-3)

Create fault isolation domains

Nothing is shared between swim lanes, including networkDatabases and servers should never be shared

No synchronous calls exists between swim lanes

Limit asynchronous calls

On asynchronous calls, you should be able to just ignore or no process now the response

On a virtualized infra, keep all virtualized servers of a physical server in the same swim lane

Tools and monitoring need to be fault tolerant as well, you don't want to lose them when you require them the most

### Never trust single point of failure (High-2)

Strive for active/active configurations

Everything fails

### Avoid putting systems in series (Medium-3)

Elements in series have a multiplicative effect of failure

Don't ignore network elements

### Ensure you can active/deactivate new features (Low-4)

Use feature toggles and circuit breakers

Implement when the cost is less than the risk = likehood * impact

## Avoid or Distribute State

### Strive for statelessness (Medium-3)

Measure the need for state in terms of revenue and increased transactions

Session and state cost money

### Maintain session data in the client when possible (High-2)

Careful with sidejacking, trasfer an authorization cookie using https for example

### Make use of a distributed cache for states (High-2)

Don't implement systems that require affinity to serve to function properly

Don't use state or session replication to create duplicates of data on different systems

Don't locate the cache on the system dong the work

You can put your data in databases, non-persistent object caches and hybrid solutions

## Asynchronous Communications and Message Buses

### Communicate asynchronously as much as possible (High-2)

On every external/third party API

For long runnig processes

On error prone/overly complex methods that change frequently

When there's no temporal constraint between two processes

### Ensure that your message bus can scale (High-2)

Scale by service or message attribute

Scale by customer, user, or site attribute

### Avoid overcrowding your message bus (High-2)

Traffic isn't free

Sample traffic where possible

Analyze the business value of every event

## Miscellaneous

### Be wary of scaling through third parties (Very High-1)

You are a hostage of their SLAs

They will not give you special treatment when there's an incident

Be aware of provider lock in

A team exceptionaly can use a third party feature to speedup implementation, but should keep it as a technical debit

### Purge, archive and cost-justify storage (Medium-3)

Match storage cost to data value
Remove data os value lower than the cost to store it
Use RFM: recency, frequency and monetization analisys

### Partition Inductive, deductive, batch and user interaction (OLTP) workloads (High-2)

Every type of iteractioin deserves its own partition

### Design your application to be monitored (High-2)

Prepare yourself to answer:

Is there a problem? (business oriented/customer centric metrics)

Where is the problem? (component/service level metrics)

What is the problem? (low level metrics)

Why is there a problem? (Posmortem)

Will be there a problem? (Last level, prediction)

### Be competent (Very High-1)

Know your tech, or have someone who knows

Delegation is different from omission

The client doesn't care if it's a supplier mistake, your are ultimatelly responsible


# Bibliography
Scalability Rules: Principles for Scaling Web Sites, Second Edition by Martin L. Abbott; Michael T. Fisher, Published by Addison-Wesley
Professional , 2016

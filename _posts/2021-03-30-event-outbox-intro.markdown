---
title: "Outbox Design Pattern"
layout: post
date: 2021-03-30 22:10
tag: 
- outbox
- pubsub
image:
headerImage: false
projects: false
hidden: true
description: "This is a simple and minimalist template for Jekyll for those who likes to eat noodles."
category: blog
author: bryan
externalLink: true
---

# Outbox Design Patter

A freind and mentor of mine recently named this simple and powerful design pattern for me and I thought I would
share it with the world. It solves a big problem with service oriented architectures where network calls can
be unreliable so every new dependency increases the probabilty of a fault happening.

So lets say that you have a service that manages orders for an e-commerce company and whenever an order is
created you want to publish an event to an event bus like Kafka or Google PubSub. We only want to publish
the event if the order is valid and it is successfully written to the database. 

So what happens when the event bus is down?

You can fail the transaction and return an 503, right?

This is defiently the easiest to implement and its not wrong if you absolutely must publish the event right away.
However, it has a lot of draw backs.

1. Signficant increase in the probably of failure (returning a non 2xx response)
2. Increased latency for clients
3. Bad customer experience as the events are not critial to their experience

Ok, lets break this down.

### 1. Probability of Failure

The probability of failure of the API roughly follows this formula:

P(bug in the API) + P(database is down) + P(event bus is down) = Total Probability of Failure

Assuming all these events are independant. So adding another external dependency increases the probability that
the request will fail. This bad... As engineers this is exactly what we are trying to prevent.

### 2. Increased latency for clients

We got another remote thing to call. Typically that other thing has to complete before returning a response to 
the client. Waiting until the other request complete is required if it is required to be transactional with the
database.

### 3. Bad Customer Experience

In the case above we are returning a failure to the clients of the API because a non critical part of the infrastructure, something
the customer does not know about or care about, caused the API request to fail. This makes us look bad and potenailly loses buisness
for our e-commerce company. 

## So How Do We Do Better?

So lets talk about what the outbox is and how it solves this problem. The outbox is a table where we can record any side effects that
we want to happen after the original transaction compeletes successfully and then have a background process transactionally call the
event bus. 

This has a lot of benifits. The side effects are now trasactional with the creation of the order all at the low cost of an addtional
database row insert. 

Ofcourse now we have to monitor these tables and make sure the side effects are executed successully but we have libraries for that.
Some ones I have used before are:

* [ActiveRecord Delayed Job](https://github.com/collectiveidea/delayed_job)
* [db-scheduler](https://github.com/kagkarlsson/db-scheduler)



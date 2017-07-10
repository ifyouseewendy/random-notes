Reference:

* [GOTO 2017 • The Many Meanings of Event-Driven Architecture • Martin Fowler](https://www.youtube.com/watch?v=STKCRSUsyP0)
* [What do you mean by “Event-Driven”?](https://martinfowler.com/articles/201701-event-driven.html)

## Event Notification

![](/assets/event-notifications.png)

This is the common model we use most of the time, to decouple the dependency between senders and receivers, eg. the Shopify core sends an event to Kafka queue with topic like "shop/update", the third party App will have Kafka workers to subscribe on the topic and consume.

Obviously, the bad part is that you don't have a big picture about what's going on as we are deliberately hide the knowledge of receivers from senders.

| Good | Bad |
| :--- | :--- |
| Decouple senders and receivers | Lose the knowledge of what's actually going on |

## Event-carried State Transfer

![](/assets/event-carried-state-transfer.png)



This also a common model which is a kind of stretching part of the first model that how does a receiver have a full knowledge of the event even though a sender has already sent sort of "enough" information about the details of change. Usually the receiver will request the information from the sender through an API, which introduces more traffic to senders. Therefore, the receiver will have its own copy of the information, like a layer of cache.

The downside of every system about "copying" is the syncing problem, data consistency.

| Good | Bad |
| :--- | :--- |
| Decouple and reduce load on senders | Replicated data leads to data consistency problem |

## Event Sourcing

## ![](/assets/event-sourcing.png)

> The core idea of event sourcing is that whenever we make a change to the state of a system, we record that state change as an event, and we can confidently rebuild the system state by reprocessing the events at any time in the future. The event store becomes the principal source of truth, and the system state is purely derived from it.

Event sourcing is a system or thought that we employ events or logs to record the process of the application state changing.  

> When working with an event log, it is often useful to build snapshots of the working copy so that you don't have to process all the events from scratch every time you need a working copy. Indeed there is a duality here, we can look at the event log as either a list of changes, or as a list of states. We can derive one from the other. Version-control systems often mix **snapshots and deltas** in their event log in order to get the best performance.



We don't build event sourcing system a lot, but we use it everyday, eg. git, Rails DB migration, state management in Redux and Elm.




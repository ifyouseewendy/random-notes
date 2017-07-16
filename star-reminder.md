# Star Reminder

### Product

As normal, I go to the office when I don't have something to do. I was reading email subscriptions and surfing online. I was referred to a Github repo about all kinds of free ebooks. I wanted to give it a star, but clearly "Unstar" button showed that I've already starred the project sometime before. It happens sometimes. Therefore, I'm wondering why not to create a service to give me a digest of starred projects every day, to refresh my memory and dig into one or two projects every once a while.

Here comes the idea. I would love to define the MVP as: 

* A service to email me a digest of 3 starred Github projects at 9am every day. \#v0.1

To make it further, it could be: 

* A service to email user a digest of {2,3,5} starred Github projects at {9am, 4pm} every {day, week}. \#v1

Generalization:

* a service to notify \(email, slack..\) user a digest of an amount \(2, 3...\) of starred items \(Github projects, Stack Overflow answers...\) at a designated time \(9am, 4pm...\) every once a while \(everyday, every week...\). \#v8

### Implementation \(v0.1 my-github-star-reminder\)

A script which can

1. fetch Gihub starred projects using my authentication
2. generate a digest consisting of {name, link, description\)
3. send me an email every day










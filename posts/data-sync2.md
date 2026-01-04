---
title: How To Synchronize Data Between Two Services, An Exhaustive Guide
description: Having the same data in two services is harder than it looks, here's how to do it.
date: 2025-11-21
layout: layouts/post.njk
---

In this article I want to talk about a ubiquitous problem in computer engineering that seems trivial
on the surface but is actually quite complex. The problem in question is how to keep data synchronized
between two different services that have their own database. Let's say you introduce data *A* in service *X*,
and you want this same data to be in service *Y* as well. How do you do that reliably?

A common and basic instance
of the problem is sending a mail and recording that the mail has been sent. Usually the mail sending is done by a service
which you call using a networked  REST / RPC / SMTP call. That mail service has its own knowledge of the status of the mail
sending. But you will usually keep a copy of that status in your main application. For example you might want to record that
a user has been sent a password reset email. This seems simple enough but the two obvious ways to do this are unreliable. Let's have a look at them.

<img class='wide schema' src="/img/eventual-consistency/send-mail-1.png">

The two basic ways to do this are to either record the status before or after sending the email. The main problem with both approaches
is that making a network call can fail in various unpredictable ways. So if the first operation succeeds and the second fails, you
are left with an inconsistent state where either the mail has been sent, but the status hasn't been recorded, or the status has been
recorded but the mail hasn't been sent. But don't worry, it is possible to guarantee to have both operations succeed, you just have to
use a completely different approach, and this is the whole point of this lengthy article.

<img class='wide schema' src="/img/eventual-consistency/send-mail-2.png">

Before going further, we need to dive a bit deeper into the nature of the failures of networked calls,
because those failures are unavoidable, and we'll have to work with them.

## On the many ways an inter-service network call can fail

Before deep diving into the failures I would like to note that the diagrams used to
model network calls are not accurate to what happens on a network and processing level. We often think of a network call as having the following logical steps:

 - a request payload is sent to another service
 - the request payload is processed by the other service
 - the service sends back a response payload
 - the response is then processed.

But on a network and processing level the separation is more blurry. There are usually packets going back and forth during the whole duration of the request; the request payload is usually sent in many small packets and each of them trigger their own acknowledgement response, and so is the response payload. There are also various keep alive mechanism that can keep sending packets in between the request and response payload transfers, and with modern application framework supporting streaming, even the processing of the request and the response is often done in parallel to their transfer.

<img class='schema' src="/img/eventual-consistency/schema-vs-reality.png">

The reason I mention this is that it is important to realize that network level failures happen at that level, not at the logical level indicated by the schematics, and can thus happen at any time, without regard of the logical boundaries of the flow of a call. But since all our next discussions will use these higher level call diagrams, I will attempt to map the different errors at the logical boundaries.

So the diagram below represents a simple call from one service to another, along with the logical failure points.

<img class='schema' src="/img/eventual-consistency/rpc-failures.png">

We often hear that network calls can fail, due to the "network being down", but in reality the nature of
the failure is much more diverse, and often due to higher-level issues rather than physical network outages. An
inter service call is a beast that involves almost all the layers of the network and application stack and
failure can happen at any of the layers in various and exciting ways. At the end of this article we will
reach different solutions that can abstract those away into a reliable channel, and we will be able to forget
about all these, but in the meantime we need to have at least a good idea of what they are so that we can
design a solution around them. So let's go failure point by failure point, and enumerate the different failure
modes for each. I hope you will forgive me for not going too much in depth with any of them.

### A. The other service is unreachable.

Our first packet hasn't even reached the server of the other service but many things can already go wrong. All
of these errors are what's usually called 'the network being down'.

- The DNS is down and you can't get the IP of the target server
- The DNS is misconfigured and you can't get the right IP
- You're getting stale IP from the DNS cache
- The IP is unreachable due to network misconfiguration
- The packets are blocked by a misconfigured firewall
- The packets are dropped because the network is overloaded
- The packets are dropped by a network level rate limiting system

### B. The other service refuses your call

It's not because your packet reached the right IP that the call is accepted. Many things can yet go wrong.

- The server is down
- The server is up, but not yet ready to accept connections
- There is a protocol capability mismatch (HTTP / TLS) between the two services
- The credentials are wrong
- The call is dropped by rate limits
- The payload of the call is refused by the application. This can happen because you're hitting an incompatible
  version of the service in a continuous deployment scenario.

### C, D. One service stops during the call.

For the call to succeed both services need to stay up and available during the entire duration of the call, but this is not something that
can be guaranteed

- A service crashes due to application level failure.
- A service is killed due to excessive resource usage
- A service is shut down by the auto scaler / scheduler
- A service has an application level timeout for the call duration

### E. The connection drops during the call

We often see in network call diagrams a return line after the end of remote processing, as if the third
party network was waiting to complete its task before sending the first packet of his response, but this
is not accurate. The connection is in fact open during the whole duration of the call, and packets are
transmitted back and forth for its whole duration. What this kind of error represents is that the connection
drops after the connection has been established, and parts of the return response never arrive. This is more
likely to happen for large responses that take a long time to transmit. Here are some reasons why this could happen

- A proxy between the services has a timeout on request duration and closes the connection prematurely
- NAT mapping drops
- Network topology changes mid connection
- Overloaded network starts dropping packets
- Overloaded server drops the connection

### F. The caller refuses the response

There is one last thing that can go wrong and that is the application being unable to process the response payload correctly.
Maybe the caller didn't expect a field being null, or is surprised by the presence of a new field. These are application level
bugs, not network errors, but the end result is the same, the call did not get through successfully.

## Minimizing Errors

If we look at those errors from a bit of a higher level we find that they belong to a few different categories

- Network configuration changes
- Application level changes
- Calls that take too long
- Excessive loads
- Credentials errors

There are strategies to minimize each category of error, and if you want to have a reliable system you should definitely put them in place.

### Manage the network close to the application.

Most of the network errors are due to network configuration changes that are made without thinking of their potential impact on the application. The best way to minimize this is to have the people managing the application manage the supporting network infrastructure. This is hard to do with 'on prem' infrastructure, but is easy to do with cloud environments, where the infrastructure can be managed as code close to the application, and the infrastructure changes can be deployed and tested with the same lifecycle as the application.

### Use forward compatible contracts and only break changes in new versions

API versioning is an art in itself, but the key thing to keep in mind is that publishing a new version of a service should not break
the existing users. This usually means that a service will only add fields to the responses, never remove them, and that the caller will
accept responses with unknown fields and ignore them. If this is not sufficient a new version should be published on a different path.
The caller service will have to eventually implement the new version and drop the old one. A version can only be dropped when there are
no more users. Be prepared to wait a long time, this process is usually measured in years. There is also a different approach where the
caller will provide a versioning field as part of the payload, and the service will return a compatible response. This might make the upgrades smoother since the calling path is never changed, which is usually a pain point as those paths tend to become hardcoded into the weirdest corners.

### Make calls short by design

The shorter the call, the less likely something wrong happens while it is ongoing. This is why you should design your application so
that every call is expected to be short, 250ms maximum. This is sometimes impossible, for example with streaming video, audio, or websocket connections. In those cases, the services serving those long running requests should be isolated from the other regular services and have as few intermediate hops to the caller.

### Keep your server lives short

A common root cause of errors is a mismatch between the server's configuration and its environment. It is much easier to load static configuration settings at the server start but reboot it often to get newer settings than having those settings dynamically updated during the server's lifetime. Servers also tend to accumulate issues during their lifetime such as stale caches or memory leaks. Rebooting the server on a regular basis is an easy and practical way to solve these problems. Having short lifecycle imposes fast boot times which has the added benefits of faster scale up during load, solving another common source of errors. You should aim for a server lifetime of 10 minutes and startup measured in seconds.

> NOTE FROM THE REVIEWER
>  1. "10 minute server lifetime" recommendation (line 135)
>    - This is very aggressive and only applicable to specific architectures (serverless, FaaS)
>    - Most production systems run instances for hours or days without issues
>    - While the Chaos Engineering philosophy supports this, it's not a universal requirement
>    - Suggestion: Frame this as "aim for short lifetimes (under an hour) where practical" or make it clear this is optimizing for a
>  specific architecture style

### Make services horizontally scale by number of requests

There is a reason why lambdas and elastic container services scale by number of requests by default. If cpu usage becomes too high,
the serving of the requests slows down, but the requests are still served. If memory is exceeded, the service is killed, yes, but it is easy to set an instance memory
limit high enough such that the number of requests served is always the bottleneck, and the memory limit is thus never hit. With autoscaling per request you ensure you always have the capacity to serve the incoming requests, except for unpredictable spikes, but there is no workaround for that one.

### Automate the distribution and rotation of credentials

A significant source of errors are due to requests being made with the wrong credentials. This is usually due to the credentials having
been changed at the service side. Correctly distributing and changing those credentials is not an easy task. One way to avoid this is by using permanent api keys that are never updated, but this is not the best
practice security wise. Modern clouds provide IAM services where you can configure in a declarative manner which service can connect to
which service and for what operations. The IAM system will then distribute the correct credentials settings on the api gateways and to the services. If such services are available to you I advise that you leave out the authentication entirely from your application code and leave it to your cloud's IAM, whose implementation is much more likely to be correct than yours.

### Centrally monitor your requests status

Some errors are transient, but some are permanent and require manual intervention. If you do not see those errors happening, you don't have any chance to fix them. The common
solution is to log every outgoing and incoming request with a standardised structured log that contains the service id, the request path, its status and duration. A request id should also be logged so that the logs of the different services can be correlated. All those logs should be collected to a centralised platform, and errors should trigger alerts to the communication channels of the service maintainers so that they can quickly work on the issue.

Here is an example of a structured log that you might want to emit for every incoming request

```
{
  "logType": "REQUEST_INBOUND",
  "service": "service-y",
  "host": "service-y.domain.com",
  "sourceIp": "245.138.12.12",
  "sourceHost": "service-x.domain.com",
  "method": "POST",
  "path": "/api/v2/orders",
  "status": 201,
  "requestId": "9c1c82a8-41dd-4c78-bb5a-fb62d7df63b4",
  "timestamp": "2025-11-28T15:32:12.151Z",
  "duration": 56,
}
```

## Idempotency


In the previous section we listed all the good practices to minimise the source of
errors for inter-service calls. But in doing so we gained something else. We made all
the errors transient, meaning that if you wait a while, the source of errors will eventually
go away. And thus you can make a call reliable by retrying it for longer than the duration of
the error.

But here comes the first problem, is it actually ok to retry a call? When we looked at the source of
errors we noticed that a few of them happen *after* the call is successfully processed by the remote service.

If we then retry for those errors, the call will be processed twice. If the call in question was
for example, sending an email, we'll send two emails, can we avoid that?


<img class='schema' src="/img/eventual-consistency/retries-non-idempotent.png">

The solution is to make sure the retried calls are idempotent. Idempotency means that you can make the
same call multiple times and get the same effect and response as doing it just once. There is no single way
to achieve idempotency, and the correct strategy depends on the nature of the call being made, so let's explore
the different strategies.

### Trivially Idempotent Calls

Some calls are idempotent by nature and do not require any adaptations. A good example is a `DELETE` call. Once
an entity has been deleted, deleting it again doesn't do anything, so you can simply have your call return a success when attempting to delete a record that has already been deleted.

Another example is `GET` calls, or any call that just reads data. Since the read doesn't modify the data, reading it more than once can be done safely, and a `GET` call can thus generally be retried. There is a catch though, which is that the data that is being read can be modified between retries, in which case the `GET` is
not idempotent, as the retry will not return the same response. One way around it is to not allow modification of data, but more on that later.

### Idempotence For Data Insertions

A common operation that must be made idempotent is the insertion of new data in a database. One such example is the recording of the mail being sent, where we probably want to add a record like `{ "sent": true, "email-type": "signup", "address": "foo@bar.com"}` to our database.

The first approach is to define a unicity constraint on key fields of our table so that double insertions can be
prevented, and in which case we would return a success as if it was the first time being inserted.

Another way is to allow double insertions, and have instead our `GET` calls on those records always return the first insertion.

Which approach to use depends on the philosophy of the design of our data model. When we prevent double insertions, it is because we conceive our database as a snapshot of the correct state of our models. When we allow multiple insertions it is because we conceive our database as a timeline of everything that happens in our system. The timeline way of doing things is generally more complex to get correct but has the advantage of not requiring transactions on insertions, a feature that might not be present in your database system, or can be a performance bottleneck. It also simplifies updates, our next topic.

### Idempotence For Data Updates

The first problem with updates is that the idempotence is not decided by the called service, but by the caller. Imagine that you have a service that holds currency balances for accounts. How can the service holding the accounts know what to do when it receives two requests to increase of the same amount for the same account? Are
those two duplicate calls coming from retries, or does the user genuinely want to do the same transaction twice?

The solution is to add to the data update call an idempotency key. The idempotency key is a separate field on the call that will be set to the same value for retries, and to different ones for legitimately similar but different calls. The idempotency key value is thus decided by the caller, usually at random, and with sufficient randomness to avoid unintended conflicts. UUIDs are a good fit for that purpose.

How to implement idempotency keys once again depends on your database design philosophy. In the case of the "database as the current state" philosophy, a separate table holding the already seen keys must be used. In the case of the "database as a timeline", the updates will be stored as new updates records, including the idempotency key. It is up to the read calls to do the deduplication.

Another important aspect of idempotence for data updates is that retries can change the order of application of your calls. Imagine you want to do 3 updates in a row, `A`, `B`, `C`, and `A` fails at first try. It might be retried after `B` and `C` have already been applied ... But the whole question of ordering in eventually consistent systems is a complex topic that I will touch on later in a dedicated chapter.

### Idempotence For Side Effects

So far we've looked at how to design idempotent api calls when we are in control of a database. But what if
we don't have a database? For example the call to send our email might be a call to a third party provider that hasn't an idempotent API. Or we might interact with a FTP Server, a printer or a filesystem. Can we retry those calls too?

The unfortunate answer is that there is no way to completely prevent the side effects of those interaction happening more than once. But we can diminish the probability to a great extent.

The idea is to guard those calls with a check for an idempotency key in a dedicated database. Before doing the actual call we check for the key. If not present we set a lock with a lease timeout, and
after the call is successful we set the key permanently. If a retry happens during the lock, we fail and retry later. If a retry happens after the key is set, we ignore and succeed.

This mechanism unfortunately suffers from the exact same problem that we've been facing since the start. Since it
does multiple calls in a single request we cannot guarantee that it will always execute perfectly. But it can reduce the duplicate side effects to a tolerable level. The real solution is to avoid having to interact with
such systems by choosing better technologies.

### The Importance of Idempotence

Before moving on the next topic, I want to stress one last time the importance of idempotence. It is absolutely essential, if you want to make your system eventually consistent, that all the calls of your service support idempotence mechanisms. And it cannot be an afterthought, as the only way to achieve this is to properly design your database model and your apis with idempotence as a core principle.

## Implementing Retries

Now that we know how to design a call so that it can be safely retried, we can look at how to properly do the
retries, as here again things are not so simple as they look at first glance.

<img class='schema' src="/img/eventual-consistency/retries.png">

### Knowing when to retry

You make a HTTP call to another service and you receive a `404` error code. Do you retry? Let's look at the possible reasons
you are getting the `404` response.
  - You made a mistake on the endpoint address. This is a permanent error, you should not retry
  - The endpoint address is correct but you're reaching an old version of the service that is being updated. This is a transient error, you should retry
  - The endpoint is correct but contains a reference to a resource that doesn't exist at all. This is a permanent error, you should not retry.
  - The endpoint is correct but contains a reference to a resource that does not yet exist on the service. This is a transient error, you should retry.

Some errors, such as timeouts or `429 Too Many Requests` give an extremely clear indication if you should retry or not. But other errors are less clear, and the correct decision depends on the particular specification of the service, and its interaction with other services as well. This lack of clarity makes implementing the retry logic error prone.

Fortunately the called service has usually a good idea if the error should *not* be retried, and can communicate that to the caller. Then the caller can retry in every case unless it received this clear indication that the call should not be retried.

Unfortunately the most popular way to design APIs, REST makes this challenging. RPC frameworks such as `grpc`, `jsonrpc`, `trpc`, or
modern query framework such as `graphql` have much better defaults. The key insight they have in common is that `200` responses indicate
a successful call that should not be retried, even if they contain an application error. They also clearly separate the endpoint address
and the call parameters, which are always put inside the request payload instead of the endpoint path.

Sometimes the decision to retry or not is taken away from you because the retries are done with predetermined retry logic. A common example is cloud event systems, such as Google PubSub, AWS SNS, or Azure Service Bus. PubSub will retry on everything that is not a `1XX` or `2XX`. SNS and Service Bus work differently, they retry on `1XX`, `408 Request Timeout`, `429 Too Many Request`, `5XX` and network errors. You should keep that in mind when designing your APIs because in event-driven systems most of your calls will be retried by event systems

### Timing your retries

How quickly, how often, and for how long should you retry? Whatever strategy you choose should have the following goals in mind
- It should retry quickly at first, to keep overall system latency low
- It should retry for long enough that the errors get solved in the meantime
- It should not overwhelm the target service with too many requests.

There is one strategy that reaches all these goals and that is commonly used, it goes as follows
- First retry immediately. There's a good chance that the second request will reach another server or use a network path in a healthier state.
- Then retry slower and slower with an exponential decay. This leaves room for the target service to scale up or recover from the excessive load.
- Randomize the delay between the retries. This prevents all the retries from going on at the same time, which plays badly with the target's service scaling algorithm, a problem called "The Thundering Herd Problem" (https://en.wikipedia.org/wiki/Thundering_herd_problem)
- Retry for days. It is much better for the system to eventually be consistent after a few days automatically than to stop retrying after 30sec and have to solve that inconsistency yourself manually.

As concrete examples GCP's PubSub default retry strategy is to retry after 10sec, double that every retry until a maximum of 10min is reached, then retry every 10min for seven days. AWS SNS retries immediately 3 times, then every 20sec 3 times, then doubles that every retry until the interval reaches 20min. Then it retries every 20min for 23 days.

### Retrying from a network call

<img class='schema' src="/img/eventual-consistency/retries-from-call.png">

The problem with retrying from a network call is that the call has an associated timeout in the range of seconds to minutes, and this is much too short to retry long enough to expect the problem to solve itself. Does it make sense then to still retry from a
network call, or any other transient place?

It can make sense to do a few targeted retries of known transient errors such as `503 Service Unavailable`, `429 Too Many Requests` and clear network errors. Doing so will hide a bit the underlying problems from the original caller, but is not by itself a way to
achieve eventual consistency.

I would argue that instead of doing in-network-call retries, you should instead immediately fail and bubble the failure up the network call chain until a proper place to retry is reached. That proper place being your end client or a durable event system.


## Event Systems as Managed Retries

As we've just seen one key aspect to make a call reliable is to make it idempotent and retry it for a long time. The problem is that the
first place you'd put that retry logic, the request where the call is made, doesn't last long enough to retry for an appropriate duration. To make the retry last long enough, you need to record durably the fact that you want to make a call and retry it. There
are two main places where you can record that, the database and an event system. The distinction between those two solutions can be
blurry since they end up achieving the exact same thing, but we need to start somewhere so let's look at event systems, especially
the ones that are natively provided by cloud platforms, such as Google PubSub or AWS SNS, since those are the ones you'll end up using
for that purpose.

### Event Systems in a nutshell

An event system, such as Google PubSub or AWS SNS has two main components, the first is a storage for events, and the second is a
system that will distribute the events to other services. The key is that events do not stay eternally in the storage, they're removed after they've been distributed. And what's an event? An event can be any reasonably small piece of data. PubSub has a limit of 10MB, SNS has a limit of 256KB. Usually an event would contain a small piece of structured data, encoded in `json`, `Avro`, `xml` or `Protobuf`. Events are usually grouped by topics, each topic containing similar events. The topics are also used for the distribution of events, as every topic can be subscribed to by different services. Each event put into a topic will be distributed
to every service that has a subscription to that topic.

<img class='schema' src="/img/eventual-consistency/events-topic.png">

The fact that a single topic can distribute events to many services is an incredibly powerful feature but what most interests us right
now for our use case is that an event subscription *guarantees at-least-once delivery*. What that means is that the event subscription embeds the retry features that we've talked about previously, and thus once an event has been published to a topic,
you can be sure that it will be received by the subscribed service, even if the service is down for multiple days. An event subscription is the reliable connection channel that we've been lacking so far. Of course, since the underlying reliability is based
on retries, all the discussion we've had on retries, especially the aspect of idempotency still applies.

For sake of completeness we have to note that there are two kinds of subscription mechanisms with different retry characteristics,
the push and the pull based subscriptions. Let's have a look at them.

### Push Subscriptions

A push subscription is a subscription that actively calls the subscribed service. When the service creates the push subscription it
specifies a `POST` endpoint, and the subscription will call that endpoint with the message and accompanying metadata as the payload.
If the `POST` call fails, the subscription will retry, and will do so until the call succeeds at which point the message will be
removed from the subscription.

<img class='schema' src="/img/eventual-consistency/push-subscription.png">

Push subscriptions are the simplest to put in place but they have two caveats. First they will push events to the service as fast as the
events come in. It is thus important that the subscribed service is able to handle quickly varying levels of loads from the events.
If you followed all the good practices we've mentioned previously while designing your subscribed services this should not be an
issue. But sometimes you want the subscribed services to be in control of the speed at which the events are processed, and for that
use case there are the pull subscriptions.

The other limitation is that the time allowed by the subscribed service to process the event is limited
to the duration of the network POST call to do the request. These would usually be limited to a minute, configurable to a maximum of 10 min or less depending on the event platform.

### Pull Subscriptions

A pull subscription is a subscription where the subscribed service actively polls for new events. It will periodically make requests
to the subscription to get the new events, and once it has successfully processed an event it will notify the subscription that
it has received the event. Only after the subscription has received that acknowledgment will it remove the event.

<img class='schema' src="/img/eventual-consistency/pull-subscription.png">

Parallel processing of the events is still possible with this setup as each poll for events will return a different set of events, even before they are acknowledged. An event sent in a poll is considered 'leased', and it will only be sent in another poll after the
short leasing period ends, unless it has been acknowledged in the meantime.

A big annoyance of the pull subscription is that you have to manage the retries and the acknowledgement yourself. But you gain
the ability to control the rate of consumption of the events which might be necessary if you interact with an inherently rate limited system. Another benefit is that polling return events in batches, which enables you to process them in batches as well. And since the acknowledgement is a separate call from the poll, you are not limited by an in-flight request timeout to process the event. Some event system take advantage of this and do not impose a time limit on the event processing.


## Event Sourcing

Now that we have a way to make reliable calls in the form of events, let's look at our original problem again. We wanted our app to record the sending of an email in a database record and send the email, but we couldn't make that work because both operations can fail. But with events the story changes as we *can* make both calls
reliable. The question is how and where to put the events, as there are right and wrong ways to go about this.
Let's first look at the *wrong* way to do it. Can you see what's wrong ?

<img class='schema' src="/img/eventual-consistency/events-wrong.png">

The problem is that while the event processing itself is reliable, posting the event on the channel is not. It is still a network call! So the original problem of having one or two of the operations succeed is still there.

The solution is to start with *one event*, and cascade from that original event. If we start from one event, either the event is registered, or it is not, and whatever is downstream of the event is guaranteed to happen due to the retries. This is why the technique is called event sourcing, we put an event as the source of our process.

Now how do we trigger both operations from one event? The naive way is to subscribe both operations to the single event. This will guarantee that both operations eventually apply, but since the two subscriptions are processed independently there is no guarantee in which *order* they are applied. We might potentially record
the mail being sent, but actually sending the mail hours later. This is probably not what we want.

<img class='schema' src="/img/eventual-consistency/event-sourcing-naive.png">

The right way instead is to simply have an event handler that does both operations one after each other

<img class='schema' src="/img/eventual-consistency/event-sourcing.png">

While this looks suspiciously similar to our original solution that didn't work, the key difference here is
that the event subscription will retry the handler until it returns a success and the handler will return a
success only if both operation succeed. Problem Solved !

### Event Cascades

Having a single event handler that does the necessary operations in sequence works well but has some limitations. First it is limited by the acknowledgement deadline of a single event, so there is a limit to how much work you can put into one handler. The other limitation is that upon failure, the whole sequence will be retried. If you have proper idempotency in place this is not a problem for correctness's sake, but if some operation in the sequence is prone to unreliability issue, the whole handler will be affected. Another last point is that the rest
of the system has no idea where you are in the sequence.

For all these considerations it might be useful to split the operation sequence into a sequence of events, where each operation triggers a new event, that then triggers the rest of the work.

Continuing with our example this would mean introducing a second `Mail Sent` event that would be emitted by our handler after the mail has been sent. The recording of the mail sending would then be done from that event.

<img class='schema wide' src="/img/eventual-consistency/event-sourcing-event-cascade.png">

I want to stress that implementing this pattern for this present use case is completely overkill and has no additional benefit, but it will let us explore a common issue that appear when this pattern is implemented for good reasons.

The issue is that a late failure of the first event handler might trigger a re-submission of the second event, and now our idempotency concerns must work not only for repetitions from a single event, but from repetitions across multiple different events.

<img class='schema wide' src="/img/eventual-consistency/cascading-retries.png">

There is a simple way to prevent this, and this is to use the same idempotency key across all the calls done from the same event source. The application should choose a key, set it in the `Send Mail Event`, and that idempotency key should be forwarded across the calls and the `Mail Sent Event`.

<img class='schema wide' src="/img/eventual-consistency/event-sourcing-idpkeys.png">

### Cloud Managed Event Workflows

Implementing the above described multi event system correctly is not easy. This is why cloud platforms provide
higher level frameworks that let you describe the event sequence with a high level description and let you plug
in the meat of the operations, with all the retries, idempotency and timing concerns handled by the platforms. I will not go into details of how to use them, but you can have a look at their documentation.

- [Azure Logic Apps](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-overview)
- [AWS Step Functions](https://aws.amazon.com/step-functions/)
- [GCP Workflows](https://cloud.google.com/workflows)

-------

### Event Bus

If you still want to implement the multi event pattern by hand, there is a way to do it that might simplify your life. The idea is to put all your service events on a single event topic with a single subscription handler. Each event will have a `type` field, which will let the handler dispatch the handling of the event to
different business logic function depending on the type. Functionally this is not different from having multiple
event stream for every step. The advantage of the event bus pattern is seen in the code of your application where
it will be easier to follow the logic of how the different events relate to each other, and it will be easier to enforce and put in common the complex logic related to idempotency keys.

And with all the event handling logic on the application side, adding a new event is an easy code change that does not require infrastructure change. Having the event handling at the application side makes it also easier to swap the event bus backend for a simplified one that can be used for local development or unit tests.

The downside of the event bus pattern is that it is harder to control what other service see of your service events as a subscription will let them see all the events. Most event system allow a subscription to filter on the type of the event, so this is not really a matter of performance, but rather of control. Should another service really depend on an internal step of your application logic ? When things are easy to do they are often done from convenience rather than good architectural sense. If you need to expose your bus events to other service it might be good idea to first forward them to dedicated topics where it will be easier to control access and ensure backward compatibility of the events structure.



---
END OF THE CURRENT WORK IN PROGRESS, THE FOLLOWING IS STILL TO BE WRITTEN


## Databases as event systems, the outbox pattern

## When Eventual Consistency Isn't Worth It

> Good idea! A chapter on "when to use these patterns vs simpler approaches" would ground the article in practical decision-making.
>   Here's how I'd structure it:
>
>   Possible title ideas:
>   - "When Do You Actually Need This?"
>   - "Choosing Your Consistency Guarantees"
>   - "Trade-offs and When to Keep It Simple"
>
>   Topics to cover:
>
>   1. The cost of guaranteed eventual consistency
>     - Architectural complexity (outbox tables, CDC, event infrastructure)
>     - Operational overhead (monitoring event systems, debugging async flows)
>     - Increased latency (async operations)
>     - State management complexity
>   2. Alternative approaches and their trade-offs
>     - Query-on-demand: Don't sync data, just query the remote service when needed
>         - Pro: Simple, always fresh data
>       - Con: Dependent on remote service availability, higher latency
>     - Best-effort with inline retries: Accept that some operations will fail
>         - Pro: Much simpler, lower latency
>       - Con: Manual intervention needed, requires compensating workflows
>     - Periodic reconciliation: Batch sync + detect/fix drift
>         - Pro: Simpler than real-time event systems
>       - Con: Inconsistency windows, doesn't prevent issues
>   3. Decision framework
>     - How critical is consistency? (financial transactions vs analytics)
>     - What's your SLA? (99.9% vs 99.99%)
>     - Can you handle manual intervention?
>     - Is the data truly needed locally or could you query it?
>   4. Start simple, evolve
>     - Many systems start with inline retries and evolve to event-driven as needs grow
>     - Premature optimization vs knowing where you're headed

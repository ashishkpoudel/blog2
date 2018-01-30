Title : Microservices - These Are Not The Droids You're Looking For
Tags : Architecture
Published : 2018-01-29
---

Quite often these days, while reading job ads I see the "microservices" requirement, sometimes even for junior positions. Microservices have become such a buzzword (next to XML and JSON - I've seen them in the same enumeration, literally) that everyone thinks they're needed. It's like:"Oh, we have a monolith right now, a big ball of mud and we want to switch to microservices". 'Cuz obviously monolith = bad coupling, microservices = good design.

In practice, the following image represents the reality.
![Monolith and microservices are both shit](https://image.slidesharecdn.com/micro-services-150905213111-lva1-app6891/95/micro-services-24-638.jpg?cb=1441537431).

Sad but true.

Personally, I'm a big fan of microservices(mS), but most people are missing the point: the value of mS lie in proper design, while the whole recipe is more of an implementation detail. But guess what? Proper design is required anyway, going distributed doesn't guarantee a _good design_. It guarantees, though, more complexity and higher costs.

That's one myth that needs busting: going distributed in any form (mS, 'old' SOA, serverless) doesn't automatically mean maintainability and scalability i.e good design, while using a monolith shouldn't automatically imply bad design. Sure, a monolith has its limits, but with a well designed app you can extract easily the relevant parts that needs to scale and make them microservices.

Few applications have the need to be a group of microservices from the beginning. Most apps can start as monoliths or a bulk of functionality as monolith and some need-to-scale functionality as microservice. But these are implementations details in the end.

Let's look at the following architecture (taken from my DDD presentation at DevTeach 2016)
![Main design](https://i.imgur.com/riqUfCC.png)

At an abstract level the app is organised as components a.k.a vertical slices (that we can group further into UI, Domain and Application) that communicate in a decoupled manner by using a pub-sub approach. The 'rules' are simple: any functionality inside a component can send a command (that can be handled anywhere in the app - except UI) and every component has event handlers that will listen to events (usually Domain Events, but it depends) that are of interest to the model. Nothing new here and it actually resembles microservices.

But it's not. It's just how we **look** at things, identifying model boundaries, UI/API functionality and how components communicate in the most decoupled manner. The main idea here is to **establish responsibilities and boundaries**. Do we really need a separate project for each component or a service bus? No, we can choose the easiest _implementation_ that respects the boundaries. I can 'enforce' boundaries by having each component in its own namespace (a soft boundary, but still a boundary) and the message bus can be an in-memory mediator (something like [MediatR](https://github.com/jbogard/MediatR) or equivalent in your stack).

My point is here to show that a fancy high level architecture doesn't mean a complex implementation. As long as boundaries and their responsibilities are respected, you can go wild (as in simple :P ) with the implementation.

Let's take a look at the UI.
![UI](https://i.imgur.com/xiw0562.png)

Easy to recognise a CQRS approach. Well, it's actually CQS but at this point it's just semantics. What matters is the UI doesn't handle commands at all, but it maintains its own read model to handle most if not all queries. Let's assume we're working on a simple app (one project) using with your favourite MVC framework. Since we don't handle commands i.e doing business logic in the UI (controller), we can put it into a nice service (class) in a different namespace, inject it in the controller, have the action invoke the method. Very easy and now that simple business logic is no longer part of the UI (from a design point of view). Yes, things change a bit if the command handler is in a different component hosted on its own server, but even then, the bulk of the use case code kinda stays the same, mostly the infrastructure bits need to change.

In a way, this is our objective: when scaling an app we want to change anything business related as little as possible, working mainly on the infrastructure.

What about that read model, is that a specific db? No, from a design point of view it's everything you need (data structures, db schema etc) to return app queries (read functionality). Sure, the UI is the 'owner' of that read model (which can be more than one), but the actual implementation should be, again, as simple as possible. This means using the same db schema that other components write to. You really need very good reasons to have a dedicated storage for your UI or any other component, as costs and complexity just increases.

For other components (Domain or not) things are similar, we can still have a specific model and apply CQRS at both application and persistence level, but in abstract terms it will still be a command or query model. We can consider the persistence as an abstraction, like some storage where business state exists, but we don't know details. The actual implementation can share the db at least for reading or for everything or each component can have their very own, isolated db. It depends on the scalability needs.

An architecture fit for your app can use a monolith for a lot of functionality and microservices (as in separated hosted services) for anything that needs to scale. Hell, we can use a bit of serverless as well for specific functionality that uses several cloud features and just invokes some api. You can literally implement a microservice like that. Is this a serverless or a microservices architecture? Neither! It's an architecture that combines bits of available tools.

 The microservice's size should be as big as it's needed for it to be scalable (but not bigger than a Bounded Context), as small as it can offer all required functionality (according to its responsibility) and its implementation should be as straightforward as possible. A fully autonomous microservice (this means its own server, db etc) is desired only if you have a proper business reason. Implementing microservices according to a recipe and not customized to actual needs just leads to complexity and high costs. Especially if you want your whole app to be a group of microservices. Sure, it _can be_ a good architecture if the design is proper, but are you going to need that from the beginning? If you are a big company with deep pockets expecting millions of users, then maybe it's worth it. But not everyone is that company and things needs to be adjusted according to your business and budget reality (and I don't know many managers who will be disappointed that you provided the needed functionality at a lower cost).

And it's similar with any type of architecture distributed or not. Modern solutions keep appearing and you've heard that big company X used this and that but unless you have the same needs and budget like company X, I suggest to understand the technique, see if it can be applied in your case then adapt it to your situation. As with all techniques, the principles (or the paradigm) are the important bit, the actual implementation should be very specific. Following a recipe to the letter because it was in a book or tutorial is how you end up with costly overengineered applications.
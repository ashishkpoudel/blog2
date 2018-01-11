Title : It's Time To Let The ORM Go
Tags : Best practices
Published : 2018-01-09
---

If you know and use Object Oriented Programming, chances are your view of the Business Layer is like this: we have business objects which contain business state and business rules. Since we need to persist the business data, we have to get tha data from the objects and save it into some storage (usually a RDBMS) then retrieve it back to restore the business objects.

Boring stuff, prone to human error, how about we automate that? And this how the ORM was born, a great idea at the time, but increasingly obsolete. They say that little is new in the development world, we just rediscover concepts that existed since the '70s, but we now have the computing power to put them to work. While it's true, I think the software development community added new tools, especially new paradigms, that can make our life easier, _if_ they're used properly.

When ORMs were in their infancy (early 2000), OOP was the coolest mainstream programming paradigm and RDBMS were the norm. DDD was very niche, NoSQl, Event Sourcing, CQRS didn't exist. Social media was at best in its infancy, Google was still competing with Yahoo, 'scalability','distributed' were niche words. Simply put, things were more primitive.

Enter the ORM, the magic tool which made things even easier. Since persistence was an implementation detail, we let the tool take care of that boring stuff and we pretend the database doesn't exist. Only business objects exist to be changed or queried, Sql be damned. One less concern you might say.

But there's a problem. The developers' mindset, which is pretty CRUD. And really, we can pretend the db doesn't exist but we still need to configure the ORM and write those mappings from a class state to one or more tables. And the developers still think data first, so their business objects are very similar to a table. While the code said "class", the mindset thinks "table" and that's how the business objects turned out to be mainly tables with added methods. Both classes and tables model a data structure, it's easy to mistaken one for another when it comes to data. After all, values from object's fields/properties go to table columns and it seems natural to have "one to many" relationships, a RDBMS concept, between business objects (relationships that inside a bounded context are part of the ubiquitous language).

Thinking data first (like a db schema) makes the ORM mapping configuration very easy. And in a lot of apps these days where an ORM is used, the Business Layer is **unknowingly designed as db tables with methods**. Yes, the ORM made some things easier but the price is a CRUD Domain.

There are very legit CRUD Domains and an ORM shines in those situations, until you realize that you can do the same things (but faster) using a data mapper (a.k.a micro-ORM) with nice helpers. Sure, you no longer get to kid yourself the db doesn't exist and you're using objects only, but guess what? It's not a bad thing. Database is a reality and we need to take it into consideration.

Even better, using Sql and all the db features makes your life easier. There are DBAs who can help you with complex queries, who can optimize the shit out of the db. But with an ORM, you depend on the tool's smarts and if your queries involve many objects the practical solution is .... get someone to write a stored procedure that you can invoke via the ORM. Yeah, in essence, bypassing the ORM. So why fear the database? It's better to exploit the 'enemy' instead of hiding it with abstractions.

 What about more complex domains? This is where problems start piling up. A domain model represents an abstraction of domain _functionality_. The "business layer consists of business objects" view is too limited and cumbersome for complex domains which require scalability, especially with the modern trends of microservices and serverless.

 Hell, even before current distributed apps, monoliths using the ORM were too slow for the different type of queries. That's why CQRS appeared which makes the ORM handle only simple queries or be just a glorious data mapper. Add different read models (projections) that might or might not need a RDBMS and Event Sourcing into the mix and the ORM's role becomes just a small part and often can be successfully replaced with a data mapper.

And the worst thing you can do is to 'design' your domain model on top of (or around) the ORM.  Start with proper DDD and you're going to struggle to configure or to find a use for the ORM. Truth is, with modern methodologies (DDD, Event Sourcing, CQRS) and light tools like data mappers, we can write more **maintainable** and flexible applications.

The ORM was conceived as a solution to simplify the developer's work with the relational database and it's a very bad idea to make it the spine of the app, that you can't remove. While it works well within CRUD apps, even there is just a bloated, obsolete tool (ok, I'll admit it, for very simplistic apps used by 5 people it's a great convenience).

It was a good idea while we didn't know better, but today we do know better and the ORM in most of non-trivial apps is just a burden. Let's make things simpler!
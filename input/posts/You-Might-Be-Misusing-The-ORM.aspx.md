Title : You Might Be Misusing The ORM
Tags : Best practices
Published : 2013-5-15
---

ORMs are trendy these days and it's so rare when someone mentions DDD or Repository without saying ORM (actually Entity Framework or NHibernate, but ORMs are popular with dynamic languages as well). The issue is the ORM is viewed like a magic tool and everyone (especially people starting to learn programming) seems to consider they are the present and the future.

 However, I think that people should be tempered a bit, as the use of ORM doesn't always simplify things, more often than not it actually complicates things.

 First of all, what is an ORM, what is its purpose? An ORM bridges the gap between the Object Oriented mindset (in fairness, lots of 'OO' code is not even remotely object oriented, but that's another story) and the relational database. This means that ORMs make sense only with a relational database which are still very common but not as popular as they once were.

 The truth is a lot of developers simply hate Sql or at least they don't like it very much. But with an ORM you can use a RDBMS without Sql! Instead of writing queries  you're just using objects. After all, it makes sense to say that a Category has a list of Posts, instead of the Post table having the foreign key category id. The ORM creates the illusion of a object oriented database without a trace of the Sql stuff (except the configuration part).

 But here where things get interesting. The developer doesn't care about the relational database and sql because it can use the ORM, but many devs forget this tiny detail: by abstracting the RDBMS the **ORM becomes itself the database.** It doesn't matter that we don't write sql anymore and we use objects directly. Those objects represent the relational database tables.

 In the case of CRUD apps, things are very simple as there are only data structures flying around. The minimum validation or some basic processing don't really change anything, we can't speak about a 'real' application with rich business logic. For this scenario the ORM is indeed a God sent tool. Incidentally, MANY examples and demos for ORMs are about this type of apps.

 But we have the 'advanced' applications. Where there is rich business behavior or the querying performance matters or both. Let's take the 'easy' case where performance matters. An ORM has an overhead which affects speed. It simply can't match manual or [data mapper ](https://github.com/sapiens/SqlFu)performance. Even more troublesome is the sql generated by the ORM. Entity Framework for example, is infamous for the convoluted sql it generates.

 But after all, an automated tool can't do very much, not in the case where you need to squize every ms of speed. For those cases you simply must have sql queries hand tweaked by an expert. And this means that one feature of the ORM goes away: abstraction of the database. The expert will write quite specific RDBMS sql, so changing the underlying db engine becomes cost prohibited. Hand tweaked also means that the ORM's own query language will be also ignored so, for this scenario the ORM just becomes an impediment. Of course, for very high reading performance chances are you're not using a RDBMS in the first place, so no ORM here either.

 The most tricky issue with ORMs is when dealing with **behavior rich applications**. It's true that some applications start as CRUD and evolve in behavior rich so that's understandable why the app was designed the CRUD way where ORM is central to everyting. But when you know the app is not CRUD and you have business objects which models business concepts and processes, then the little detail that the ORM is the database matters a lot.

 Because there is a mismatch between how a behavior rich business object looks and how the object that represents the database looks. A business object contains other business objects which can be entities or value objects. It's very complicated to map an object like this to some relational tables. And even if you manage to do it, the business objects contains ORM specific artifacts such as virtual properties, attributes or properties acting as foreign keys, all things that have no place in the business object. After all, the ORM entities represent the database, they are _objects who store state optimized for querying_, so **WHY** are you using the database directly in the Business Layer!? And why a business object should **EVER** care about lazy loading?! Is lazy loading a business concept?

 Oh and you will have to fight one big problem too: the unmaintainability of the code will raise exponentially with the rich behavior. You'll have a technical nightmare which can be avoided if you understand the ORM's place. Btw, CQRS helps a LOT when dealing with rich behavior.

 Every time you're using the ORM's entities as the base for the Domain entities, you're violating the **Separation of Concerns**(SoC) principle. You're simply mixing together the business logic and the persistence logic. Even worse with DDD is that you start from the ORM which models a relational schema after all and then try to fit business behavior on that structure. Business objects are not the same and they have not the same purpose as the database objects! One models behavior while other models how state is persisted and queried. Different concerns! When you are modifying business objects to care about the ORM, you are leaking persistence technical details into the Domain, hence the SoC violation.

 When modeling business objects, please FORGET that you're using an ORM. Use the [Repository pattern](http://www.sapiensworks.com/blog/post/2012/02/22/The-Repository-Pattern-Explained.aspx) to bridge the Business Layer to the Persistence Layer. The ORM is a persistence layer detail, so use it just for its intended purpose: as an object oriented database where state is stored in objects (entities) optimized for relational querying. If you don't care about relations just use a Document database or save the business object in serialized form.

 I know that in theory the ORM should map ANY object to relational tables, but let's be honest: that ORM Entity accurately represents one or more relational tables in the db. Their properties are mostly primitives which are easily mapped to RDBMS suported types. Any ORM entity is designed with the **relational database mindset**.

 For example, I'm curious how the tables and mappings for this object would look like

  

```csharp
public class MyObject
{
 /*some concrete types may have constructor dependencies */

  public ISomeInterface Property1 {get;set;}
  public List<IOtherInterface> Property2 {get;set;}
  public MyValueObject Value {get;set;}
}

```
  In conclusion, ORMs are GREAT for simple, CRUD applications. You don't want to use one if querying performance is very important and you have to be careful with it when dealing with behavior rich applications. Don't forget that the ORM follows a relational database mindset, while the Business Logic follows the actual Business mindset.
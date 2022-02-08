# Repositories Again?!?

Ok so let me just say here that I wrote the original repository chapter years ago (well at least years before I am writing this bit), and I have altered that chapter slightly too.

So why am I going over it again?

I am doing so because I quite often see people stumbling a bit even with the improved repository pattern shown in the previous chapter and I also have settled on a slightly different/improved approach, so lets cover some scenarios where the previous example (and normal generic repository pattern) can cause issues and then delve into the possible improved version.

## The original rant about Generic Repositories

Go check the previous chapter for more information but the key problems Generic Repositories had were:

- Implementation per model type
- Lots of implementations to maintain
- Potentially become dumping grounds with each new permutation of logic needed
- Can lead to circular dependencies or lots of duplicated logic

We showed how to sidestep the masses of repos by making a simpler generic approach, and how to cut down on the logic dumping ground and reuse issues by introducing `IQuery` to separate the querying mechanism out from the repository layer.

So with that in mind lets have a look at a common problem people still come up against and how we can solve it.

## Empty interfaces/classes to give better DI/Context

So this scenario can happen for a few different reasons but ultimately you end up seeing loads of empty repositories which just exist to make DI simpler, which looks something like:

```csharp
public interface IUserRepository : IRepository<User> {}
public class UserRepository : OurRepository<User> : IUserRepository
{
    // constructor guff
}

public interface ISkillRepository : IRepository<Skill> {}
public class SkillRepository : OurRepository<Skill> : ISkillRepository
{
    // constructor guff
}

// Loads more empty interfaces/classes for each type needed
```

So while this isnt the end of the world, it can easily bloat and it goes against the reason we did this in the first place which was to stop the **Implementation per model type** approach of common Generic Repository approaches.

## Why Have Generics?

So lets rewind for a moment, why do we need generics at all on our repository?

The simple answer is going to be so that we can tie our CRUD operations to our underlying generic types right? but what if I told you we could pretty much do this for free by just pushing the generic constraint to the method layer rather than the class layer?

If we did that we no longer need to have a type of repository per entity type, we just have one `IRepository` implementation and thats fit for all.

So what would that look like?

```csharp
public interface IRepository
{
    T Retrieve<T>(object key); // You can still add the key generic in
    void Create<T>(T item);
    void Update<T>(T item);
    void Delete<T>(T item);

    IEnumerable<T> Find<T>(IFindQuery<T> query);
    T Query<T>(IQuery<T> query);
}
```

That doesn't look too bad, but really if we think about this a bit more, we don't even really need any of these bits of logic in here, and if we think more about how we express these notions of CRUD, we could probably manage it all via queries.

## CRUD Via `IQuery` implementations

In most cases your crud logic in your repository will be delegating to some ORM which has the notion of entities, but even if it doesn't you have probably made your own wrappers to handle that anyway.

So if we look at a common implementation of CRUD logic, as per the previous chapter:

```csharp
public void Create(T item) => _connection_.Create<T>(item);
public void Retrieve(object id) => _connection.Get<T>(id);
public void Update(T item) => _connection.Update<T>(item);    
public void Delete(T item) => _connection.Delete<T>(item);
```

There is no reason why we cannot do this like so:

```csharp
public class CreateEntityQuery<T> : IQuery<T>
{
    public T Entity { get; }

    public CreateEntityQuery(T entity)
    { Entity = entity; }

    public T Execute(IDbConnection connection)
    { return connection.Create(Entity); }
}
```

Now imagine we do that for each one of the above CRUD options, cool we now have 4 queries each doing one of the default CRUD operations, which we can use like so:

```csharp
myRepository.Query(new CreateEntityQuery(myEntity));
```

> It should automatically pick up the generic for free on everything except for the `RetrieveEntityQuery` as that one you would be passing in the key not the type.

## Removing CRUD From Repository To Extension Conventions

Now we can just remove those CRUD endpoints from the repository and just add them in as nice extension methods for our repo like so:

```csharp
public static T Create<T>(this IRepository repository, T entity)
{ return repository.Query(new CreateEntityQuery<T>(entity)); }
```

Once we have wrapped each CRUD operation we can easily then just call it like so:

```csharp
myRepository.Create(myEntity);
```

Which looks no different to how it did before, but we now only need one `IRepository` implementation, which can just be injected in everywhere as `IRepository` and everything works as is.

> In most cases ORM frameworks handle key lookups for retrieval as `object` types anyway, but if you explicitly need to specify the type for your scenario you just need to add that into the extension/query layer when needed, it still not a massive problem.

## Summary

This whole approach makes it so we no longer need to inject multiple facade repositories into our service layer objects to do database querying, and it also makes it far more explicit to developers that **ALL DB LOGIC SHOULD BE IN IQUERIES**, so it stops the whole place becoming a dumping ground.

You can also more easily build custom conventions on top of this i.e `CreateMany(T[] entities)` or other soft deletion mechanisms etc. This also makes testing far easier as you can now just mock a single repository rather than several when testing your service layer.

> I have ended up using this approach and have found it far easier in the long run as there are no accidental dummy repository implementations for people to accidently put logic in, and you no longer need to do lots of manual DI for each type of repository, while finally also removing that issue of "I need to run a query but its not against a specific entity", where you just pick a random repository to use.

To clarify as well the end `IRepository` and `Repository` would look like this once all the above work has been done:

```csharp
public interface IRepository
{    
    T Query<T>(IQuery<T> query);
}
```

```csharp
public class DatabaseRepository : IRepository
{
    private IDbConnection _connection_;

    public DatabaseRepository(IDbConnection connection)
    { _connection_ = connection; }

    // All CRUD & Find are conventions pushed to extension methods now

    public TQuery Query(IQuery<TQuery> query) => query.Query(_connection_);
}
```

As you can see its super thin, you can add whatever logging etc you want in here and can express any conventions via extension methods, all while keeping everything clean and easily testable.

## Bonus Blurb - How Do I Test The Repository Extension Methods??!

So as this may catch a few people out I just want to mention that you cannot mock your repositories CRUD operations now as they are using extension methods, which cannot be mocked.

**"THAT'S RUBBISH, WHY DID YOU TELL US TO DO THIS!??!"** I hear you yell from afar, but worry not, you can still test super easily, you just have to mock at the query level like so:

```csharp
// Old way to mock (assuming Moq here, but just use your mocking fwk syntax)
var mockRepo = new Mock<IRepository<Foo>>();
mockRepo
    .Setup(x => x.Create(It.IsAny<Foo>()))
    .Returns(/*whatever*/);

// New way (again assuming Moq)
mockRepo
    .Setup(x => x.Query(It.IsAny<CreateEntityQuer<Foo>>()))
    .Returns(/*whatever*/);
```

As you can see its not a super big change, and if you need to mock multiple calls to same thing you just use `SetupSequence` or whatever sequential calling mechanism your mocking framework provides.
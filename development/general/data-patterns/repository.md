# Repository

This is more of a high level pattern which abstracts away data access and query concerns for data. So for example if you had a data store of items in your game and you wanteda way to be able to pull out a weapon with a specific name, or save a new item to the data store etc, a repository would allow you to do that in a more streamlined way.

## Opinionated rant

It is included here as a lot of people get it wrong. You can go google it now and almost every link to the repository pattern will show you something where they advocate you having an `IRepository<T>` then making your own implementation per type, such as `UserRepository : IRepository<User>` which would contain the standard `CRUD` based operations of a repository but with `User` specific methods like `GetAllActiveUsers`.

> When CRUD is mentioned in the context of repositories or data it is an abbreviation for `Create`, `Retrieve`, `Update`, `Delete`. These are the common actions that data interactions fall under so although you may use different named methods like `Get, Save, Update, Remove` you are basically adhering to the CRUD concept, so it is more a notion than a pattern.

I dislike this approach and I have a better approach which is more isolated, more reuseable and also more flexible. As some of the problems with the former approach is that you tend to keep dumping all your logic into your repositories, so your `UserRepository` goes from having the basic CRUD methods to having lots of business logic which will keep snowballing every time you want to add more query types.

### Whats Wrong With Existing Approaches?

As all the logic for getting data from the underlying data source (be it a database, in memory list, xml file etc) is contained at the **repository** level, if you want to have a method in another repository access the logic you need to have one repository access another, i.e.

```csharp
public class SkillsRepository : IRepository
{
   public GroupRepository GroupRepository {get;} // set via constructor? assuming no interface, should probably have one too

   public IEnumerable<Skills> GetAllSkillsFromGroup() { ... } // Call group repo to get users in group then extract all their skills
}
```

So this in best case causes interdependencies between repositories, and at worst case can cause circular dependencies which are going to cause you a big headace. You may be luck and you never need to share logic between repositories, or you may just copy the query logic over which isnt very **DRY** of you, but it will at least get the job done.

So the problems with the original touted design is:

- Implementation per model type
- Lots of implementations to maintain
- Potentially become dumping grounds with each new permutation of logic needed
- Can lead to circular dependencies or lots of duplicated logic

> So with this in mind lets look at a slightly modified approach which offers another perspective on the same problem and removes most of the negatives.

## Making the repository

Let's start off with making a repository interface which almost all examples will agree upon.

```csharp
public interface IRepository<TItem, TKey>
{
    TItem Retrieve(TKey key);
    void Create(TItem item);
    void Update(TItem item);
    void Delete(TItem item);
}
```

So before we go any further lets just go over the basics. The interface has 2 generic types, the first being the data object that is to be retrieved and stored, like your `Item`, `Player`, `Quest` etc, the second being the type of the key/identifier for this resource. 

> The key is sometimes omitted if you are not working purely with relational databases, so within here we will simplify and remove the key type generic but keep in mind you may need it.

So now we have shown the *vanilla* repository we will start to differentiate from the majority of other patterns and we add the notion of an Execute method and replace Get with Find method. We will also remove the key type as Find allows us more flexibility here.

```csharp
public interface IRepository<TItem>
{
    void Create(TItem item);
    void Retrieve(Guid id);
    void Update(TItem item);
    void Delete(TItem item);
    
    IEnumerable<TItem> Find(IFindQuery<TItem> query);
    T Query<T>(IQuery<T> query);
}
```

So as you can see here we now have a way to get a collection of `TItem` instances from the data source, as well as provide a way to execute some logic against the data source.

## Modelling the queries

The interfaces for the queries would look like:

```csharp
// Represents the most basic type of query
public interface IQuery<T>
{
    T Query(IDbConnection connection);
}

// Represents a query that returns a collection of data
public interface IFindQuery<T> : IQuery<IEnumerable<T>>
{
    IEnumerable<T> Query(IDbConnection connection);
}
```

As we are using a database as our example data store here (for simplicity), you may have a different kind of data store in your scenario such as a 3rd party API for storing data or a local file etc.

> There is a brief blurb near the end of this chapter on creating your own abstractions over your data source if its not a database.

## Implementing the repository

So currently we have got our `IRepository`, the query classes, and we understand the notion of a data source (`IDbConnection` in this example). So lets put it all together and see how it should be implemented.

> We are assuming some ORM such as Dapper or EF etc are added here to provide the CRUD extension on the `IDbConnection`

```csharp
public class DatabaseRepository<T> : IRepository<T>
{
    private IDbConnection _connection_;

    public DatabaseRepository(IDbConnection connection)
    { _connection_ = connection; }

    public void Create(T item) => _connection_.Create<T>(item);
    public void Retrieve(Guid id) => _connection.Get<T>(id);
    public void Update(T item) => _connection.Update<T>(item);    
    public void Delete(T item) => _connection.Delete<T>(item);

    public IEnumerable<T> Find(IFindQuery<T> query) => query.Query(_dataSource);
    public TQuery Query(IQuery<TQuery> query) => query.Query(_dataSource);
}
```

Now rather than having a repository for each type, we have a single repository based around the interaction with the data source. 

> In some scenarios such as using in memory data sources or external file system you may need to manually save the changes after each step or knowingly persist the changes at a later point. For databases there is also the `Unit Of Work` pattern which lets you wrap up a few things together and rollback if it fails.

#### Example usage

Anyway so lets do a quick use case for the above code we wrote:

```csharp
var ourDbConnection = // imagine this is connected;
var userRepository = new DatabaseRepository<User>(ourDbConnection);

var getActiveUsersQuery = new GetActiveUsersQuery();
var activeUsers = userRepository.Find(getActiveUsersQuery);
```

## Implementing find queries

Now the above example is just whimsical but if you imagine there is a user model, and we need to get all active users, we can express that specific query within the `GetActiveUsersQuery` which is a type of `IFindQuery`. Let's do an imaginary implementation of this query to show how it would work:

```csharp
public class GetActiveUsersQuery : IFindQuery<User>
{
    public IEnumerable<User> Query(IDbConnection connection) 
    {
        // Again we assume some ORM provides us the Query<T> extension here
        return connection.Query<User>(x => x.IsActive);
    }
}
```

Again we are just making up our `User` class, but this shows how we have isolated our query logic into this specific class.

As mentioned earlier in the vanilla implementation of a respository this logic resides within the repository, so you would often end up having to write more and more methods to expose this logic, so using this notion of queries which wrap up the query concerns and are re-useable you can keep your repository objects lightweight and push the business logic queries into their own classes.

There are two main benefits here, one is that you do not have to store the underlying data source, it is passed to you by the executor (repository in this instance), so you separate your concerns in a nice way. The other benefit is that you can pass in arguments to the classes without much issue, so you could easily write something like:

```csharp
public class PredicateFindQuery<T> : IFindQuery<T>
{
    private Func<T, bool> _predicate;
    
    public PredicateFindQuery(Func<T, bool> predicate)
    {
        _predicate = predicate;
    }
    
    public IEnumerable<T> Query(IDataSource<T> dataSource) 
    {
        return dataSource.Query<T>(_predicate);
    }
}
```

This allows you to just write any old predicate you want to query into the data source. I would probably still advocate making specific queries to represent your logic so it shows intent. 

However if you just want to get on and do some testing of queries or just don't want to have to keep instantiating query objects just make it a public property and off you go.

## Implementing more specific IQuerys

Now we have not discussed the `IQuery` type yet. So where the `FindQuery` is mainly there to get a readonly collection of matching results for a repository. `IQuery` is there to provide an all purpose query mechanism to get or alter data in some way, so this provides us a way to write data, so lets go over that with a quick example.

```csharp
public class BanAllCheatersQuery : IQuery<int>
{
    public int Query(IDbConnection connection)
    {
        var sqlQuery = "UPDATE users SET IsActive = false WHERE HasCheated = true"; 
        var numberOfCheatingUsers = dataSource.Query(sqlQuery);
        return numberOfCheatingUsers;
    }
}
```

So it would update all the cheating users to be inactive and return back how many rows have been effected to the consumer.

> One thing to note here also about the `IQuery` pattern is that it allows you to use whatever approach/logic you want to interact with your data source, so you can do as much or as little as you want within these things, and its all decoupled, encapsulated and reusable across any repository

## More information

This has been a large block on the pattern and although the above use case focuses on databases you may need to alter the pattern slightly to fit your own scenarios, as even databases are not all relational these days, but the key thing is that we wrap up any data specific communication/logic into queries and have 1 implementation of what the repository should do.

> It is worth noting this is a brief blurb on the downsides to the default Generic Repository Pattern often shared online, there are a few other things that you can do to make things even more succinct but they can be for another chapter.

## A brief detour on data source abstraction

In the real world the data source could be anything from a database to a 3rd party API, even a XML/JSON file. So while almost all example of `IRepository` online are for database interactions we can use them to express any data abstraction.

In this example we will assume you have your game data in some files, be it XML/JSON/Binary and you read it into a big list which will be in memory for the duration of the game, so the `IDataSource` will wrap this.

It would look something like:

```csharp
public interface IDataSource<T>
{
    IList<T> DataItems {get;}
    
    public void SaveChanges();
}
```

Then our actual implementation for our in-memory list of objects:

```csharp
public class InMemoryDataSource<T> : IDataSource<T>
{
    private readonly IList<T> _entries;

    public InMemoryDataSource()
    { _entries = new List<T>(); }

    public InMemoryDataSource(IList<T> entries)
    { _entries = entries; }

    public IList<T> DataItems => _entries;
    }
    
    public void SaveChanges()
    {
        // do something like serialize back out
    }
}
```

> If you wanted to here you could expose methods for querying the data to make it more like a database, but we will try to keep it all simple for now so you are able to just see the high level picture, then customize the underlying classes and interfaces to suit your scenario.

So as you can see you can wrap your data source however you want, as long as you provide a way for the repository to access it you are golden
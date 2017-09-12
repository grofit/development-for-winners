# Repository

This is more of a high level pattern which abstracts away data access and query concerns for data. So for example if you had a data store of items in your game and you wanteda way to be able to pull out a weapon with a specific name, or save a new item to the data store etc, a repository would allow you to do that in a more streamlined way.

## Opinionated rant

It is included here as a lot of people get it wrong. You can go google it now and almost every link to the repository pattern will show you something where they advocate you having an `IRepository<T>` then making your own implementation per type, such as `UserRepository : IRepository<User>` which would contain the standard `CRUD` based operations of a repository but with `User` specific methods like `GetAllActiveUsers`.

> When CRUD is mentioned in the context of repositories or data it is an abbreviation for `Create`, `Retrieve`, `Update`, `Delete`. These are the common actions that data interactions fall under so although you may use different named methods like `Get, Save, Update, Remove` you are basically adhering to the CRUD concept, so it is more a notion than a pattern.

I dislike this approach and I have a better approach which is more isolated, more reuseable and also more flexible. As some of the problems with the former approach is that you tend to keep dumping all your logic into your repositories, so your `UserRepository` goes from having the basic CRUD methods to having lots of business logic which will keep snowballing every time you want to add more query types.

## Making the repository

Let's start off with making a repository interface which almost all examples will agree upon.

```csharp
public interface IRepository<TItem, TKey>
{
    TItem Get(TKey key);
    void Save(TItem item);
    void Update(TItem item);
    void Delete(TItem item);
}
```

So before we go any further lets just go over the basics. The interface has 2 generic types, the first being the data object that is to be retrieved and stored, like your `Item`, `Player`, `Quest` etc, the second being the type of the key/identifier for this resource. 

> The key is sometimes omitted if you are not working purely with relational databases, so within here we will simplify and remove the key type generic as in the game world you may be working more with in memory of flat file style databases.

So now we have shown the *vanilla* repository we will start to differentiate from the majority of other patterns and we add the notion of an Execute method and replace Get with Find method. We will also remove the key type as Find allows us more flexibility here.

> There is also the notion of a `HybridQuery` but I will leave that out and explain it later on so you can decide if you want to add it or not.

```csharp
public interface IRepository<TItem>
{
    void Save(TItem item);
    void Update(TItem item);
    void Delete(TItem item);
    
    IEnumerable<TItem> Find(IFindQuery<TItem> query);
    
    void Execute(IExecuteQuery<TItem> query);
}
```

So as you can see here we now have a way to get a collection of `TItem` instances from the data source, as well as provide a way to execute some logic against the data source.

## Modelling the queries

The interfaces for the queries would look like:

```csharp
public interface IFindQuery<T>
{
    IEnumerable<T> Find(IDataSource dataSource);
}

public interface IExecuteQuery<T>
{
    void Execute(IDataSource dataSource);
}
```

This may seem a little confusing as we have not discussed the `IDataSource`, this is an abstracted notion of how you access the data. If you are using databases you could easily replace this with `IDBConnection` which would abstract away the database. Given in most cases game data is read into memory you would probably just have this as a wrapper around the list of data in memory, however you could make it abstract the file system if you wished it to do manual file reads/writes.

## Abstracting the data store/source

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
 
        public InMemoryDatabase()
        { _entries = new List<T>(); }

        public InMemoryDatabase(IList<T> entries)
        { _entries = entries; }

        public IList<T> DataItems 
        {
            get { return _entries; }
        }
        
        public void SaveChanges()
        {
            // do something like serialize back out
        }
}
```

> If you wanted to here you could expose methods for querying the data to make it more like a database, but we will try to keep it all simple for now so you are able to just see the high level picture, then customize the underlying classes and interfaces to suit your scenario.

## Implementing the repository

So currently we have got our `IRepository`, the query classes, the `IDataSource` interfaces and implementations, so now lets look at making a repository instance and then using it with some queries.

```csharp
public class InMemoryRepository<T> : IRepository<T>
{
	private IDataSource<T> _dataSource;
	
	public DatabaseRepository(IDataSource<T> dataSource)
	{ _dataSource = dataSource; }

    public void Save(TItem item)
    { 
        _dataSource.DataItems.Add(item);
        _dataSource.SaveChanges();
    }
    
    public void Update(TItem item)
    {
        // Method only saves
        _dataSource.SaveChanges();
    }
    
    public void Delete(TItem item)
    { 
        _dataSource.DataItems.Remove(item); 
        _dataSource.SaveChanges();
    }
	
	public IEnumerable<T> Find(IFindQuery<T> query)
	{ return query.Query(_dataSource); }

	public void Execute(IExecuteQuery<T> query)
	{ 
	    query.Query(_dataSource); 
	    _dataSource.SaveChanges();
	}
}
```

Now rather than having a repository for each type, we have a repository based around the interaction with the data source. So as shown above we don't need to `Update` the item instance as all items will be reference types (in this scenario), so you changing an item would automatically update the in-memory version. 

> One of the things you may have noticed in the above example is that we are saving our changes after every interaction. This is fine for now, however further down the line you may want to only save after a set of changes have occurred, like a database transaction. In this scenario you could easily make a new implementation of `IRepository` which doesn't save automatically, and then you can have the transaction handler manage the saving. This is also known as a **Unit Of Work** pattern, which we will look into later. As if you were to be using a `FileSystemRepository` where the `IDataSource` is a file system handle, you would need to manually update the file system, or an actual database every time a change occurred, which is going to be costly for performance.

#### Example usage

Anyway so lets do a quick use case for the above code we wrote:

```csharp
var aLotOfUsersFromAFile = // imagine this is populated;
var userDataSource = new InMemoryDataSource<User>(aLotOfUsersFromAFile);

var userRepository = new InMemoryRepository<User>(userDataSource);

var getActiveUsersQuery = new GetActiveUsersQuery();
var activeUsers = userRepository.Find(getActiveUsersQuery);
```

## Implementing find queries

Now the above example is just whimsical but if you imagine there is a user model, and we need to get all active users, we can express that specific query within the `GetActiveUsersQuery` which is a type of `IFindQuery`. Let's do an imaginary implementation of this query to show how it would work:

```csharp
public class GetActiveUsersQuery : IFindQuery<User>
{
    public IEnumerable<User> Query(IDataSource<User> dataSource) 
    {
        return dataSource.DataItems.Where(x => x.IsActive);
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
        return dataSource.DataItems.Where(_predicate);
    }
}
```

This allows you to just write any old predicate you want to query into the data source. I would probably still advocate making typed queries to represent your logic so it shows intent. However if you just want to get on and do some testing of queries or just don't want to have to keep instantiating query objects just make it a public property and off you go.

## Implementing execute queries

Now we have not discussed the `ExecuteQuery` type yet. So where the `FindQuery` is there to get a readonly collection of matching results, `ExecuteQuery` is there to alter data in some way, so this provides the write concern to the find's read concern. so lets go over that with a quick example.

```csharp
public class BanAllCheatersQuery : IExecuteQuery<User>
{
    public void Query(IDataSource<User> dataSource)
    {
        var allCheatingUsers = dataSource.DataItems.Where(x => x.HasCheated && x.IsActive);
        
        foreach(var cheater in allCheatingUsers)
        {
            cheater.IsActive = false;
        }
    }
}
```

So it would retrieve all active cheats, then disable them. This approach can be done for updating sets of data without having to do it for each individual set.

## More information

This has been a large block on the pattern and although the above use case would work fine for most game development scenarios in the web/app world you would probably end up dealing with databases more be it relational or document/graph etc, so in those cases you may need to change around how you abstract away certain parts.

It is entirely possible to make it so abstracted and generic that you could cope with almost any scenario and underlying connections etc, however in most cases its a pointless endevour and you should really only cater to what you would expect to use, and in the context of game development the above should serve you well enough.

Now we didn't cover Hybrid queries and they are not really **needed** as such but the notion is that you provide a way to do a query which returns a defined type, so if you wanted to just select a small part of a data model, or get all names of active users without their entire user model, or something like:

```csharp
public interface IHybridQuery<TInput, TOutput>
{
    TOutput Query(IDataSource<TInput> dataSource);
}

public class GetUserMetaDataQuery : IHybridQuery<User, IDictionary<string, string>>
{
    public IDictionary<string, string> Query(IDataSource<User> dataSource)
    {
        return dataSource.DataItems.Select(x => x.Metadata);
    }
}
```

This would need a new method on the repository, but this allows you to have more flexibility in how you get your data back if you find you need to reduce the data chatter between components.

You can also look at adding the `Get(TKey key)` method back in if you have some notion of a keyed value, this can make it easier to get individual models with an Id. In most cases this requires your models to have an interface describing the key though so I know in game development a lot of people do not bother unless it is going to a database.

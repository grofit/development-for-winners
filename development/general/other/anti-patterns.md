# Anti Patterns

Now that we have covered some good patterns and approaches it seems useful to add this section to deal with patterns which seem at face value like a good idea, but in reality are not such a great idea in the long run. This is not to say that you should NEVER use these patterns, but more a case of think about if the problem you are trying to solve could not just be solved in a better way.

> Rather than go in depth on all of them we will just give "a quick rundown" over what they are, why they are seen as good, and why they cause issues.

## Static classes

```csharp
// Some static class
public static EventSystem
{
    public static void Publish<T>(T message) 
    { 
        // ...
    }

    public static void Subscribe<T>(Action<T> subscriber)
    {
        // ...
    }
}

// Anywhere you like
EventSystem.Publish(new SomeMagicMessage());
```

### Why its seen as good

So static classes basically provide you a class which always exists and can be accessed anywhere so no need to instantiate, this means no need to pass instances into constructors or have to worry about different implementations, there is only ever one, its always there.

### Why its actually bad

Because you cannot change it.

Seriously...

Ok so lets look at a scenario where you want to unit test some class which uses `EventSystem`, how do you mock out the dependency? you can't right... and what if you are using multi threaded code... uh oh your static class may blow up if you are not accounting for threading. 

You lose a lot of control and configurability over how your logic is run when you depend upon logic in static classes (extension methods to some extent can fall into the same trap), as you are not able to change the implementation without changing your source code, this impedes testing as well as overall design.

### A better way?

First of all use **IoC** and inject in your dependencies, this way you can change the actual implementation whenever you want, it makes your code more testable and more configurable, which ultimately makes it more re-useable. If you pair this with **DI** you also are in a position to set your object to act like a singleton which is basically the same as a static class anyway without any of the design drawbacks of explicitly depending on a static class.

## Singleton classes

```csharp
// Some interface
public interface IEventSystem
{
    void Publish<T>(T message);
    void Subscribe<T>(Action<T> subscriber);
}

// Some singleton class
public class EventSystem : IEventSystem
{
    private static EventSystem _instance = new EventSystem();

    private EventSystem() {}

    public static EventSystem Instance => _instance;

    public void Publish<T>(T message) 
    { 
        // ...
    }

    public void Subscribe<T>(Action<T> subscriber)
    {
        // ...
    }
}

// Anywhere you like
EventSystem.Instance.Publish(new SomeMagicMessage());
```

### Why its seen as good

A singleton is basically a fancy static class but with some additional benefits, like it can implement an interface because its not a static class, it can also be passed around as a varaible if needed as its both a static and an instance. It also enforces that there is only one instance of it in existence just like a static class, all other benefits are same as a regular static class.

### Why its bad

Same reasons a static class is bad.

### A better way

Same solution as a static class, create it as a **DI** singleton so its lifetime is configurable and more testable.

### Generic Repository Dumping Ground (aka Generic Repository)

```csharp
// Some generic repository interface
public interface IRepository<T>
{}

// Some repository implementing methods for Users
public class UserRepository : IRepository<User>
{
    public User CreateUser(SomeGuff guff) { /*...*/ }
    public User UpdateUsersEmailButNothingElse(Guid id, string email) { /*...*/ }
    public IEnumerable<User> GetAllUsersWithMiddleNameJames() { /*...*/ }
    public IEnumerable<User> GetActiveUsers() { /*...*/ }
    // Repeat as many specific bits of logic as you need
}
```

### Why its seen as good

Its putting all responsibilities for the type (`User` in this case) into one class, **SRP** to the max! you can inject this in anywhere you need to do something against a user and you know exactly where to put your logic into when you need to do more stuff related to that type.

### Why its bad

Where do you draw the line? it just becomes a massive dumping ground for any and all CRUD logic related to that entity, you also run into the common problem of *"OH NO, MY `UserGroupRepository` NEEDS TO ACCESS THE `UserRepository` TO DO SOME STUFF"* then you end up with repositories including other repositories just to access a couple of methods, then you end up making it harder to test, and each repository has LOADS of tests because it has to test every method you have come up with.

### A better way?

Look at the **repository** section, the generic repository pattern does not need to be bad if you make generic CRUD operations and then put any specific guff like `GetAllUsersWithMiddleNameJames` into their own `IQuery` classes. This allows you to re-use queries within other queries/repositories/classes without including the whole repository and also makes things more testable/mockable as you are testing most parts in isolation.

It may seem like more of a faff up front but in the long run it can pay dividends and also improve your re-use without causing a massive spaghetti dependency nightmare where all repositories end up communicating to share logic.

## Service Location

There is no point mentioning it too much here as its already mentioned in the **service location** topc, however its worth noting here that its bad, and you should probably use **Constructor Injection** via **DI** instead wherever possible.
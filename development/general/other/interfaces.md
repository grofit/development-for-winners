# Interfaces

I dont know who needs to read this, I hope no one, but if you do here it is.

**INTERFACES ARE CONTRACTS**

**INTERFACES ARE NOT BEHAVIOURS**

> Imagine me yelling this at the top of my voice

If you already knew this, then skip this whole chapter, you are a good one, thank you, on behalf of the whole technical industry.

## What Are You On About?

Over my years drifting through the technical world like some wandering mercenary for hire, I have come across quite a few people who have absolutely no idea why interfaces exist and why we should use them.

So as apparently its not common knowledge to everyone, interfaces have a few great use cases:

### Providing a CONTRACT for behaviour

So the first great use case is the most important as this basically allows us to provide a contract for expected inputs/outputs from any implementations.

Now this comes with some handy benefits over using a class directly.

- We can mock the interface without any side effects in tests to satisfy dependencies
- We can provide as many implementations of that behaviour as we like
- We can access implementations without having a hard dependency on them
- Interface hides any underlying implementation details that may be public (generally for testing)
- Can be more easily proxied in AoP scenarios

The first one is amazeballs though, without interfaces its VERY painful trying to mock things in tests, as you will always be subject to a class' constructor even when trying to mock unless you do a LOT of hackery.

> The key take away here though is that the interface removes a lot of coupling issues on dependencies as well as giving MUCH GREATER flexibility going forward, while finally making testing much easier.

### Providing CONVENTIONS to build off

This one is not quite as apparent as the first but is a cool use case for an interface, so while all interfaces are going to provide a contract, you can also use them as a convention to build upon.

For example lets look inside the .net framework for a use case of this.

```
IEnumerable<T> -> ICollection<T> -> IList<T> -> List<T>
```

So the first interface provides a contract for a lazy iterator, but is also a convention for building off for other more specific conventions.

> You may be thinking "whoa this sounds a lot like inheritance, and you said that was bad somewhere else in here", which is true, but as we are not inheriting behaviours and just contracts its ok as we are constantly building off the convention.

The second interface takes the first convention but adds the notion of a `Count` property which lets you get the count without iterating, the 3rd interface builds off this by adding an accessor (`[]`) to the convention so you can use it like an enumerable, but also like an array, then finally we have the implementation which implements ALL of the conventions in the chain.

> There are loads of other conventions here like `IDictionary` and `IReadOnlyCollection` etc which all build off same basics but add slightly different use case conventions on top. The key thing though is that we can treat this object like any of the downstream contracts easily so a `List<T>` can be passed to anything that needs `IEnumerable<T>`, `ICollection<T>` or `IList<T>`.

### Providing ASPECTS for composite contracts

While .net doesnt support true mixins like other languages do, you can mimic some of that behaviour by using interfaces as aspects of a larger behaviour.

So for example if you were making an RPG game and you wanted to express things that had a name on their head you could make an `IHasName` interface, and anything with that interface would get a name on their head, or an `IUsable` would let the player interact with this object.

The usages here can be as granular as you like but rather than having one large interface describing a lot of this stuff you can break it down a bit more like so:

```csharp
public interface IGameEntity : IHasName, IHasInventory, IIsMovable, IHasStats {}
public interface IPlayerEntity : IGameEntity, IIsPlayerControlled {}
public interface INpcEntity : IGameEntity, IIsAIControlled {}
```

This kinda touches on the 2nd point of conventions, but as you can see you can have really granular interfaces which express each **aspect** of what behaviour an object has, but can be exposed via composition for higher level interfaces.

> It is worth mentioning here though that if you were going down this road and end up with LOADS of these sort of granular interfaces you may be better off looking at an ECS style pattern which lets you do this in a far more dynamic way.

### Interface Specificity

One final thing worth mentioning which can be useful for interfaces is that you can often wrap up a faffy long winded interface in a higher level more specific one which may make things more usable for your needs.

For example lets say we have a lovely repository like:

```csharp
public interface IRepository<T,K>
{
    // ...    
}
```

To consume that we would need to put lots of `IRepository<MyModel, int>` and `IRepository<SomeOtherModel, string>` etc, which is fine but may be unsightly, or there may even be scenarios where you need to use the same generic twice for some reason like `IRepository<string, int>` which can be problematic as if there are more than one implementation, which one do you need?

So to solve this we can add another interface layer on top for end consumption like this:

```csharp
public interface IMyModelRepository : IRepository<MyModel, int> {...}
public interface ISomeOtherModelRepository : IRepository<SomeOtherModel, string> {...}
public interface ILocaleRepository : IRepository<string, int> {...}
public interface IKeywordRepository : IRepository<string, int> {...}
```

As you can see we can still use them as if they are the underlying `IRepository<T,K>` type but we have a nicer more specific high level interface wrapping it up, which should then be used as the implementation contract.

> This also can make your **DI** configuration seem a bit more sane such as `container.Bind<ILocaleRepository>().To<LocaleRepository>()`, however even if there is only 1 **REAL** implementation under the hood you still need 1 facade wrapper class per interface type here, which is a downside to this approach.

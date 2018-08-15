# Dependency Injection

## What is it?

Simply speaking its a way of resolving class dependencies using large container of type resolvers. There are plenty of **DI** frameworks for almost every modern programming language, and as we are focusing on **C#** there are plenty of frameworks like **Ninject**, **AutoFac**, **Unity** (Microsoft) and many more. However in the context of game dev (Unity) there are not that many which are supported, so the main options are **Zenject** and **StrangeIoC**, although for this example we will use **Ninject** here purely because it is slightly more lightweight, and there are lots of docs for it.

> You can find all the documentation and information on their websites:
- [**Ninject**](https://github.com/ninject/ninject/wiki)
- [**AutoFac**](https://autofaccn.readthedocs.io/en/latest/)

At a high level almost all dependency injection frameworks share the same principals, so although we are using a specific framework here you can easily apply the same principals even if the syntax is different. So lets begin with a simple binding container file, this is where you often put all your setup bindings.

### Lifecycle

Generally in the **DI** world you will have the same life cycle for your objects:

- Binding
- Resolving
- Activation
- Deactivation

So to begin with you will **bind** all your object telling them how they can be **resolved** then once this has been done you are often able to provide additional logic in terms of any custom code to run when an object is **activated** (instantiated) and when it is **deactivated** (disposed). Not all **DI** frameworks call these steps the same, i.e **Bind** may be known as **Register**, or **Resolve** may be known as **Get** etc but generally the syntactical differences mean little, its still the same thing.

Activation and Deactivation are not super important to know about as generally you wont be doing much in this area, but its worthwhile knowing that the concept exists, you also can build upon this and do things like **AOP** which we will discuss later.

### Binding Setup

So to begin with **Ninject** has the notion of a **MonoInstaller** which is where it contains all binding setup.

```csharp
using Ninject;

public class MyInstaller : NinjectModule
{
    public override void Load()
    {
        Bind<ISomething>().To<SomeImplementation>();
        Bind<ISomethingElse>().To<SomethingElse>();
    }
}
```

Now as you can see we are binding the type `ISomething` to the class `SomeImplementation`, you can also use `typeof(T)` as an argument instead of the generic method used above. I am just making up the scenarios but hopefully you can visualize the interface and the class which implements it, in-case not here is how it would look:

```csharp
public interface ISomething {}
public class Something : ISomething {}
```

So this is a common binding scenario where you take an interface and bind it to a class *(often known as a Concrete Class in this context)*, which will mean that if I were to have a class like so:

```csharp
public class SomeClassWithDependency
{
	private ISomething _something;

	public SomeClassWithDependency(ISomething something)
	{
		_something = something;
	}
}
```

The dependency framework knows how to resolve the `ISomething` class for you, as we told it whenever you see `ISomething` pass it a `Something`. As mentioned almost all **DI** frameworks has the notion of this, and it is usually known as *Bind* or *Register*.

#### Object Scoping (Transient, Singleton etc)
Now we have covered the HOW of binding, lets look at how you can improve the binding lifetime of objects. So the above example will be known as **Transient** which means it will basically create a new instance of `Something` for every `ISomething` resolved. So for example if we had 3 classes with a dependency on `ISomething` there would be 3 instances of `Something` created. This may be fine however in some situations you may want to have only 1 instance of a given class.

```csharp
Bind<ISomething>().To<Something>().InSingletonScope();
```

So this now will provide us only a single instance of `Something` for every dependency. This here allows us to have an object which acts like a **singleton** but without any of the downsides. As mentioned in the previous chapters there is rarely a need for **singleton** and **static** classes explicitly when you use **DI** correctly, making your classes a lot less coupled and far easier to test as this is moved to be a configuration concern.

#### Binding to Instances/Self
Another relevant binding scenario would be to an instance, which is not used that often but in some scenarios where you need to do some complex setup to create an instance, which would look like:

```csharp
var something = new Something(); // or some complex setup
Bind<ISomething>().ToConstant(something);
```

This would pass the instance you have created around rather than letting the **DI** framework handle the creation. Almost all **DI** frameworks have the above notions.

You can also bind a class to itself, which is mainly used for **Concrete Classes**.

```csharp
Bind<ConcreteClass>().ToSelf(); // Just use itself and sort the dependencies
```
> There are far more binding scenarios, some are specific to the game dev world (such as binding to **prefabs**, **methods**, **gameobjects** with **Zenject**) an some are specific to the web dev world, ultimately you can read up more on this for each DI framework on their sites.

### Skill Transference
So although this is all **Ninject** specific, the things learnt here can apply to other frameworks and platforms. For an example here is how to do common things in:

#### Using Zenject (Unity 3d)

```csharp
using Zenject;

public class MyInstaller : MonoInstaller
{
    public override void InstallBindings()
    {
        Container.Bind<ISomething>().ToTransient<SomeImplementation>();
        Container.Bind<ISomethingElse>().ToTransient<SomethingElse>();
        Container.Bind<ISomethingMore>().ToInstance(new SomethingMore())
    }
}
```

#### Using AutoFac

```csharp
using AutoFac;

public class MyModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        builder.RegisterType<SomeImplementation>().As<ISomething>(); // Notice how its other way around
        builder.RegisterType<SomethingElse>().As<ISomethingElse>().InstancePerDependency();
        builder.Register(c => new SomethingMore()).As<ISomethingMore>();
    }
}
```

As you can see although the syntax is slightly different it is still doing the same thing under the hood.

## Resolving Bindings

As mentioned earlier in almost all cases you will **ALWAYS WANT TO USE CONSTRUCTOR INJECTION** which is done automatically for you assuming you have adhered to **ioc**. There are however other scenarios you may need to handle, such as property injection or in the game dev world scene/gameobject injection (These will be discussed more in the game dev specific sections).

> You almost always want to use **constructor injection** because it means your objects are unaware of the DI framework, i.e if you want to use property injection you often have to put an attribute on your property, and this property means you have to add `using SomeDIFramework` which makes that model and everything that uses it dependent upon a specific DI framework. You ideally do not want to fall into this trap as it is like a virus that spreads.

Once you have setup how dependencies should be resolved you now need to get an instance of the type you need from the container (term for whatever stores all your bindings). This can be handled different ways depending on your scenario, for example in ASP MVC (web world) you would end up loading a bootstrapper which does the resolving for you, so you never need to manually resolve anything, however lets pretend we do need to manually resolve stuff.

### Manually Resolving Instances

Lets just fast forward a minute and say "this is how you resolve objects":

```csharp
var somethingImplementation = container.Get<ISomething>();
```

Now that we have that out of the way lets rewind and look a bit more in depth as to what is happening from start to finish.

> This is not 100% what is happening as the actual reflection may sometimes happen up front, sometimes at resolve time, and also in some cases it may have other metadata around how it should handle bindings, but for all intents and purposes this is accurate enough for you to get your head around what is happening under the hood.

```csharp
// Our class in some file
public class SomeClassWithDependency : ISomeClassWithDependency
{
	private ISomething _something;

    // We have a dependency in ISomething
	public SomeClassWithDependency(ISomething something)
	{
		_something = something;
	}
}

// In our module we start binding
Bind<ISomething>().To<Something>();
// 1. Get type ISomething
// 2. Track that it has an implementation for type Something

Bind<ISomeClassWithDependency>().To<SomeClassWithDependency>();
// 3. Get type ISomeClassWithDependency
// 4. Track that it has an implementation for type SomeClassWithDependency

// This lives in some file where you need an instance of ISomeClassWithDependency
var someInstanceWithDependenciesMet = container.Get<ISomeClassWithDependency>();
// 5. Get type ISomeClassWithDependency from the binding information on the container
// 6. Get the default implementation type bound for ISomeClassWithDependency
// 7. Get the constructor/s for SomeClassWithDependency
// 8. Check what dependencies the constructors require (in this case ISomething)
// 9. Go back to step 5 for each dependency (this is looping through creating the dependency tree)
// 10. Once all dependencies are met instantiate and return implementation for ISomeClassWithDependency
```

Now that may seem like a lot of steps but its actually quite simple and there is not much magic happening, it is just analyzing the dependency tree for what you need and sourcing all the dependencies ahead of time and returning you an object with everything built.

In most real world scenarios you may have very large trees as the more you adhere to good design practices (i.e composition over inheritance, ioc etc) you will end up having lots of smaller objects that will all be generated automatically for you as and when they are needed.

As we mentioned a while ago there are also the notion of scopes/lifetimes to factor in here, so if lets say we had done `Bind<ISomething>().To<Something>().InSingletonScope()` the container would only instantiate the implementation for `ISomething` once and then every time it was needed (be it directly or as a dependency in another class) it would just return back that existing implementation. This basically allows you to have singleton style instances without coding it as a singleton (see **anti-patterns** for topic on why singletons are iffy).

### Auto Resolving

In most real world use cases you wont need to resolve anything manually as most large frameworks in web/app world have bootstrapper libraries which will automatically resolve your classes for you under the hood, for example in [ASP MVC you can bootstrap Ninject](https://github.com/ninject/Ninject.Web.Mvc) which will automatically let all your bound `Controller` classes be registered within MVC so all you need to handle is the binding aspect. Same sort of thing in Unity with **Zenject** you just give it a project/scene context and your installers and off it goes, automatically resolving all your parts for you.
## Dependency Injection

### What is it?

Simply speaking its a way of resolving class dependencies using large container of type resolvers. There are plenty of **DI** frameworks for almost every modern programming language. As we are focusing on **C#** there are plenty of frameworks like **Ninject**, **AutoFac**, **Unity** (Microsoft) and many more. However in the context of Unity there are not that many which are supported, so the main options are **Zenject** and **StrangeIoC**, although for this example we will use **Zenject** here purely because it is slightly more lightweight, as **Strange IoC** has a bit of conventions that come with it.

> You can find all the documentation and information about **Zenject** on their website, http://strangeioc.github.io/strangeioc  and you can find further information about **Zenject** on their website, https://github.com/modesttree/Zenject

At a high level almost all dependency injection frameworks share the same principals, so although we are using a specific framework here you can easily apply the same principals even if the syntax is different. So lets begin with a simple binding container file, this is where you often put all your setup bindings.

#### Binding Setup

So to begin with **Zenject** has the notion of a **MonoInstaller** which is where it contains all binding setup.

```csharp
using Zenject;
using UnityEngine;
using System.Collections;

public class GameInstaller : MonoInstaller
{
    public override void InstallBindings()
    {
        Container.Bind<ISomething>().ToTransient<SomeImplementation>();
        Container.Bind<ISomethingElse>().ToTransient<SomethingElse>();
    }
}
```

Now as you can see we are binding the type `ISomething` to the class `SomeImplementation`, you can also use `typeof(T)` as an argument instead of the generic method used above. Now I am just making this up but hopefully you can visualize the interface and the class which implements it, in-case not here is how it would look:

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

Now we have covered the HOW of binding, lets look at how you can improve the binding lifetime of objects. So the above example will be known as **Transient** which means it will basically create a new instance of `Something` for every `ISomething` resolved. So for example if we had 3 classes with a dependency on `ISomething` there would be 3 instances of `Something` created. This may be fine however in some situations you may want to have only 1 instance of a given class.

```csharp
container.Bind<ISomething>().ToSingle<Something>()
```

So this now will provide us only a single instance of `Something` for every dependency. This here allows us to have an object which acts like a **singleton** but without any of the downsides. As mentioned in the previous chapters there is rarely a need for **singleton** and **static** classes when you use **DI** correctly, making your classes a lot less coupled and far easier to test.

Finally another relevant binding scenario would be to an instance, which is not used that often but in some scenarios where you need to do some complex setup to create an instance, which would look like:

```csharp
var something = new Something(); // or some complex setup
container.Bind<ISomething>().ToInstance(something);
```

This would pass the instance you have created around rather than letting the **DI** framework handle the creation. Almost all **DI** frameworks have the above notions.

> There are far more binding scenarios specific to **Zenject** such as binding to **prefabs**, **methods**, **gameobjects**, **getters** and far more, all available within the documentation on the site.

### Skill Transference
So although this is all **Zenject** specific the things learnt here can apply to other frameworks outside of unity. For an example here is how to do everything we have done above in **Ninject**.

```csharp
using Ninject;

public class GameModule : NinjectModule // Ninject's container for bindings, same as our MonoInstaller
{
    public override void Load() 
    {
        Bind<ISomething>().To<Something>(); 					// same as our Bind<ISomething>().ToTransient<Something();
        Bind<ISomething>().To<Something>().InSingletonScope();  // same as our Bind<ISomething>().ToSingle<Something>();
		Bind<ISomething>().ToConst(new Something());			// same as our Bind<ISomething>().ToInstance(new Something());
    }
}
```

> I am sure you really don't care about Ninject but it is worth while seeing how easy it is to apply what we are learning here.

### Resolving Bindings

So it is all well and good us setting up all these bindings but it is pointless without a way to resolve them. So next up we want to...
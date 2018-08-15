## Service Location

### What is it?

**Service Location** *(SL)* stems from the same problem that **IoC** tries to solve, which is around resolving class dependencies without hard coding them.

> ## Always go for IoC wherever possible
Now before I go any further I just want to say that **SL** is inferior to **IoC**, if you have the choice ALWAYS opt for **IoC** as you will have less dependencies and simpler code, as to use **SL** you will require a class to do the location, and this will couple every class in your code to this object.

So as mentioned in the **IoC** section the problem is classes instantiating their dependencies internally like this:

```csharp
public class ManuallyInstantiatingDependencies
{
    private ISomeDependency _someDependency;
    
    public NotUsingIoC()
    { _someDependency = new SomeDependency(); }
    
    public void DoSomething() 
    { _someDependency.MakeTheMagic(); }
}
```

So in most cases you would *hopefully* solve this using **IoC**, however lets assume you don't have access to the constructor and the dependencies are private so you cannot use **Property Injection** with a **DI** framework. So this is where **SL** comes in.   

```csharp
using SomeDependencyContainer = SomeDependencyManager.Container;
 
public class UsingServiceLocation
{
    private ISomeDependency _someDependency;
    
    public UsingServiceLocation()
    { _someDependency = SomeDependencyContainer.Resolve<ISomeDependency>(); }
    
    public void DoSomething() 
    { _someDependency.MakeTheMagic(); }
}
``` 

So as you can see above we have the notion of some global cache of dependencies, this could be some sort of factory class or as quite often used a **DI** container which is made static *(This is a bad thing)*. Now on face value I am sure some of you will be thinking this makes your classes easier to use than **IoC** but trust me it's a false economy.

Let us quickly go over some of the problems with the above approach:

* All your classes with dependencies will be coupled with your `SomeDependencyManager` class
 * This means if you want to change **SL** class LOTS of code must change
 * Your classes are no longer as portable as they will require the consumer to have setup the locator ahead of time 
* You cannot test your classes without setting up your **Service Locator** class (i.e setting up the dependencies for `ISomeDependency`)
* You are unable to know what dependencies this class uses without looking at the source code

### So is Service Location worth it?

Nope.

Well that is the quick answer, if you can use **IoC** then you should be doing so, if not this is a better solution than manually instantiating your dependencies at it at least gives you the ability to change the implementation of your dependency without having to touch your classes logic.

RARELY is there a situation where it is a good thing to have a static object which all classes with dependencies are coupled to, but there are some NICHE scenarios where knowing this approach can come in useful, like writing your own class to proxy another class and add some notion of dependency management. This is something which is sometimes done in Unity when using **MonoBehavior** classes, as if you were to extend the base class and put your own proxy class in the way with some notion of resolving dependencies you can at least carve out a bastion of sanity within the chaos, something like this:

```csharp
using SomeDependencyContainer = SomeDependencyManager.Container;
public abstract class SensibleMonoBehavior : MonoBehavior
{
	protected T Resolve<T>()
	{ return SomeDependencyContainer.Resolve<T>(); }
}

public class PlayerBehaviour : SensibleMonoBehavior // Notice how we now use our proxy class
{
	private IPlayerSettings _playerSettings;

	public PlayerBehaviour()
	
	void Start()
	{
		_playerSettings = Resolve<IPlayerSettings>();
	}
}
```

The above example can simplify some MonoBehavior scenarios and you can even add the notion of `OnBinding` as an abstract method which auto triggers in start to make it a bit more explicit as to what should happen where.  

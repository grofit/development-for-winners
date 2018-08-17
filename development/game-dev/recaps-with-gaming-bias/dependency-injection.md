# Dependency injection in the gaming world

## This again?

As mentioned in the general section there are loads of different frameworks that allow you to achieve **DI** but in the game dev world things are often a bit trickier, this is down to how you need to often support multiple platforms (i.e windows, linux, ios, android, webgl etc) and also you are not always doing everything in code, you may be using the unity editor or some other scene first style approach, which means you have to take what is good about **DI** in general and apply it in a different way.

In the context of Unity there are not that many which are supported, so the main options are **Zenject** and **StrangeIoC**, although for this example we will use **Zenject** here purely because it is slightly more lightweight, as **Strange IoC** has a bit of conventions that come with it.

> You can find all the documentation and information on their websites for both:

- [**Zenject**](https://github.com/modesttree/Zenject)
- [**StrangeIoC**](http://strangeioc.github.io/strangeioc)

As mentioned almost all DI frameworks share the same sort of principals, just the syntax is different and maybe the features offered.

## Bindings

So to begin with **Zenject** has the notion of a **MonoInstaller** which is where it contains all binding setup, this is the same as a **Ninject** module.

```csharp
using Zenject;

public class GameInstaller : MonoInstaller
{
    public override void InstallBindings()
    {
        Container.Bind<ISomething>().ToTransient<SomeImplementation>();
        Container.Bind<ISomethingElse>().ToTransient<SomethingElse>();
    }
}
```

As you can see, nothing really shocking. It is basically the same as the **Ninject** version we showed before, however in this case installers need to be registered with a type of context:

- ProjectContext (Allows you to load a set of bindings via installers across all scenes)
- SceneContext (Allows you to load an installer for a given scene)
- GameObjectContext (Allows you to setup GO level stuff, but we wont be going into that too much)

Here are some other examples of simple bindings:

```csharp
Container.Bind<ISomething>().ToTransient<Something>(); // Bind it with transient scope
Container.Bind<ISomething>().ToSingle<Something>(); // Bind it with singleton scope
Container.Bind<ISomething>().ToInstance(new SomethingImplementation()); // Bind it to an instance
```

> There are far more binding scenarios specific to **Zenject** such as binding to **prefabs**, **methods**, **gameobjects**, **getters** and far more, all available within the documentation on the site.

## Resolving

Within **Zenject** you can do manual resolving quite easily given a container and just call `Resolve<T>()` on it, however given with a lot of Unity being scene driven it also allows you to inject into `GameObject` classes via attributes (as you cannot access the constructors of `MonoBehaviour` classes).

```csharp
using Zenject;

public class MyMonoBehaviour : MonoBehaviour
{
    [Inject]
    public ISomeDependency SomeDependency { get; set; }

    [Inject]
    public void OnDependenciesResolved()
    {
        // All dependencies have been resolved
    }
}
```

This is only needed within **MB** classes as you still get to have constructor injection for any and all classes you have constructor access to.
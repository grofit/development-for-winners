# Aspect Oriented Programming (AOP)

This is quite an advanced topic, so don't worry if you are a bit lost to begin with, ideally before looking at this you should have a good grasp on **Dependency Injection** and **Reflection**.

## What Is It?

Its basically a way of having code run within a method without altering the source code for that method. Which sounds pretty magical right, and if you look google it you will get all sorts of more in depths explanations but at the end of the day its a way to take certain cross cuttin concerns such as logging, transaction handling, performance metric gathering outside of your source code and just kinda hook them in when needed.

For example if we looked at a piece of code that did logging.

```csharp
public void DoSomething()
{
    _logger.Log("Starting DoSomething");
    // do actual logic
    _logger.Log("Finished DoSomething");
}
```

While this code is pretty trivial it shows common use cases where people want to add some simple logging to a method, but by doing this they make the containing class larger as it now needs to know about some `ILogger` or whatever, and also the method needs to manually have code stipulated to do the logging.

So wouldn't it be cool if we could just have some code trigger when starting/ending a method call which just did the logging for us without making the source code aware of this even happening.

## The Ways To Do This

Depending on your language there may be 1-N ways to do this, so I will only really touch on C# here, but just be aware different technologies/platforms have different ways of doing this approach.

### Interface Proxying / Interceptor Approach

If your **DI** framework supports it you can just tell that to proxy your interfaces when it is doing it's binding. While its different for each framework the gist of it is to stipulate when binding your interfaces what methods you would like proxied and how you would want them to be intercepted with.

> Quite a few of the open source DI frameworks in C# support interceptors such as Ninject, Autofac, DryIoC and many more, however the built in MS DI framework is VERY limited and can't really do any of this.

So for example in Ninject you could do something like this if you are using the `Ninject.Interception` plugin.

```csharp
kernel.InterceptAround<ISomeInterface>(                 // Interface to proxy around
    instance => instance.DoSomething(),                 // Method to proxy
    invocation => logger.Log("Starting DoSomething"),   // Before invocation
    invocation => logger.Log("Finished DoSomething"));  // After invocation
```

This is a bare bones version but this would allow you to trigger any logic you wanted before and after a method without the method even knowing that its happening. You can do even more fancy stuff if you define your own `IInterceptor` class and just intercept methods with that allowing you to re-use the interceptor wherever you want in a more structured way.

> The key thing to note about this proxying approach is that the **DI** container will dynamically proxy your interfaces at runtime which comes with a minor overhead and requires support from the framework, also it can be problematic in **AOT** compilation scenarios.

### IL Weaving Approach

This approach basically alters the **IL** (Intermediate Language) inside your dll file to add in the cross cutting concerns after compilation. From a performance perspective this is a bit faster as its baked into the dll after its been built, but requires a framework/tool to process your output and re-write chunks to hook into external functionality.

> A few of the main players in C# that do this are Fody and PostSharp, but I think there are some others out there and they use different approaches to applying the post processing of the DLL.

The difference here is that rather than your interception being done via **DI** configuration it is done via external configuration or through some assembly level attributes.

Rather than going into the details on it all you can look over the documentation for each framework.

## Why Is This Useful

Regardless of which approach you take it's super useful to know this pattern exists as quite often we end up littering logic with non functional concerns that just make it harder to maintain and test.

It can reduce a load of boilerplate code and make everything easier to read, maintain and test. The only downside with it all is that you can potentially have some overhead up front with interface proxying.

The use cases are also far more than just logging, you could have it so that all units of work are implicitly wrapped in a transaction at a higher level which auto rolls back or in, you can have authentication handlers on specific methods depdning on the user context as well as having it so certain event methods have a chance of raising an event which can trigger something else. 

The use cases for this pattern are huge, but it is quite tricky to set up and understand if you are new to it.


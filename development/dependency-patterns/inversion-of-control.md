## Inversion of Control
### What is it?

So I am sure you will have heard someone mention **Inversion of Control** *(IoC)* at some point, and I would say it is one of the most basic patterns for sensible software development. You may have heard people use the terms **IoC** and **DI** *(Dependency Injection)* interchangeably which is incorrect as **DI** is is really a way to achieve **IoC** but they often are used together.

Now rather than waffle on lets have a look at a class which is using **IoC**:

```csharp
public class UsingIoC
{
    private ISomeDependency _someDependency;
    
    public NotUsingIoC(ISomeDependency someDependency)
    { _someDependency = someDependency; }
    
    public void DoSomething() 
    { _someDependency.MakeTheMagic(); }
}
``` 

So this is possibly the simplest example of **IoC** and focuses really on what is most important about it, which is the way dependencies are passed into the class rather than being instantiated locally. You may currently the above approach without realising it, but lets just see another way of how this could be expressed and is quite often done.

```csharp
public class NotUsingIoC
{
    private ISomeDependency _someDependency;
    
    public NotUsingIoC()
    { _someDependency = new SomeDependency(); }
    
    public void DoSomething() 
    { _someDependency.MakeTheMagic(); }
}
```
> There is also another way to solve the above problem known as **Service Location**, which we will get to shortly.

So same class, doing the same thing but this time we are not passing in anything via he constructor, we are just newing up the instance of our dependency inside the constructor. I know lots of people do this as its a simple way to stop having to have complex constructors and just to be able to atomically use their class without worrying about passing lots of external stuff in. However this approach here is fraught with problems, so lets quickly go over some of the issues you will face in this second example:

- You cannot change the implementation of `ISomeDependency` being used
- You will struggle to test this class in isolation because you cannot mock the dependencies of it
- If you were to roll out a new default implementation of `ISomeDependency` you would have to change this source code

So taking a look again at the original example we can see that because we pass in our `ISomeDependency` we no longer have any of the above problems. Something so simple as just passing in a dependency rather than creating it yourself can be one of the largest steps taken towards a better design, and this approach is often referred to as the **Hollywood Principle** because of the *"Don't call us, we'll call you"* approach.

### Why is this important?

So as I have alluded to above, this is one of the most basic requirements for writing good unit tests. As without this ability to pass in dependencies you are unable to mock out bits of the test you do not care about, making it a lot easier to test only the important bits. It also allows you to make your code a lot simpler to change and/or maintain.

> Imagine you have an `IPathfinder` interface and you decide to make an `AStarPathfinder` implementation and its ok, your path finding calls are quick enough but take up a lot of memory for larger maps. You then make an `OptimizedAStarPathfinder`, you can now use this implementation throughout your entier codebase without changing a line of code within the classes which consume it when using **IoC**

This also leads onto the subject mentioned above which is **DI**, as most of the time you will use a **DI** framework to satisfy all the dependencies of your classes in a simple way.

> Just because this may seem too much of a hassle having to manually set each dependency in the constructor it is well worth it once we build upon this and get into **Composition** and **Unit Testing** topics, as this is the pre-requisite to being able to achieve them.
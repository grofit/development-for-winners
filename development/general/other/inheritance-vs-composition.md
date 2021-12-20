# Inheritance vs Composition

Now we have covered a lot of the important design pattern fundamentals lets have a little chat about why composition is almost always preferable to inheritance.

> While this is mainly going to focus on the topic from a coding perspective the points discussed can also be applied to a lot of scenarios in the development world, i.e `why having "Common" libraries may be bad`

## What Is Inheritance?

So im sure we all know the answer to this as you know how to inherit a class, if not then you have done very well to get this far without knowing this. Just incase you do not know (I know you do though, lets be honest) I will give a quick example of inheritance.

```csharp
public class Animal {}
public class Human : Animal {}
```

TADA! Not quite my finest work but we can see we have a base class `Animal` and `Human` is derived/inherited from that, so any behaviours an `Animal` has, a `Human` will also have, and that is the **KEY** thing to discuss here.

### Forcing Behaviour Upon Descendants

So one of the main benefits of inheritance is that you can put some logic within a base class, and know anything that inherits it will have access to that logic. This sounds good at face value, and you can even make your methods `virutal` which will let descendant implementations override or build on top of your existing logic.

In some cases this is ok, but in most cases you will hit a point where you want to completely ignore the inherited behaviour, and this is where inheritance falls down, as you will always end up in a position where you have to hack around/ignore base behaviours.

> Its also worth noting that inheritance in most languages does not allow multiple inheritance, meaning you can only really inherit from 1 class. In C++ you *can* do multiple inheritance but this often leads to the diamond problem where they both have a shared base class behaviour which causes issues when they combine.

## What Is Composition?

So composition requires an understanding of **IoC** as rather than getting our behaviour from a base class, we instead get our behaviour from dependencies.

Incase you are unsure here is an example:

```csharp
public interface IPrinter {}
public interface IComputer {}

public class Office
{
    public IPrinter Printer {get;}
    public IComputer Computer {get;}

    public Office(IPrinter printer, IComputer computer)
    {
        Printer = printer;
        Computer = computer;
    }

}
```

As you can see we can have the notion of an office which can use functionality that already exists in other interfaces, but we are not tied to a specific implementation and we can use as much or as little of the functionality of these objects as needed, no need to override anything via virtuals or fight any base behaviour.

> Just to mention im using interfaces here, but you could use classes too, but interfaces are far better as they are just a contract for behaviours not the actual implementation for behaviours.

This makes testing far easier as we can test the `IPrinter` and `IComputer` in isolation, as well as allow multiple instances of the same behaviours without issue, i.e having N printers in an office. You couldn't easily handle that if you were using inheritance, but having an office inherit from a printer sounds like a bad use case in hindsight now :D

> Here is a [gist](https://gist.github.com/grofit/a8c9c5aab72697b524d8) I did on the topic a while ago where it shows how we want to build off a `Vehicle` notion but it gets out of control quickly.

## Why Is Composition Better Then?

So composition is better because you have far more flexibility in many scenarios, as with inheritance you are always building off existing implementations of logic/data which can easily go against what you want downstream.

For example lets assume we have the notion of a **Vehicle** and we want to give it the ability to move and shoot. If we first made a **car** which can drive around and shoot, what if we then want a **tank** which has multiple guns, or a **jet** which can also fly with multiple weapons.

You can express the above scenarios with inheritance, but you end up duplicating your logic unless you want your **jet** to inherit from tank etc.

### Passing In Dependencies

Also another HUGE reason why composition is far better is because you are passing in **dependencies** which represent the composite behaviour you wish to consume, meaning you can more easily change what you pass in externally, via **DI** or other mechanisms assuming you adhere to IoC.

If you use inheritance you are unable to change the inheritance tree per instance as you are locked into a singular inheritance tree, but with composition you can pass in anything to satisfy the dependencies so you can not only have FAR MORE flexibility but also it's easier to express more specific variants as all you are doing is making a new object with composite dependencies vs adding another chain in an inheritance tree or branching it.

> Inheritance still has its uses, so it's not something you should 100% avoid using, its just you need to be sure if inheritance is actually a good idea for what you need as in a lot of cases rather than having logic shared via inheritance its often better to have it shared via its own object that can be passed in and consumed however the consumer wishes.
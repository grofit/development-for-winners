## Entity Component System Pattern (ECS)

### What is it?

At a high level it is a way to separate your data and logic concerns into components (where your data lives) and systems (where your logic happens), the way the data is split makes it very extensible and flexible.

As it is not object oriented but makes heavy use of composition it so **ECS** can often be a tricky one to wrap your head around. It is easier to think of the 4 main parts that make up the pattern:

- **Entity** (Contains many components)
- **Component** (Contains game data)
- **Group** (Entity selection mechanism)
- **System** (Runs logic on entities)

Lets take a deeper dive into each section and look at how typical **ECS** frameworks function and how we can think in terms of the components instead of typical models.

### Thinking in ECS

Lets start with how we change our thinking as this is really the most important bit to be discussed here.

As mentioned above entities contain many components, and components contain game data, but before we get more in depth on what these bits do it is worth thinking about how this pattern differs from others in the way you model your data.

#### Normal data separation approach

So lets assume you have a character who can move and jump, in other patterns you may have:

```csharp
public class Player
{
    public string Name {get;set;}
    public Vector3 Position {get;set;}
    public float MovementSpeed {get;set;}
    public bool isJumping {get;set;}
}
```

That looks good and represents what data that player needs to function, however this is grouping a lot of responsibility together, as you are saying the player can:

- Has a name
- Can move
- Can jump

From here you will probably have a class (or a few) which deal with the logic for these interactions, and they would take a `Player` instance and do the work.

So far sounds ok, but if I have a class which is a `MovementController` which takes a player and carries out movement, does it really care about the `Name` of the player?

This is where **ECS** kinda differs from other design patterns as it makes your data lots of little bits which do specific things, and the entity just cobbles it all together, so lets look at the same example in **ECS** world.

#### Same approach but managing data the ECS way

So lets assume same scenario, we want something that has a name, can move and jump and we need something to carry out logic on these variables.

So rather than having 1 single model which represents all the responsibilities we split it down into little bitesize chunks.

```csharp
public class MovementComponent
{
    public Vector3 Position {get;set;}
    public float MovementSpeed {get;set;}
}

public class JumpingComponent
{
    public bool isJumping {get;set;}
}

public class HasNameComponent
{
    public string Name {get;set;}
}
```

May seem a bit verbose upon first look, but here we are splitting our concerns into different chunks, so we have a component which wraps up the data needed for moving, jumping and displaying their name somewhere.

However its when we look at how systems process this stuff that we really see the benefit here, as our **systems** use **groups** as a contract to get applicable **entities** and run data on them.

Rather than continue on from here, lets pause at this point and look at the parts of **ECS** in more detail then revisit this scenario.

### Component

As seen above a component can be thought of as an isolated bit of data which is used for a specific purpose. So in the example of `MovementComponent` we can say that we need to know the current position and the movement speed so when input is triggered we can calculate a new position.

The main change in thought process is how we break up our data into such small pieces, as you could easily say that to jump you need to know about position and movement speed, and this is where groups and composition comes into play.

### Entity

Entities can be thought of as little databases, they just act as a container for the components it needs, so for example if your entity needs to move it would have a `MovementComponent`, but then if your entity needs to jump it would also have a `JumpingComponent`.

Most ECS frameworks implement things differently but almost all frameworks will make sure an entity has methods like:

```csharp
// Add a new component
entity.AddComponent(myComponent);

// Get an existing component
entity.GetComponent<MyComponent>();

// Check if a component exists
entity.HasComponent<MyComponent>();
```

With this you can easily add as many **components** as you want to an **entity** and retrieve them and check if they exist. This may seem really simple, and that's because it is, at its core an entity just acts like a big array of components.

### Groups

Before we get into **systems** it is pretty important to understand groups as they act as the contract between a **system** and **entity**.

Lets say you have a **system** (we will get into systems more next) which needs to carry out logic on all entities which can move and jump. You need some way to let the system only get entities it cares about, which in this case would be all entities which contain both `MovementComponent` and `JumpingComponent`.

Groups tend to be more of a notion than a specific implementation, although some ECS frameworks do implement specific group classes, where there is class representing groups I tend to name them like the high level notion they represent, like for example if we have a group for `MovementComponent` and `JumpingComponent` we could could think of this as a `PlatformingGroup`.

### Systems

Systems are another large part of the puzzle, as they are the things that make use of the data you expose from the **components**.

They often vary in implementation between ECS frameworks, but generally all **systems** have some form of method which acts as an executor per update for all entities matching a given **group**.

So lets just imagine a system implementation like below:

```csharp
public class PlatformingSystem : ISystem
{
    public IGroup Group 
    { 
        get 
        {
            return new Group(typeof(MovementComponent), typeof(JumpingComponent));
        }
    }

    public void Execute(IEntity[] entities)
    {
        // Do something with all entities which have
        // both MovementComponent and JumpingComponent
    }
}
```

As we can see here in this whimsical example that the system exposes what entities it wants to act on, and a method which will carry out logic on those entities.

In most cases there will be a layer above systems which orchestrates when they run, be it a main loop or some other executor of sorts, but ultimately in *most* implementations the systems `Execute` method is called every game update, but there are some implementations which allow you to specify when the execute should be run or what triggers should run the logic.

### Benefits of this approach

So now we have covered all the bits which make up the pattern and how they all fit together lets see what the benefits of this pattern are.

#### Fully extensible

The extensibility of the pattern is probably its biggest benefit, because of the way **systems** subscribe to certain entities it makes it very easy to add new systems. 

Lets pretend we already have a working prototype for a game, and in it our character runs around on various environments. We could easily add a new `FootstepsSystem` which listens for all entities which can move and when their position changes it plays a sound clip, or even emits a couple of dust/water particle systems.

As you can see you can quickly isolate your logic and data and keep using this additive logic approach where you keep adding more systems without worrying too much about breaking changes.

#### Separation of data and logic

This is another big thing, and most patterns will opt for data separation, but this separation makes it easy to only deal with the bits of data you care about and re-use it wherever it is needed.

### Being pragmatic and modernising

There is also something to be said for being pragmatic when it comes to using patterns like this, as historically things like events have been expressed as components which are removed after being processed, but this doesn't work well with pub/sub style models, so I found it better to add the notion of an event bus into the ECS paradigm. 

Also given that ECS systems typically run every update, what if you don't want your system to run every update, you want them to run every 5 seconds, or when an event is raised or when an entities health is <= 0. Typically you would just have some delay or predicate on your system start, but if you know about Rx (covered in previous chapters) you could easily make use of this with ECS to create reactive systems, where instead of executing every update and not doing anything, they only execute when an `IObservable` is triggered.

### Wrapping it up

Hopefully this gives a quick overview to the pattern and the benefits of changing your mindset slightly.

This being said it can easily become difficult to manage larger ECS projects as once you start getting a large amount of components and systems it can be tricky to remember what does what, so it is important you name your components and systems well.

We also have not really covered how to implement your own ECS system as this can be quite complex, but there are a myriad of existing ECS systems which are all geared for different scenarios, be it performance, functionality, reactivity etc. So it is advised to use an existing one rather than re-inventing the wheel.
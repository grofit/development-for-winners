# Strategy-like Patterns

There are a myriad of patterns out there which all basically remove the need for having complex hard coded `if` or `switch` statements, they all basically do the same thing, which is remove the need for the hardcoded control flow and replace it with something more dynamic, be it a single class that you call which does it all for you, or a set of objects you loop through. Rather than detail each one individually, I will just group them all together under the same banner and give you a bit of info on what the differences are between some of the main patterns.

> Patterns are just guides as to how to solve a problem, you can stick to a pattern exactly how the text books or just see the high level benefit and see how it effects your scenario, in this example I am going to make use of a bit of each pattern while not really sticking to any individual one just to show that it can make sense to be a bit pragmatic with patterns at times.

## An overview on the specifics of each

Here is a little blurb on how some of the main varients of the strategy pattern differ, I will use the term **handler** a lot to indicate something that contains isolated logic.

### Strategy Pattern

The strategy pattern is basically a way to dynamically pick a handler (some logic) based upon some predicate, the difference between this and other similar patterns is that in this case you would only apply one of the strategies that you have. So if you were to have 5 available `IStateHandler` implementations, only one should be applied.

i.e If you were a character who could only be in one state at a time, you could use this pattern to decide which state is applicable and run it

### Chain of Responsibility

This pattern differs on a couple of fronts, generally you will be able to apply multiple handlers within a given scenario and there will be a single entry point which will delegate down to other handlers, this can be achieved by inheritance with recursive calls to same base methods to apply logic, or via a relay sort of system where you pass on the data to the next in the chain.

i.e If you were in a guild/clan and you had different roles which allowed access to different things, if you wanted to see if you had access to the guild bank you would need to go through each role handler to see if any of the ones applied to you allow you access to the bank.

### State Pattern

This pattern is again very similar to the **Strategy** pattern in that only 1 handler will be active at once, but it generally operates back to front, so your handler (state) object takes the context (whatever your handling) and alters that depending on the current state then will potentially change the state on the context.

I would give you an example here but really I cannot think of a real use case for this, given there are so many similar patterns which are less invasive/coupled.

## Scenario showing the control flow problem

So lets show a quick example of a problem where we could use one of these sort of patterns to improve the code.

```csharp
public class Character
{
    public EntityStates CurrentState { get; }
    
    public void Update() 
    {
        switch(CurrentState)
        {
            case EntityStates.Idle 
            {
                // play an idle animation
                // check for a timer to see if you need to do a random idle pose
                // regen health
            }
            break;
            
            case EntityStates.Patrolling
            {
                // move character towards waypoint
                // play walking animation
                // check for collisions
            }
            break;
            
            case EntityStates.Fleeing
            {
                // move character in direction but faster
                // play running animation
                // check for collisions
                // very similar to walking
            }
            break;
            
            // More cases with more logic
        }
    }
}
```

As you can see here we basically have a lot of states and the character can only be in one state at a given time. This works fine for the moment but as the game goes on we will keep adding more and more states, making it harder to maintain and potentially test as this `Update` method will end up just becoming a dumping ground for logic.

> You may have thought "we could use composition to reduce the noise in the cases" which you could do and that would be a step in the right direction, but you would still need to explicitly have an instance of each state in the character, which would in turn grow as the concerns the character can handle increases.

## So lets make it better

So we want to make the switch statement more dynamic as well as not creating too much noise so the update is succinct and easy to test if needed, so lets look at a better approach.

```csharp
public interface IStateHandler<T>
{
    bool CanHandle(T data);
    void Handle(T data);
}
```
We can add an `IStateHandler<T>` which lets us define an object which will check if it can be run, and encapsulate the logic needed to run if it is a match.

With this in place lets create some handlers to deal with the existing logic we would need in this scenario.

```csharp
public class CharacterIdleStateHandler : IStateHandler<Character>
{
    public bool CanHandle(Character character)
    { return character.CurrentState == EntityStates.Idle; }
    
    public void Handle(Character character)
    {
        // play an idle animation
        // check for a timer to see if you need to do a random idle pose
        // regen health
    }
}

public class CharacterPatrollingStateHandler : IStateHandler<Character>
{
    public bool CanHandle(Character character)
    { return character.CurrentState == EntityStates.Patrolling; }
    
    public void Handle(Character character)
    {
        // move character towards waypoint
        // play walking animation
        // check for collisions
    }
}

public class CharacterFleeStateHandler : IStateHandler<Character>
{
    public bool CanHandle(Character character)
    { return character.CurrentState == EntityStates.Fleeing; }
    
    public void Handle(Character character)
    {
        // move character in a direction but faster
        // play running animation
        // check for collisions
        // very similar to walking
    }
}
```

It is a bit more code, but we can now re-use our state handlers if needed and test them in isolation, we can also improve the character and make it more dynamic by just letting it take in all applicable state handlers via **IoC** then just loop through until we find the applicable one.

```csharp
public class Character
{
    public EntityStates CurrentState { get; }
    
    // We can inject in any and all state handlers for the character
    private IEnumerable<IStateHandler<Character>> characterStates;

    public Character(IEnumerable<IStateHandler<Character>> states)
    { this.characterStates = states; }

    public void Update() 
    {
        // Loop through all our states finding the right one
        foreach(var state in characterStates)
        {
            // Check if this state should handle it
            if(state.CanHandle(this))
            { 
                // Handle it and stop looping
                state.Handle(this);
                break;
            }
        }
    }
}
```

This whole bit is a lot to take in, but as you can see we have made it so you can easily add new handlers and the code for `Character` does not need to change, this is a huge win as we can now test our handlers in isolation, and if they need dependencies to operate we can inject them in without polluting the `Character` class. This is adhering primarily to the **Strategy** pattern but we could easily alter the scenario and it starts blurring into some amalgamation of the other patterns.

## In the real world

In the real world we may want to stray slightly from the focus on 1 state handler and allow additive state handling, so you could potentially change the update method to:

```csharp
public void Update() 
{
    // Loop through all our states finding the right one
    foreach(var state in characterStates)
    {
        // Check if this state should handle it
        if(state.CanHandle(this))
        { state.Handle(this); }
    }
}
```

This would allow you to have multiple handlers running per update, so for example if you wanted to have a character fleeing while attacking, or idling while healing, you could have multiple handlers that are simultaniously applicable, this opens up more doors to you while still keeping the benefits of the previous implementation. This is a half way house between **Strategy** and **Chain of responsibility** but as we are not internally deferring the logic we are not quite adhering to text book implementations of either pattern.

> If we were going to stick on this approach where there was only a single handler you could optimize further by having each state handler bound to a specific state in a dictionary so you dont need to keep looping every update, this would probably be better implied by making each `IStateHandler` expose a state which it is tied to.

## Final blurb

Like with any pattern, be pragmatic. They are generally guides on how to solve something, not rules. In this case there are soooooo many patterns which have a huge overlap with each other in this domain, ultimately the goal is to simplify code, increase flexibility/testability and isolate concerns, we can do that with any of these patterns but each of them offers a slightly different flavour, and while in some cases you may want to stick to one of them as a text book implementation, you don't want to limit yourself to just driving within the lines, go off road, see what wonders await you.
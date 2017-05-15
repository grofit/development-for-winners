### Factory

The factory pattern is one of the simpler patterns you will come across. Its purpose is to hide the underlying creation mechanism for a class so you can create something in an isolated way and get an implementation you need without having to worry about actually initializing it yourself.

#### Getting started

For example if we had some classes like:

```csharp
public enum AnimalType
{
    Unknown = 0,
    Dog = 1,
    Cat = 2
}

public interface Animal
{
    string MakeSound();
}

public class Dog : Animal
{
    public string MakeSound()
    { return "woof woof"; }
}

public class Cat : Animal
{
    public string MakeSound()
    { return "meow meow"; }
}
```

Then if we wanted to get an animal based upon the type we could make a factory like so:

```csharp
public class AnimalFactory()
{
    public IAnimal CreateAnimal(AnimalType type)
    {
        switch(type)
        {
            case AnimalType.Dog: { return new Dog(); }
            default: { return new Cat(); }
        }
    }
}
```

That is a basic factory which would generate the implementations you require without you having to worry about how it does it.

#### Example usage

```
var animalFactory = new AnimalFactory();
var myAnimal = animalFactory.CreateAnimal(AnimalType.Dog);

Console.WriteLine(myAnimal.MakeSound());
```

> Historically factory patterns used to be used a lot to abstract away hardware resources, like textures, vertex buffers etc. As when you were making a cross platform engine and needed to work with OpenGL, DirectX (or other rendering technologies) the developer rarely cared about the underlying hardware, they just wanted a texture of a certain size to use, or a vertex buffer of a certain size to provide to the renderer, so this pattern helped abstract away these concerns and exposing high level objects to represend the underlying resources.

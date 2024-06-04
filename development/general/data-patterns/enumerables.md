## Enumerables

This is more specific to C# for the most part but lots of languages have the notion of `Enumerables` but in C# specifically they are a bit more interesting.

So `Enumerables` are basically just data structure agnostic collections of data which can be **enumerated**. So rather than having to know we have an `Array`, `List`, `Queue`, `Dictionary` etc we can just say we have an `IEnumerable` and we can support almost any collection and **enumerate** over the elements contained within.

> As mentioned the `Enumerables` are agnostic of how the data is stored/made available, so you could make almost any data structure `Enumerable` as long as you can keep providing the next element available.

### Enumerables and Lazy Loading

This again is more specific to C#, as in C# an `IEnumerable` also doest guarantee the data already exists, it could be lazy loaded or could be in memory already. All it cares about is that you provide the consumer a way to access the next element.

With this in mind you never know how many elements are in the `Enumerable` until you **enumerate** over them. Which may seem rubbish at face value, but in the long run its actually pretty powerful and can actually be used to do some interesting things.

> In other languages/platforms `Enumerables` may not be lazy loaded by default and may only exist on certain types, its only recently JS supported the notion of `Enumerables`, and Ruby has an additional Lazy module needed for lazily loaded `Enumerables` etc.

To show you what I mean here about the data being lazy loaded, imagine the below code:

```csharp
private static Random _random = new Random();

public static IEnumerable<int> GenerateRandomNumbers(int amount = 5)
{
    for(var i=0;i<amount;i++)
    {
        yield return _random.Next(1,100);
    }
}

public static void Main()
{
    var someNumbers = GenerateRandomNumbers();
    
    Console.WriteLine(string.Join(", ", someNumbers));	
    Console.WriteLine(string.Join(", ", someNumbers));
    // These 2 lines will show different numbers
}
```

This may be mind blowing to some people who have never really thought about `Enumerables` before, as the `someNumbers` variable doesnt actually contain any data, its just a reference to an `Enumerator` that will provide us the data we need.

You may also be thinking *"oh wow, this is really bad"* as you cant guarantee the contents will always be the same on multiple enumerations.

> The compiler/IDE will often tell you that you are doing multiple enumerations and its dangerous, you may be fine if the underlying instance is an `Array` or something, but it's worth knowing about.

You can also do fun things with this, like if I was to take the above example and make a method like so:

```csharp
public static IEnumerable<int> GenerateInfiniteRandomNumbers()
{
    while(true)
    {
        yield return _random.Next(1,100);
    }
}
```

This may look crazy, and you are thinking *"This will just lock the CPU forever"* and it may do if you use it incorrectly, but what if you did this.

```csharp
var infiniteNumbers = GenerateInfiniteRandomNumbers();
Console.WriteLine(string.Join(", ", infiniteNumbers.Take(3)));
Console.WriteLine(string.Join(", ", infiniteNumbers.Take(10)));
// You would get a line of 3 numbers, then a line of 10 numbers
```

This would work perfectly fine, and you can quite happily have infinite `Enumerables` because its lazy loading the data and using the `yield` keyword to return execution to the caller once its got the next element.

### `Enumerables` vs `Collections`

So now we know about this super cool stuff, why would we use this over `Collections`?

It often comes down to the use case for the data, if you need to get a specific element, or need to know the `Count` or `Length` of the data then you cannot do that without **enumerating** the entire set for an `IEnumerable` but for a `Collection` its already in memory so it knows exactly how many are there and how its laid out.

> Its worth noting there are many levels of `Collection` depending on what you need, i.e an `IReadOnlyCollection` only guarentees a count, not a way to get an element via its index without enumeration, for that you would need to use an `IReadOnlyList`.

Knowing about all the various data structures and how they can be accessed can be really useful for when you want to pick the best tool for the job.

### This is why Linq is great

I wish every language/platform had **Linq**, it is one of the best things about C# and it is only possible because of `IEnumerable`.

We touched on it briefly above when we used infinite numbers but you can use **Linq** to constrain your data on the fly, for example:

```csharp
var infiniteNumbers = GenerateInfiniteRandomNumbers();

var constrainedNumbers = infiniteNumbers
                            .Where(x => x < 10)
                            .Take(20)
                            .OrderByDescending(x => x)
                            .Distinct()
                            .Take(5)
                            .Select(x => x.ToString());


Console.WriteLine(string.Join(", ", constrainedNumbers);
// Outputs 1-5 numbers in descending order under 10 i.e 9, 8, 5, 3, 1
```

This is a silly example but if we run through whats actually happening, we are first getting infinite numbers, then we are taking 20 of them, ordering them from highest to lowest (descending), then we are removing any duplicate numbers, taking the top 5 and converting them all to strings.

We could do this multiple times too like this:

```csharp
Console.WriteLine(string.Join(", ", constrainedNumbers);
Console.WriteLine(string.Join(", ", constrainedNumbers);
// Both lines would likely be different
```

With this you can make entire lazy loaded data filter chains with such ease, when you combine this notion with more complex data structures like `Observable` objects for data streams you can do some amazing stuff.

> As `constrainedNumbers` is also an `IEnumerable` you could daisy chain more **linq** onto it, to create another pre-filtered object for use elsewhere.

### Forcing Enumeration

You can have your cake and eat it to some degree with `IEnumerables`if you call `.ToArray()` or `.ToList()` you can force **enumeration** of all elements up front into an `Array` or `List` structure which will allow you to loop over as much as you want and not get any different results, lets show this using the other example.

```csharp
public static void Main()
{
    var someNumbers = GenerateRandomNumbers().ToArray();  // We add ToArray()
    
    Console.WriteLine(string.Join(", ", someNumbers));	
    Console.WriteLine(string.Join(", ", someNumbers));
    // These 2 lines will show the same numbers
}
```

So if you have an `Enumerable` object but need to iterate multiple times etc you can do so with the above approach. You can also access specific elements via indexes i.e `someNumbers[2]` and get the Count/Length of the objects.

### Finishing Words

As you can see the humble `IEnumerable` is a really powerful concept, and as virtually all collections of data implement it you can make use of Linq and related `IEnumerable` functionality anywhere.

It also shows the key difference (in C# at least) between regular `Collections` and `IEnumerable` as they both serve a different use case, and while its nice passing around `IEnumerables` everywhere, you may find you need `Collections` in some cases for performance reasons or to do more complex looping via `for` loops etc.
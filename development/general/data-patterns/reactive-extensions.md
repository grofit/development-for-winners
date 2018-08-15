## Reactive Extensions aka RX

There is a plethora of information online on this subject so I will not be delving too deep into the world of **rx** but it is worth knowing the basics as this can really help you in the realm of game development and/or real time applications.

- [ReactiveX Intro Docs](http://reactivex.io/intro.html)
- [IntroToRx Website](http://www.introtorx.com)

There is an rx implementation for most platforms these days, from PHP to C# but we will focus on the C# implementations, of which there is a more general **System.Reactive** (rx.net) framework and a more unity specific one **UniRx** (rx for unity) version, there is a huge amount of overlap so we will just cover it under the unified **rx** banner.

### A quick warning

RX can be a lot of things to a lot of people, once you know how it works it can quite often it can be seen as a panacea for any problem. This often leads down a rabbit hole where you are expressing complex concerns in a large observable stream, however it can easily get out of hand and become difficult to debug, test and maintain.

It is one of those things that requires a lot of effort to fully wrap your head around and just because you *CAN* do something, doesn't always mean you should.

So with that warning out of the way...

### What is it?

At its heart it is the notion of reactive streams, which are represented as `IObservable<T>`, which exposes a `Subscribe` method which will notify you when the value changes.

It is a lot like how the notion of `async` works where you are waiting on something to happen then you handle the result and do some logic with it, but this can be represented as a data push opposed to a data pull.

This can be super useful when you want to do something when a value changes, like checking if a character has died when their health changes:

```csharp
healthObservable.Subscribe(HealthHasChanged);

public void HealthHasChanged(int newHealthValue)
{
    if(newHealthValue <= 0) { /* They died */ }
    // do anything else here
}
```

#### Important objects

- `Observable<T>` (readable stream)
- `Subject<T>` (readable/writable stream)
- `Observer<T>` (does something on a stream)

There are also a few other related interfaces but in most common use cases you wont really care much about them unless you want to write your own custom extensions or observable creation objects.

### Creating Observable Streams

Almost 99% of your rx usage will be interacting with `IObservable<T>` implementations, but how on earth do we create an observable implementation to use?

If you are wanting to react to your data changing you would generally want to create them as `Subject<T>` or if using `UniRx` create them as `ReactiveProperty<T>` (which is a handy wrapper around `Subject<T>` and some other interfaces).

Which using UniRx would look like:

```csharp
public class Player : IDisposable
{
    public ReactiveProperty<int> Health {get; private set;}

    public Player()
    {
        Health = new ReactiveProperty(100); // Start with 100 hp
    }

    public void Dispose()
    { Health.Dispose(); }
}

myPlayer.Health.Subscribe(x => Console.WriteLine(x.ToString));

// would cause the above subscription to trigger with 20
myPlayer.Health.Value = 20; 
```

You can also create observables from the helpers provided in the `Observable` class, like so:

```csharp
Observable.Interval(TimeSpan.FromSeconds(1)).Subscribe(...)
```

There are a myriad of helpers available to create observables based on timings, web requests, file operations, key inputs etc.

### Filtering streams

Now you know how to make observable streams you should also know there are lots of extensions methods that let you filter streams. Most of this is already done for you, but you can easily make your own extensions with `IObserver<T>` implementations.

So lets focus on our existing scenario, we want to know when our characters HP has hit 0 so we can do some death trigger. Currently we know how to subscribe to every health change, but what if we only want to subscribe to the moment the health hits 0.

```csharp
myPlayer.Health
    .Where(x => x <= 0)
    .Subscribe(HandleDeath);
```

As you can see that is super simple, we just use a `Where` extension which will filter the incoming values and only progress the stream if the predicate matches.

Also lets think about the health display, what if we want to display the current health as text, but we also want to cap the health display at 100 (so if they got a buff or something to put them over 100 temporarily we just dont display more than 100).

```csharp
myPlayer.Health
    .Where(x => x <= 100)
    .Select(x => x.ToString())
    .Subscribe(Console.WriteLine);
```

Problem solved, as you can see its easy to quickly create a stream that represents what you care about, you can even wrap it up and re-expose it as a new stream if you wanted.

```csharp
public class Player : IDisposable
{
    public ReactiveProperty<int> Health {get; private set;}
    public IObservable<Unit> HasDied {get;} // Unit is like a void in rx

    public Player()
    {
        Health = new ReactiveProperty(100); // Start with 100 hp
        HasDied = Health.Where(x => x <= 0).AsObservable();
    }

    public void Dispose()
    { 
        Health.Dispose();
        HasDied.Dispose(); 
    }
}
```

### Clearing up

Currently we have shown that you can create streams and easily filter on them, but we have glossed over the point that they need to be disposed of.

So if we look at this example listed above:

```csharp
myPlayer.Health
    .Where(x => x <= 0)
    .Subscribe(HandleDeath);
// Uh oh, this is going to leak memory if not disposed
```

We are not clearing up the subscription, so really this should look like:

```csharp
var subscription = myPlayer.Health
    .Where(x => x <= 0)
    .Subscribe(HandleDeath);

// then when you no longer need this subscription call
subscription.Dispose();
```
Now as the stream should be seen as a constant torrent of changes, you expect your subscription callbacks to be invoked many times, which is why they are long living subscriptions. However in some cases you only care about that subscription firing once, in which case it would then need to be disposed of.

Luckily most implementations are smart enough to self dispose in these scenarios (always check the implementation supports this), so if I only wanted to know about the player dying once I could change this logic to:

```csharp
myPlayer.Health
    .First(x => x <= 0)
    .Subscribe(HandleDeath);
```

This now is often clever enough to know that the current stream is only going to yield a single value and tidy up after itself so you dont need an explicit dispose in these scenarios.
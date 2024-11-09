# Service Composition

There may very well be a better title for this section and maybe there is a specific term/pattern that encapsulates this notion, but as I do not know it I will just continue to waffle on and update it in the future if someone points it out.

> It is also recommended you check out the `repository` section as this is basically the same sort of thing but at the service layer.

## Quick rundown on n/3 tier architectures

I was going to put `n-tier`/`3-tier` architectures in its own bit, but its all quite simple so I will whack it in here. So most applications can be broken down into 3 layers at the heart of it:

- Consumer Layer (UI/API/Console/Sockets etc)
- Logic/Service Layer (Services)
- Data Layer (Repositories, DataSources etc)

When you think about things this way almost every app, api, worker you have worked on has got these 3 layers. You generally have your `data` layer at the bottom which will be where you have dependencies on your data sources and maybe `repositories` abstracting them away. Then you have the `Service` layer which is generally where you have your core logic which depends on the `data` layer and does stuff with the data in some way. Then finally we have the `Consumer` layer which is where you expose the logic for the consumer to use, be it a web api where they can do a HTTP request for a product, or a UI of some sort where they press a button and it delegates to logic.

> The Logic layer may actually delegate to 3rd party Services, such as other APIs, so it doesnt always mean that the logic lives directly inside your library/project, but more there is something that happens that is agnostic of the consuming layer, and depends upon the data layer.

## What is a `Service` then?

So if we focus right in on the Logic layer, thats where your services live, and they will often look something like this:

```csharp
public class UserService : IUserService
{
     // Assume under the hood its a UserData generic
    public IUserRepository UserRepository {get;}

    // Lets assume this has some helper methods for sending an email
    public IEmailProvider EmailProvider {get;}

    // ASSUME CONSTRUCTOR INJECTING ABOVE IN
    
    public async Task<User> GetUser(Guid id) 
    {
        var user = await UserRepository.Get(id);
        if(user == null) { throw new UserNotFoundException(id); }

        // Lets pretend we have some auto mapper style extensions which convert between data/service layer models
        return user.TransformTo<User>();
    }

    public async Task<User> RegisterUser(UserDetails userDetails)
    {
        // Eject if the email address exists
        var isEmailAddressTaken = await UserRepository.Query<HasUserWithEmail>(userDetails.Email);
        if(isEmailAddressTaken) { throw new EmailTakenException(userDetails.Email); } // Notice its not a HTTP specific exception or anything

        // Create the user in the DB getting us our unique ID
        var userData = await UserRepository.Create(UserDetails.Email, UserDetails.Name);

        // Send the user an email containing their name, and the id for a clickable link to their user page in the content
        EmailProvider.SendNewUserEmail(userData.Email, userData.Name, userData.Id);

        // Transform and return the user data into a user service layer model
        return userData.TransformTo<User>();
    }

    // Other methods for deleting users, updating them etc
}
```

This looks reasonable right? I am sure we have all seen stuff like this before, and its fine. Although like a train derailing I want to quickly go off on a few tangents before I get back to the core issue with the above approach.

### Why not share models between layers?

As you can see we have a `UserData`, `UserDetails` and `User` model, all of which seem to store similar data but in reality they will probably be quite different in terms of what they contain/expose. In some cases it may be perfectly fine to share a `User` model between all layers and call it a day, but it often isn't as simple as that due to the security/contract related issues.

For example your `data` layer models may contain information such as `CreatedOn, CreatedBy` audit style information that should really be bubbled up to other layers, partly for security, partly because no one cares about it. You also may want to combine normalized anemic data models somewhat, for example you may have multiple tables that contain complex grouping/permission information, but in the logic layer you just want pre-joined bits of information as their own models etc.

Same sort of thing goes for the `Logic` to `Consumer` layers, the logic may care about certain bits of information that the end user/consumer may not need to see for various reasons, so having different `Aspects` of the same model in your different layers/domains may make more sense.

Finally it gives you more flexibility to change lower layer data structures without impacting higher levels, for example if you end up moving a field from one model to another in the `data` layer then the logic layer can just absorb this change into its transform logic and everything above it will keep working as intended. It also lets you maintain the `consumer` facing contracts irrespective of the lower layer refactors.

> Its not like this is 0 effort to do though, maintaining models per layer (and sometimes multiple models per layer) can be hard word and in some cases there may genuinely be no reason to do this, or at least do this up front, but keep in mind that if you are exposing data that is not needed on layers above then maybe you have a design issue.

### Why not just return HTTP/Consumer specific exceptions from service layer?

This is a good question, and quite a few times I have had people put HTTP exceptions (i.e 404, 403, 401 etc) in service layers and tell me "its fine", but in the above example we have not done so. This is mainly because:

- We dont want to have to depend on HTTP/Consumer/Transport related libraries in our logic layer (this would taint all calling code too)
- We may not always be calling this service logic from a consumer (more on that later)
- We are making it more complex to understand what went wrong, a custom exception allows you to provide useful context

> HTTP has mainly been used as the example here but this could relate to other consumer layer libraries, its just more people know the HTTP side of things and know what a 404 is.

One final point here is that using custom exceptions you free up the consumer to make their own decision on how to handle this, for example in some code you may want to treat a missing user as a HTTP 404 result, but in others it may be ignored or be a different error code/process. You ultimately give the caller the ability to decide for themselves how that layer should treat this exception.

## So why was that `Service` example bad?

The example we showed before is pretty common and will work fine, people will write tests for it, and rejoice at the quality of their outputs, and its not bad. However as with everything in this book, there are some other ways that this can be done which will potentially make your life easier in the long run.

Lets start with the main problem, the service is doing too much... WHAT??! I hear you cry from the nearest rooftop, its a `User` service, its job is to provide functionality pertaining to users, and you are right it does that, but lets imagine a scenario which is quite common where you have one service depend upon another.

```csharp
public class ProfileService : IProfileService
{
    public IUserService UserService {get;}
    public IProfileRepository ProfileRepository {get;}

    public async Task<Profile> GetProfile(Guid userId)
    {
        var user = await UserService.GetUser(userId);
        var profileData = await ProfileRepository.Get(userId);
        return profileData.TransformTo<Profile>(user);
    }

    // Some other methods
}
```

This is a bit whimsical but ive seen plenty of examples where one service depends upon another, much like in the previous `Repository` chapter where people often have one repository depending on another one to call into some existing logic. This makes sense though, there is literally a whole principle called DRY which means "Dont Repeat Yourself", so doing this is good right?

At face value it is, but the problem here is that we only want to use the `GetUser` method from the service, yet we have to take all the other methods the interface exposes. You could argue at this point "If we use the interface segregation principle we could cut down on that" and yeah you could, but then you end up with an interface per method you want to call... that is stupid right....?

Lets put a pin in that and come back to it when we rewind a minute and look at how we would test our `UserService` implementation.

### Pretending to test the service

Lets write some whimsical unit tests.

```csharp
public class UserServiceTests
{
    [Fact]
    public async Task should_return_exception_if_user_doesnt_exist()
    {
        // some made up guid
        var dummyGuid = Guid.NewGuid();

        // mock our repo to return null
        var mockRepo = Substitute.For<IUserRepository>();
        mockRepo.Get(dummyGuid).Returns(null);

        // Have to make this because service needs it
        var mockEmailProvider = Substitute.For<IEmailProvider>();
        
        var userService = new UserService(mockRepo, mockEmailProvider);

        // Assert exception is thrown and contains id
        var exception = await Assert.ThrowsAsync<UserNotFoundException>(() => userService.GetUser(dummyGuid));
        Assert.Equals(dummyGuid, exception.UserId);
    }

    // More tests
}
```

Looks great right, but if we look at line 136 I cant help but think that whole `IEmailProvider` is pointless, as our call site doesn't need that in any way, I could pass in a `null` value, but either way we are saddled with needing this email provider because the containing class requires it.

## WILL YOU GET TO THE POINT AND TELL US WHAT YOU WANT US TO KNOW?!?

Yeah sorry, that train derailed pretty far there...

Lets be honest, those problems listed above are not show stoppers right, they are just minor annoyances. However if we continue this way as the services grow, so too will their dependencies, and then their unit tests, and the raft of dependencies downstream consumers will end up requiring.

So yeah lets put the train back on the tracks here and get to the point.

> I am doing this purely out of spite to you the reader, and I am going to regale you for a short period about how bad I am at Open Transport Tycoon when it comes to trains. I can never quite get the train signals working how I want, it causes me no end of pain. To be honest I never even wanted to play OTTD but a friend of mind REALLY wanted to play it, and one time came to our fancy dress LAN night as a train conductor hoping we would play it, but alas we didnt. Fast forward 10-20 years and we all have played it together now and very much enjoyed it, although as mentioned above I am still really bad at managing train systems.

Remember back in the `Repository` section we had these same problems, and to solve it we just took each individual method and put it in its own `Query` object, and believe it or not, we will just do the same thing here. So the previous example would just become something like:

```csharp
public class GetUserAction : IGetUserAction // Call it action/service/unit/operation... whatever you fancy
{
     // Assume under the hood its a UserData generic
    public IUserRepository UserRepository {get;}

    // ASSUME CONSTRUCTOR INJECTING ABOVE IN
    
    public async Task<User> GetUser(Guid id) 
    {
        var user = await UserRepository.Get(id);
        if(user == null) { throw new UserNotFoundException(id); }

        // Lets pretend we have some auto mapper style extensions which convert between data/service layer models
        return user.TransformTo<User>();
    }
}
```

Seems fine, not much has really changed, we just named it something else and took away the other responsibilities and the dependencies.

```csharp
public class RegisterUserAction : IRegisterUserAction
{
     // Assume under the hood its a UserData generic
    public IUserRepository UserRepository {get;}

    // Lets assume this has some helper methods for sending an email
    public IEmailProvider EmailProvider {get;}

    // ASSUME CONSTRUCTOR INJECTING ABOVE IN
    
    public async Task<User> RegisterUser(UserDetails userDetails)
    {
        // Eject if the email address exists
        var isEmailAddressTaken = await UserRepository.Query<HasUserWithEmail>(userDetails.Email);
        if(isEmailAddressTaken) { throw new EmailTakenException(userDetails.Email); } // Notice its not a HTTP specific exception or anything

        // Create the user in the DB getting us our unique ID
        var userData = await UserRepository.Create(UserDetails.Email, UserDetails.Name);

        // Send the user an email containing their name, and the id for a clickable link to their user page in the content
        EmailProvider.SendNewUserEmail(userData.Email, userData.Name, userData.Id);

        // Transform and return the user data into a user service layer model
        return userData.TransformTo<User>();
    }
}
```

Now wherever we want to use these action thingys we can just inject in `IRegisterUserAction` or `IGetUserAction` and they can be tested in isolation without any dependency bleeding etc.

> Pretty anti-climactic right, thank goodness I put in a mini story about OTTD to make this chapter more interesting.

In all seriousness though this may seem super trivial but its actually pretty important, as now we can express our logic in bitesize pieces which do a specific thing which makes it easier to re-use, compose and test however you see fit.

### Some other info worth thinking about

This is a bit more whimsical and unrelated but I know a lot of you out there are getting the cloud rammed down your throats, and are being told "WE NEED TO DEPLOY OUR APIS AS LAMBDAS/AZURE FUNCTIONS" etc. Due to this a lot of people teams just take their big APIs and lob them on Lambdas with a HTTP trigger to direct to it and call it a day, maybe even call it a micro service if they feel adventurous.

When you deploy code in a serverless way you end up paying for execution time, and what do you think happens when your execution needs to spin up a whole API? it takes a while right?. To make things leaner in these situations and reduce the cold boot time and total execution time you only want to inject in the specific things you care about, and the above approach allows you to sidestep large dependency resolution chains in some cases as you only call the service logic you need, not large logic containers that attempt to do everything for a given domain entity (i.e User, Product, Group etc).

If you dont care about all this and still want a sort of `Fascade` accessor for logic you can just make your `UserService` and under the hood just inject all the actions you want and just pass though calls, but I feel it somewhat avoids the benefit of it.
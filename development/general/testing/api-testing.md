# API Testing

This is me asking you, please stop using Postman, just use some Unit Test Runner with some Restful HTTP library, i.e in C# there is RestSharp.

As with all other chapters let me bore you for about 5 minutes on why I am asking this and what you can do about it.

> When I say API in this chapter I am really talking about HTTP based restful web apis, where you make a http call to a specific route and expect a specific response code/data.

## What is API Testing?

This level of testing may be called many things, API Testing, Behaviour Testing, Acceptance Testing etc, really all we want to do is verify that when we call the API with a specific state we get the expected outcome, we often align and drive these tests from the acceptance criteria of incoming tasks/stories. If your AC is bad then your tests will probably be sketchy, but in a way this forces you up front to have good tests and not rely upon some test entity somehow making up the AC on the fly when your code is done.

As mentioned in previous chapters `Unit Testing` specifically tests that a method does what we expect, it is purely focused on code and may not align with any higher level requirements directly. So this does not replace that sort of testing, but acts at a far higher level.

For example you may have an API that lets you create a user, so your code flow would probably be something like:

```
HTTP Request `<api>/user` POST -> UserApiController.CreateUser -> UserService.CreateUser -> UserRepository.Create<User>
```

Depending on what happens on that journey you may end up returning a 200, 404, 401, 403 etc. You would probably write unit tests for your `UserService.CreateUser` as thats doing the heavy lifting here, maybe some tests for a `UserValidator` or something if you have validation, the rest would just be optional for if we have time.

However if we just rely on unit tests we can only verify the validator and service layer bits work, we havent checked that we get the right HTTP responses back or checked the DB ends up in the right state, and we cant easily do that via unit testing.

> Someone will jump in and say you can mock controllers and db stuff and while you can it often ends up being more faff than its worth, even if you make `integration` or lower level `system` tests that focus on the code flow. However here I want to propose we focus on the higher level and instead of testing our code, we test our requirements.

This is where we come to `Postman` which is a tool a LOT of places use to solve this problem, there are others, and some places just dont have tests at this level, all of which have their own problems, and hopefully I will address some of them now.

## Why is Postman bad?

I dont want to pick on Postman specifically, and its actually a very good tool for what it does, but my main stance here is that you don't need it, and any time I join a team which uses it, we end up with major problems when it comes to testing the APIs. Some problems are more in depth than others but I will try to break down the high level problems (not all may apply to you) then go into possible solutions.

### Postman can only interact with the API

This may not seem like a problem at face value, like lets say I want to test that creating a user that already exists returns a 409 (or similar). How do we make sure the user already exists? we could manually fudge the data and tell everyone in the team DONT TOUCH THIS USER, or we could call the user endpoint once with a user that doesnt exist and get that to create it. However then if we do that, what if that call doesnt work? or what if we already added that user from a previous test run?

I am only touching the surface level issues where, but they go deeper, as often most test suites will have daisy chained tests that all NEED to be run SEQUENTIALLY and if any fails the data in the DB will be in an inconsistent state for the tests requiring manual interactions.

> This reliance on the API as the only touch point means that you may be subject to side effects too which you may not want or could impact further tests down the line, for example what if your Create User call raises an event for processing async in another app, you creating test data via this will start triggering other processes that are nothing to do with your test case.

### Postman needs bespoke test code to be of any use

Almost all postman tests will need some JS code internally to actually do the verification and assertions, maybe even some basic 3rd party calls as part of your test, but all of this test is sat inside of postman and needs their tools to access, its harder to debug and refactor or re-use across other tests. It also cant make any use of your existing rich data/service layer which already exists in your repo, as it has no way to call or consume that code.

### These tests are separate to your repo/change management

You are at the mercy of their own branching mechanism and PR process, their tests and logic do not live within your repo with the rest of the codebase, they cant be easily ran from within your repository (will touch on Newman later). They also have a complex environmental management system that means you will often need to ensure you have a lot of config data duplicated between your repo and postman.

### Who *OWNS* the Postman tests

Quite often the testers own the tests in postman, and in some cases will be the sole runners of the tests, this means that the developers never really interact with these tests in any meaningful way, therefore never write any automation for their API changes, its just left to the testers to do and this means lots of code gets pushed up for release before it may even work properly (you would have to manually test it yourself first).

You may work in a place where the testers expect the devs to have some input on the tests and possibly run them, and thats somewhat better as at least you can run them before you push your code up, but as mentioned in the previous example you need to have data setup a specific way or have to have some hacked environment setup to run locally. An even worse scenario is where devs are expected to create and maintain postman tests but the testers run them and complain to devs when things dont work. I worked in a place like this and it was infuriating as it was just such a crippled process where we had to do what would be simple in code in a 3rd party tool jumping through many hoops just so a manual tester could press a play button and say "they passed".

### It cannot verify non API exposed state

As it can only access the endpoints your API exposes (there are some plugins but none ive seen solve this problem) you can only assert on what is returned to you via your request, so what if you want to verify that a success has put a message in a queue, or added an audit log in a database, or even that some computed data has been correctly triggered etc.

You could expose "test" endpoints that are somewhat hidden but thats a rubbish solution and a security concern, also its a waste of effort having to make bespoke endpoints JUST for this 3rd party tool to do something you could do yourself far easier.

### Automation/Reporting requires effort

Most devs/testers will be using the tool manually, however if you are using a build pipeline and want to run it as part of that you will need to use their automated runner Newman which can be a bit tricky to setup as you need to feed it specific vars contextual to postmans environments and test ids etc, once you have them you also may struggle to get the reports out in a format that you want as well as potential issues when the environment you are wanting to run it on may not be externally accessible for whatever reasons.

### IT COSTS MONEY

$14 per user per month with usage limits for the most basic tier, why would you pay this when chances are you already have devs who could do this better and for cheaper.

## Moving away from Postman/3rd Party Tools

The solution to this problem is not a complex one, you just move the tests into your code repository using your existing unit test runner to orchestrate, but add in a RESTful HTTP lib or the built in one to do calls. We already have a rich data layer we can make use of without having to hand roll any data integration code, although you are free to write your own bespoke stuff separately.

If we revisit the example of creating a user mentioned above in code, we could write a test like so:

```csharp
public class UserApiTests
{
    // Assume resolved via test framework dependency management
    public IRepository Repository {get;}
    public EnvironmentConfig EnvConfig {get;}

    [Fact]
    public async Task should_return_409_when_creating_a_user_with_a_taken_email_address()
    {
        // Create our existing user
        var dummyUserData = new UserData {
            Email = "something@somewhere.com",
            // Any additional guff 
        };
        var createdUser = await Repository.Create<UserData>(dummyUserData);
        
        // Make our HTTP Request
        var client = new RestClient(EnvConfig.ApiUrl);
        var request = new RestRequest("/user", RestSharp.Method.POST) 
        { 
            RequestFormat = RestSharp.DataFormat.Json 
        };
        request.AddBody(new UserDetails { 
            Email = dummyUserData.Email,
            // any other guff
        });
        var response = client.Execute(request);

        // Assert stuff
        Assert.Equal(HttpStatusCode.Conflict, response.StatusCode)
    }

    // More tests
}
```

That was fairly easy right? we just called the DB added a user in with the email address, then called create with that email address and verified it was a 409 response.

I have glossed over some pain points here, like how we populate the `EnvironmentConfig` and how we tidy up data after tests are run, authentication etc, but if we ignore that for now there is nothing really magical happening here, if we were to do this with postman we would need to setup the environment stuff too. We wouldnt have access to the DB so would need to just all create before or in a separate test and hope it worked, as well as doing some js code internal to the test to verify bits, all to do the same thing as like few lines of code.

![meme](https://github.com/user-attachments/assets/0fb79a1b-06a8-4220-b583-62d4327736ba)

> I very much wanted to put this meme here, I hope you enjoy it as much as I do

### THE HUGE BENEFITS OF THIS

Let me mention before we continue that this way has sooooooo many upsides: 

- You the can run these tests locally, on a build server, wherever you want (more complex, but will get to that later)
- Tests are atomic and can be run in any order or even in parallel in some cases
- All tests are part of the code repo and undergo same PR/Branching rules as rest of the code
- Tests can be run before code is comitted/pushed/whatever
- It didnt cost you a penny (ignore developer time, which I would argue would end up being more time in Postman anyway)
- Runs VERY fast as it can go directly to data without needing to do HTTP hops/overhead
- You can verify anything via code, check an event went into a queue, an object went into S3/Azure Blob etc

Also you are basically writing a huge regression suite that acts as a safety net for any dev who wants to start doing new work, they pull down their repo, they run all the tests and if it all works, they know they are good to go. You also know the tests should all be passing as its in source control and is probably plugged into your build pipeline so your build wouldnt pass if the tests didnt work.

> This last point is somewhat true for both sides, but anecdotally almost all postman tests ive seen end up being flakey as they get more complex in nature so often they ended up getting turned off on builds or at least flagged as non failing if tests dont pass for whatever reason.

### The New Problems

So as I mentioned in the previous sections there are some problems that you will come up against, and these problems exist in both worlds but you need to solve them in code rather than in postman/whatever.

#### Environment Data
The first major problem you will face is how to get the environment running and setup the environment config. You can hardcode stuff to run on your localhost to begin with and thats fine, but when you want to run it anywhere you will need to have a config file, and that config file needs to be changed per environment. There are a myriad of ways to do this and you are probably already doing something similar in your build pipeline for secrets etc elsewhere so I would just follow whatever pattern you have there to get access to DB connection strings, api urls, test logins etc.

Getting the environment running can be simple of tricky, if you use docker/containers you can just run all of these tests within a docker compose which runs the tests as well as your api/db etc, but if not you can just manually get it running before you run the tests. Again there are many ways to do this and you probably already do this when you deploy your environments on the build server for testing, so just follow that pattern.

#### Cleaning Up Data
This can be as complex as you want it to be, but I have had great success making a `DataHelper` style class in the test project which wraps the `Repository` and tracks what data has been added and automatically removes it at the end of a test being run. As most test runners expose methods/hooks/attributes/interfaces/whatever you can use to get a method executed when a test finishes regardless of it being a success or failure.

You can also use this for streamlining creation of test data which can make your tests more succinct and self cleaning, for example a really basic thing would be:

```csharp
public class DataHelper
{
    public Repository Repository {get;}
    public Dictionary<Type, object> DataTracker {get;} = new Dictionary<Type, object>();

    public async UserData CreateUser(string email = null)
    {
        var randomUser = new UserData() {
            // Use some random lib to populate stuff i.e Faker.Net
        };

        var userData = Repository.Create<UserData>(randomUser);
        TrackCreatedData<UserData>(userData.Id);

        return userData;
    }

    public void TrackCreatedData<T>(object id)
    { DataTracker.Add(typeof(T), id); }

    public async Task ClearTestData<T>()
    {
        if(!DataTracker.ContainsKey(typeof(T))) { return; }

        foreach(var entry in DataTracker[typeof(T)])
        { await Repository.Delete<T>(entry.Value); }
    }

    public async Task ClearTestData()
    {
        ClearTestData<UserData>();        
    }
}
```

You inject this in wherever you want it and then after your test has complete, be it via XUnit fixtures or NUnit Setup or whatever your test running tool uses. As long as you provision your data from the `DataHelper` or at least call `TrackCreatedData` with any data you are provisioning yourself in your test it should get cleared up automatically.

> This may not work 100% depending on how complex your data layer is and if its all databases or queues etc, but assuming you have most of the functionality already in your data layer it should be fairly easy for you to implement.

## Getting Testers Involved

All teams are different, some teams only have manual testers, some have manual testers who know postman, and some have more technical testers who know how to code and do testing stuff. The solution to this problem depends a lot on team composition and skillset as well as what level of involvement the testers want.

> If we assume minimum involvement, they can just look at the build system output and if that says it passed and they are happy your tests all meet the acceptance criteria then you are good. If they want full involvement and know how to code they can literally work on the tests on your branch with you however you want. In reality though when you adopt this sort of approach you are not as dependent on testers, as each dev on the team assumes a small amount of testing responsibility. Personally I find this great as I know before I push anything up it all works as expected, and I have some confidence in the code im checking out, some devs wont like this as its more work for them, but I would argue in the long run it will save you time and having to faff with the test team heckling you about stuff.

There is some middle ground though, with an approach known as `BDD` which I have a whole section about in the `writing-requriements` chapter, and for the previous test example it would look something like this:

```gherkin
@Scenario: Create a user with a taken email address
Given I have an existing user
When I request a user is created using the same email address
Then I should get a 409 response

@Scenario: Creating a user
When I request a user is created
Then I should get a 200 response
And the response should contain the user information
And the user should be saved
And a NewUser event has been sent
```

Thats pretty succinct right? and we even did a more complex 2nd test there assuming we needed to hook into the response data, the database data, an event bus/queue. You couldnt simple do any of that latter test with postman, it would let you interrogate the response object but you couldnt go much deeper, so you would not have much confidence the whole AC was met without further testing, be it manual or integration tests etc.

Assuming the team as a whole (the business, the testers, the devs etc) are happy enough to all use this syntax (known as `gherkin`) then you can get the testers and business to write the above gherkin syntax and the devs (and maybe testers) write the underlying steps which make it run, which we will cover in the next section.

> Also notice I didnt have a `Given` stage on the 2nd test, didnt need one, you dont always need to setup data before hand, so dont worry if you dont have anything to setup, although you may also need to add lines like `Given I am already logged in` which would cover the auth part of the api in real world scenarios etc.

### How to get the BDD stuff running?

It's all well and good me showing you some BDD stuff and saying its great but how do you actually use it?

> In .net there is `Specflow`, in js there is `Cucumber.js` and many other Cucumber variants on other platforms, although here we will just focus on Specflow and the high level bits, you can follow their docs on how to setup the runner and infrastructure bits.

The way it works is that you have 2 parts, the `*.spec` files which contain the gherkin syntax for the tests, then the `Step` classes which have the pattern matching logic, for example if we wanted to target the above bits:

```csharp
[Binding]
public sealed class ApiStepDefinitions
{
    public const string ApiResponseKey = "api-response";

    public TestContext TestContext {get;} // Some imaginary state store for test data

    public ApiStepDefinitions(TestContext testContext)
    {
        TestContext = testContext;
    }

    [Then("I should get a (.*) response")] // Notice the regex passing through vars for reusable steps
    public void GivenTheFirstNumberIs(string statusCode)
    {
        var apiResponse = TestContext.Get<RestResponse>(ApiResponseKey);
        Assert.Equal(statusCode, apiResponse.StatusCode.ToString());
    }
}

 [Binding]
public sealed class UserStepDefinitions
{
    public const string ExistingUserKey = "existing-user";

    public DataHelper DataHelper {get;}
    public TestContext TestContext {get;}

    public UserStepDefinitions(DataHelper dataHelper, TestContext testContext)
    {
        DataHelper = dataHelper;
        TestContext = testContext;
    }

    [Given("I have an existing user")]
    public async Task IHaveAnExistingUser()
    {
        var existingUser = await DataHelper.CreateUser();
        TestContext.Add(ExistingUserKey, existingUser);
    }

    [When("I request a user is created")]
    public async Task IRequestAUserIsRequested()
    {
        var someUserData = new UserDetails {
            // whatever, you could also use specflow | | table syntax to pass data in
        };

        // Make our HTTP Request
        var client = new RestClient(EnvConfig.ApiUrl);
        var request = new RestRequest("/user", RestSharp.Method.POST) 
        { 
            RequestFormat = RestSharp.DataFormat.Json 
        };
        request.AddBody(someUserData);
        var response = client.Execute(request);
        TestContext.Add(ApiStepDefinitions.ApiResponseKey, response);
    }

    [When("I request a user is created using the same email address")]
    public async Task IRequestAUserIsRequested()
    {
        var existingUser = TestContext.Get<UserData>(ExistingUserKey);

        var someUserData = new UserDetails {
            Email = existingUser.Email,
            // whatever, you could also use specflow | | table syntax to pass data in
        };

        // Make our HTTP Request
        var client = new RestClient(EnvConfig.ApiUrl);
        var request = new RestRequest("/user", RestSharp.Method.POST) 
        { 
            RequestFormat = RestSharp.DataFormat.Json 
        };
        request.AddBody(someUserData);
        var response = client.Execute(request);
        TestContext.Add(ApiStepDefinitions.ApiResponseKey, response);
    }
}
```

I havent modelled every step but that should be good enough as an example, its pretty easy to use and you can expose as many steps with flexible re-usable syntax as you want via regex. So you can potentially make a load of steps for common Api interactions and then just let people compose them for new tests without needing to write new steps.

This is not always going to be the best solution for every team but it can work really well if everyone is on the same page. If the team/company is more dev heavy and have less testers then maybe you dont need to put in the extra effort for the BDD part but its always something to think about if you want to align your requirements and test criteria in a common way.

> Regardless of how you do it be it with BDD or in raw code, you are often going to find tests are more valuble and more stable, faster and consistent. It also allows testing closer to the implementation point and more flexibility on how and what you test, so stop using Postman as a crutch as you honestly don't need it for the most part.
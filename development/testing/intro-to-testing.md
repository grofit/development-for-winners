## Software Testing Introduction

### What is it?

Software testing can mean a lot of different things to different people, however these days it is often split into 2 main camps:

- Manual Testing
- Automated Testing

Now the names are pretty descriptive, as you would expect manual testing would contain tasks which require an actual tester to go and do like **smoke testing**. These days it is common to try and automate as much testing as possible, which leads us into automated testing.

This is where we will be focusing most of our time as automated testing is still not understood by a lot of developers who see it as a waste of time or just seem to think of it as some development evil which needs to be avoided. This could not be further from the truth and the more testable *(even if you dont test it)* your code is, the better quality your code will be in most cases.

> Testing can seem like a boring subject but it can save you LOTS of time on a project and make you write better code, even once you know how to test code there is no law stating you MUST test EVERYTHING.

### Automated Testing

So as this is just an introduction we will quickly cover some of the main types of tests which would generally be automated on almost any project, be it a web project an application or a game.

- Unit Testing
- Integration Testing
- System Testing

A lot of developers seem to think that all automated tests are classed as **unit tests** which is incorrect, they are purely a test which targets a method or function. An **integration test** is generally a test which tests a component interaction, and finally system tests would be full end to end tests of multiple components running as they would in the real world.

> There are other types of tests as well such as **acceptance testing** and **A/B testing** and many others but we will not cover them here.

#### Unit Testing

So as mentioned above a unit test primarily focuses on isolating a particular method and verifying it acts as expected within a given scenario, for example:

```csharp
public class Calculator
{
	public int Add(int a, int b)
	{ return a + b; }
}
```

We can easily assume that a + b is going to work correctly, but I am going to write a test for it anyway so you can see how a test would normally look *(if you didn't already)*.

```csharp
using NUnit; 	// NUnit is our test framework

[TestFixture] // This is an NUnit attribute to indicate its a test class
public class CalculatorUnitTests
{
	[Test] // This is an NUnit attribute to indicate its a test method
	public void should_add_numbers_together()
	{
		var expectedResult = 10;
		var a = 4;
		var b = 6;

		var calculator = new Calculator();
		var actualResult = calculator.Add(a, b);

		Assert.That(actualResult, Is.EqualTo(expectedResult);
	}
}

```

Automated tests often require a test runner (which NUnit has, and Resharper has) and require some attributes on your classes/methods to indicate to the runner what is a test and what is actual logic, as you can see above. So if you wanted to run this test you would need to either use the NUnit test runner and provide it the dll which is output from the project containing the above code, or you could simply use **Resharper** and it will let you right click on a file and test or will add nice icons to the left of your test allowing you to run from there.

>You can find more information out about NUnit from their website http://www.nunit.org 
>or Resharper on the JetBrains website https://www.jetbrains.com/resharper

So we run the calculator and confirmed when we add 4 and 6 we get 10. If for some reason that didn't add up to 10 we would get an exception thrown and the test would fail.

I know this is an OVERLY simple example of a unit test, and you are thinking its pointless but lets say we were to now add a divide method to the calculator.

```csharp
public class Calculator
{
	public int Add(int a, int b)
	{ return a + b; }

	public int Divide(int a, int b)
	{ return a/b; }
}
```

Now we could write the same sort of test for it:

```csharp
[Test]
public void should_divide_numbers_together()
{
	var expectedResult = 2;
	var a = 10;
	var b = 2;

	var calculator = new Calculator();
	var actualResult = calculator.Divide(a, b);

	Assert.That(actualResult, Is.EqualTo(expectedResult);
}

```

Again that works ok and everyone is happy, however what if I was to write the test:

```csharp
[Test]
public void should_handle_division_by_zero()
{
	var expectedResult = 0;
	var a = 10;
	var b = 0;

	var calculator = new Calculator();
	var actualResult = calculator.Divide(a, b);

	Assert.That(actualResult, Is.EqualTo(expectedResult);
}

```

That test would fail currently, as we have not handled that scenario, and any other developer could use this method that way. So in order to fix it we would probably check if either number is 0 then just return 0 without doing a division. However the key point to realise here is that your unit tests are here to confirm your method reacts how you expect, to verify the correct use case scenarios and the incorrect use case scenarios.

You could alternatively throw the exception and write your test to verify that the exception is thrown, this is useful if you were to have a database connector class and wanted to throw a `DatabaseCouldNotBeContactedException` or `IncorrectCredentialsException` etc.

> Remember the goal is to have as much test coverage as is deemed sensible, if you have a method which is simply passing through to another method then maybe its not worth writing a unit test, as the test case may be implicitly covered by an integration or system test, but at least think on it and make a concious decision to not add a test.

##### Mocking Overview

Now finally before we move on to a brief overview of integration tests there is something which I want to touch upon which we will get into far more later, and this is handling dependencies and the fundemental reason why **Inversion of Control** is SO important.

Let us use a simple example, to highlight this.

```csharp
public class UserAuthenticator
{
	private IRepository<User> _userRepository;

	public UserAuthenticator(IRepository<User> userRepository)
	{
		_userRepository = userRepository;
	}

	public bool ValidateCredentials(string username, string password)
	{
		var findUserByCredentials = new FindUserByCredentials(username, password); 
		var locatedUser = _userDatabase.FindAll(findUserByCredentials);
		return (locatedUser != null);
	}
}
```

So the above example is a simple user authenticator, it depends upon a `userRepository` and exposes a method to verify if a users credentials are correct. We have made up the `IRepository<T>` which would basically wrap the **data source** interactions and an `FindUserByCredentials` which is a made up query object. Then we basically check to see if the user was found and if so return a true.

> If you want to know more about the generic repository pattern there is a section later on in the book on that topic.

so lets write a simple test for the expected result.

```csharp
using NUnit;
using NSubstitute; 	// NSubstitute is our mocking framework (many others exist)

[TestFixture]
public class UserAuthenticatorUnitTests
{
	[Test]
	public void should_return_true_with_valid_credentials()
	{
		var username = "Joe";
		var password = "p@ssword";

		var dummyUser = new User();
		var mockRepository = Substitute.For<IRepository<User>>();
		mockRepository.FindAll(Arg.Any<IFindQuery<User>>()).Returns(dummyUser);

		var userAuthenticator = new UserAuthenticator(mockRepository);
		var actualResult = userAuthenticator.ValidateCredentials(username, password);

		Assert.That(actualResult, Is.True);
	}
}

```

Notice how the test is pretty succinct, there is no faffing around creating databases and making sure users exist before we run the test. All we are doing here is making a **mock** version of our repository, which lets us tell it how to respond to interactions, so for example where we do `mockRepository.Query(Arg.Any<IFindQuery<User>>()).Returns(dummyUser);` we are telling the mock object that when it receives a call to the `FindAll` method *(which accepts an `IFindQuery<T>`)* it should ignore what is actually passed in and just return the instance of `dummyObject`.

> This may seem a little daunting at first but mocking is one of the most important aspects of unit testing, you can mock any interface and take complete control of it, allowing you to isolate the bits you care about testing. We will cover more on this later when we try to do some real world examples.

There are also some REALLY important things that are worth discussing here before we progress and this is normally where most people would have the epiphany of how coding with ``Inversion of Control`` in mind is better than other ways.

##### Examples of bad code and broken unit tests

So lets pretend in an alternative reality for a moment we decided that we didn't want to have to pass in our repository as its too much effort and **IoC** is something we don't care about *(This is all pretend, don't worry)*.

Now lets say that I have the *GREAT* idea of using a static class for the `userRepository`, so lets just have a quick look how that code would look.

```csharp
public class UserAuthenticator
{
	public bool ValidateCredentials(string username, string password)
	{
		var findUserByCredentials = new FindUserByCredentials(username, password); 
		var locatedUser = UserDatabase.FindAll(findUserByCredentials);
		return (locatedUser != null);
	}
}
```

Now first of all, the code is a lot less, which instantly makes a lot of developers happy, so this cannot be a bad thing right? so lets go and test this approach with a static class.

```csharp
[Test]
public void should_return_true_with_valid_credentials()
{
	var username = "Joe";
	var password = "p@ssword";

	var userAuthenticator = new UserAuthenticator();
	var actualResult = userAuthenticator.ValidateCredentials(username, password);

	Assert.That(actualResult, Is.True);
}

```

The test will fail because the `UserRepository` static instance has not been setup and you cannot mock static classes as they initialize themselves. At this point you will either be thinking *"Yes this makes sense"* or you will be thinking *"Why dont we just setup the database then"*. If you thought the latter then you are no longer unit testing, as the moment you start requiring a real dependency and not a mock, you are moving to an integration or system test. Also assuming you were to go down this route and you setup your database, you now need to add a fake user called `Joe` with a password `p@ssword`.

Hopefully at this point you are starting to realise that although you have saved a few lines of code you have actually created something which is very difficult to test and also requires a lot of extra code to setup.

> Unit tests are all about mocking dependencies, the only way you can do that is by passing in those dependencies when you create the object. So if you start trying to take coding shortcuts the design suffers.

Let us take another quick example, which would be where someone has decided to use service location to satisfy the dependency.


```csharp
public class UserAuthenticator
{
	private IRepository<User> _userRepository;

	public UserAuthenticator()
	{
		_userRepository = ServiceLocator.Resolve<IRepository<User>>();
	}

	public bool ValidateCredentials(string username, string password)
	{
		var findUserByCredentials = new FindUserByCredentials(username, password); 
		var locatedUser = UserDatabase.FindAll(findUserByCredentials);
		return (locatedUser != null);
	}
}
```

Now don't get me wrong that is probably a bit better than the static repository example, but this too will suffer the same sort of issues, as shown:

```csharp
[Test]
public void should_return_true_with_valid_credentials()
{
	var username = "Joe";
	var password = "p@ssword";

	var dummyUser = new User();
	var mockRepository = Substitute.For<IRepository<User>>();
	mockRepository.FindAll(Arg.Any<IFindQuery<User>>()).Returns(dummyUser);

	ServiceLocator.Bind<IRepository<User>>().ToInstance(mockRepository);

	var userAuthenticator = new UserAuthenticator(mockRepository);
	var actualResult = userAuthenticator.ValidateCredentials(username, password);

	Assert.That(actualResult, Is.True);
}
```

Now that may not look too bad, but you are not unit testing as your unit now depends upon a **Dependency Injection** framework being up and running, so you are integration testing at best here. Also your code is now tied to your **DI** configuration object, so lets say you were using **Ninject** then want to move to **AutoFac** *(These are both DI frameworks for .net)* you would have to re-write all code which depends upon this and unit tests.

> More information around **IoC** and **ServiceLocation** can be found in other chapters.
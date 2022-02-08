# Mocking Overview

Mocking is one of the **MOST IMPORTANT** parts of unit testing, and is one reason why you really need to adhere to **IoC** when you are writing code.

Mocking itself is just the notion of creating a mock/fake object to satisfy a dependency, but then you orchestrate it how you want.

Let us use a simple example, to highlight this.

```csharp
public class UserAuthenticator
{
    // Requires a repository
	private IRepository<User> _userRepository;

    // Satisfy it via IoC
	public UserAuthenticator(IRepository<User> userRepository)
	{
		_userRepository = userRepository;
	}

    // Logic that uses the internal dependency
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
using NSubstitute; 	// Using NSubstitute here, but you can use Moq, whatever

[TestFixture]
public class UserAuthenticatorUnitTests
{
	[Test]
	public void should_return_true_with_valid_credentials()
	{
		var username = "Joe";
		var password = "p@ssword";

		var dummyUser = new User();

        // This creates an instance of IRepository<User> which is fake
		var mockRepository = Substitute.For<IRepository<User>>();

        // This tells the mock object to return dummyUser 
        // if it gets a call to FindAll with any argument of the given type
		mockRepository.FindAll(Arg.Any<IFindQuery<User>>()).Returns(dummyUser);

        // Satisfy the dependency with the mock and call the method to test
		var userAuthenticator = new UserAuthenticator(mockRepository);
		var actualResult = userAuthenticator.ValidateCredentials(username, password);

        // Verify that its valid
		Assert.That(actualResult, Is.True);
	}
}

```

Notice how the test is pretty succinct, there is no faffing around creating databases and making sure users exist before we run the test. All we are doing here is making a **mock** version of our repository, which lets us tell it how to respond to interactions, so for example where we do `mockRepository.Query(Arg.Any<IFindQuery<User>>()).Returns(dummyUser);` we are telling the mock object that when it receives a call to the `FindAll` method *(which accepts an `IFindQuery<T>`)* it should ignore what is actually passed in and just return the instance of `dummyObject`.

> This may seem a little daunting at first but mocking is one of the most important aspects of unit testing, you can mock any interface and take complete control of it, allowing you to isolate the bits you care about testing. We will cover more on this later when we try to do some real world examples.

There are also some REALLY important things that are worth discussing here before we progress and this is normally where most people would have the epiphany of how coding with ``Inversion of Control`` in mind is better than other ways.

## Examples of bad code and broken unit tests

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

The test will fail because the `UserRepository` static instance has not been setup with a valid connection to a database etc, and you cannot mock static classes as they initialize themselves. At this point you will either be thinking *"Yes this makes sense"* or you will be thinking *"Why dont we just setup the database then"*. If you thought the latter then you are no longer unit testing, as the moment you start requiring a real dependency and not a mock, you are moving to an integration or system test. Also assuming you were to go down this route and you setup your database, you now need to add a fake user called `Joe` with a password `p@ssword` to your database, then ideally remove it once your test is complete... thats far more code to do than our original mocking approach.

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

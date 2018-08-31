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
	var expectedResult = 5;
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
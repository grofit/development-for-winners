# Things to know about JS

So Javascript is a prototypal language, which has been driven down an Object Oriented route, which is a good thing in a way, but because there is a lot of confusing information out there I want to just go over a whole load of common quirks which will at least get you started on the path to more sensible javascript once you know how to manage/avoid the odd bits.

## Functions and this

Functions are an interesting thing in JS as you dont always know where you are invoking from, which can be very different to languages like C# where you always know what `this` is. So lets just dive into some scenarios to highlight this.

### What the what?!?

```js
// 1. simple function
function SayHello() { return "hello"; }

// 2. but whats difference with this?
var SayHello = function() { return "hello"; }

// 3. or this one?
var SayHello = () => { return "hello"; }

// All of the above could be consumed by
SayHello();
```

So thats pretty confusing right? whats the point of having 3 different ways to define a function?

There is some reason to all this, now just to cover the differences in whats happening:

1. This one is just a simple function called `SayHello`, does what it says but it gets hoisted(?!?!?)
2. This is actually a variable called `SayHello` which is an anonymous function which isnt hoisted
3. This is also a variable called `SayHello` which is an anonymous function which enforces `this` scope and isnt hoisted 

Still not much wiser on the subject? possibly a bit more confused... whats this hoisting business all about? and why is `this` enforcing good?

### Hoisting?!?!?
I wont go too in depth on hoisting as its silly and you will rarely have to care about it, but the idea is that JS isnt actually executed the way you envision it being executed, so lets look at an example of this.

```js
SayHello(); // but we havent defined it yet?? (this actually works)
function SayHello() { return "hello"; }
```

In the above example you probably would be right in pointing out that they are using a method before it has been declared, but because of the crazy world of hoisting that is actually happening is that the function is being raised to the top of the script scope and executed first, this means that although the source code LOOKS like `SayHello` doesnt exist at point of call, when its executed it actually will (go google the subject, its crazy).

So now we have our head wrapped around that lets look at why the second example previously isnt hoisted.

```js
SayHello(); // sadly no hoisting, so this wont work
var SayHello = function() { return "hello"; }
```

So this one wont work, crazy I know. This is because the former approach is using a function declaration, whereas this is using a function expression so its only evaluated when it reaches that line, so when you created functions this way you are basically stopping the hoisting from moving your functions around, which can be handy.

The 3rd example with the fat arrow (thats what they call it in the JS world) is a bit more complex as that tackles a new subject which is known as `this`, so lets put that to one side for now and come back to it in a moment.

## Classes in JS

Ok so ES6 introduced classes, but they only allow public members and dont *currently* offer much benefit over the existing approach to classes from the ES3+ world, so its recommended that you just define your classes in the simpler way so you can control public and private easier.

```js
function MyClass()
{
    var privateVariable = 0;
    var privateFunction = function() {};

    this.publicVariable = 10;
    this.publicFunction = function() {};    
}

var myInstance = new MyClass();
myInstance.publicVariable; // 10
myInstance.privateVariable; // undefined
```

If you are wondering why we are creating a function, and then treating it like a class, thats because in the JS world the notion of what a class actually is (a scoped container of logic and data) is the same as what a function is. As in Js functions provide scope (this is important) and are treated like objects.

So when we do `this.` we are basically making public members which are exposed on the current scope (the function), then when we do `var` we are only scoping it internally so it cannot be accessed outside, however it can be accessed in closures which we will get onto later, as this whole section is going to be one big mental freak out. So lets just jump into the next crazy thing which is `this`, which is applicable when working with classes.

### this (and the fat arrow)

Lets just dive straight into the problem, so we can discuss why its a problem and address it.

```js
function User()
{
    this.id = 10;
    this.name = "Bob";
    this.logUserName = function() { console.log(this.id + "_" + this.name); };
}

var bob = new User();
bob.logUserName(); // 10_Bob
```

Ignoring the hardcoded crazy in there, we can see that we ask bob to give us his username and it outputs his id and name combined, everything is peachy.

However `this` is a transient thing in JS, and just to show how transient it is lets look at a use case with Bob.

#### The problem with Bob

```js
// Imagine we have this on the page
// <button id="show-name-button" name="some-button">CLICK ME</button>

var bob = new User();
var button = document.getElementById("show-name-button");
button.onclick = bob.logUserName;

// Click Button and Console shows: show-name-button_some-button
```

WHAT THE WHAT?!?! If you dont believe me [check here](http://jsfiddle.net/gnyb94c8/1/). You will see that `this` is being changed on the fly, so when the method is invoked from the click callback it is making `this` actually be the DOM element that raised the event.

This is a common thing in the JS world, depending on what is invoking your method and how its being invoked `this` can change and be something completely different to what you think it is, events will overwrite `this` and anyone can overwrite it too, so lets try this example:

```js
var newThis = { id: 55, name: "Not_Bob" };
var thisIsntWhatYouThinkItIs = bob.logUserName.bind(newThis);
thisIsntWhatYouThinkItIs(); // outputs 55_Not_Bob
```

The [bind method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) basically lets you manually overwrite what `this` should be, so anyone could easily call your public methods and shove whatever data they wanted into them. This could be REALLY problematic if you were to do some sort of critical logic which depended on `this`, like if you were to verify the current user `return sendToAuthenticatedWebApi(this.myPrivateDetails)`, as anyone could easily wrap the method and pass in whatever they wanted to replace `this.

> `this` in JS isnt the same as `this` in other languages like c#, it is basically the executing scope rather than the class container, so always be weary when using `this` in the context of a class.

#### self = this

So before ES6 there was no *nice* way to solve the problem, so instead of relying upon this, another approach was formed:

```js
function User()
{
    var self = this; // self is a handle to the class scoped this

    self.id = 10;
    self.name = "Bob";
    self.logUserName = function() { console.log(self.id + "_" + self.name); };
}
```

As you can see we create a private variable called `self` (others sometimes call it `that` or `_this`) and we pass it the scope of the class, this cannot be altered as easily, so this gives us a more robust way to mitigate these issues, as you can use `self` instead of `this` and know you are getting the class instance, you can also still make use of `this` if you want to be able to get the current executing scope i.e a class method that wants to access the class AND the element being triggered.

> There are some other quirks which can cause `self` to not be what you think it is, such as closures with an overriding `self` (we will get to closures shortly) or someone could change `self` on your instance if they REALLY wanted to, but it is a common way to solve the crazy.

#### Enter the fat arrow

As mentioned, before ES6 there was no way to handle this without using `self`, but with ES6 we now have the fat arrow!

```js
function User()
{
    this.id = 10;
    this.name = "Bob";

    this.logUserName = () => { console.log(this.id + "_" + this.name); };
}
```

Using the fat arrow enforces the `this` scope to the containing scope not the execution scope (its a bit more complex but you can look more into that yourself), so when using the fat arrow you are stopping `this` from being changed on the fly, which can be very beneficial when you want to enforce class level scope.

### Constructors?

So hopefully at this point you have realised that we have not mentioned anything about constructors, so if your class wants to be passed anything you need to be able to handle that, and luckily thats pretty simple to deal with.

```js
function User(id, name)
{
    // id and name are classed as private, so we re-expose them as public
    this.id = id;
    this.name = name;

    this.logUserName = () => { console.log(this.id + "_" + this.name); };
}

var tom = new User(22, "Tom");
tom.logUserName(); // 22_Tom
```

So the function arguments act as the constructor here and that there gives us what we need to make a basic class.

### What about static and inheritance etc?

You can do all that in JS, but the moment you try to go too far down the OO rabbit hole in JS you will end up writing more oddities to bend JS' prototypal nature to your OO needs. So if you are going to want to go beyond basic classes I would advise dropping JS entirely and moving onto **Typescript** which will be covered far more later on, as almost every major framework out there supports typescript and even if it doesn't in most cases you are using composition over inheritance so its often easier to just side step this stuff, but if you REALLY want to do it here are examples of what you need to do:

```js
// Basic User class
function User(id, name)
{
    this.id = id;
    this.name = name;

	this.getUserName = () => this.id + "_" + this.name;
    this.logUserName = () => { console.log(this.getUserName()); };
}

// Add static variable
User.someStaticValue = 10;

// Add static method
User.someStaticMethod = () => { console.log(User.someStaticValue); }

// Inherit from User
function Admin(id, name, accessLevel) {
    User.call(this, id, name); // does the inheritance bit

    this.accessLevel = accessLevel;
    this.getUserName = () => this.id + "_" + this.name + "_" + this.accessLevel; // override getUserName
}

var adminUser = new Admin(2, "Phil", 100);
adminUser.logUserName(); // 2_Phil_100
```

There is also the topic on `prototype` and instance methods vs prototype methods as you can actually attach new things to an instance at runtime, but its rare you would want to do that, as its crazy.

## Closures

-- COMING SOON --
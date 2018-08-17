# Model View ViewModel (MVVM)

This pattern originated as a way to manage complex UI interactions such as web apps, but has increasingly found its way into other use cases, such as game development where you have complex views that need to be data driven, because of that it can be seen as a useful pattern regardless of if you are doing game dev, web dev or application development.

One thing to note here is that it can be beneficial to look at MVVM in the web world as a lot of the skills and notions cross over as the high level pattern is generally the same.

- [Knockout JS](http://knockoutjs.com) (A really simple JS MVVM framework)

In unity there was only really 1 great implementation which has been made open source but has not had much traction with pushing it forward, but info on it can be found here:

- [uFrame MVVM](https://github.com/InvertGames/uFrame.Documentation/blob/master/uFrameMVVM/pages/home.md)

## What is it?

This pattern is centred around view separation with reactive contracts that update the view and view model based upon user inputs or progmatic changes.

The main parts of it are:

- Model (The source of data)
- View (The visual element which is driven by the view model)
- View Model (The orchestrator for the view)

As the view and the **VM** are separate the idea is that the view depends on a given VM but the VM has no idea about what the view is doing or what it looks like.

It is easier to think of the view models as contracts and the view requires one to function, but the VM itself has no dependency on any given view, and you may have multiple views for each view model too.

Anyway lets cover the specific parts in a bit more depth.

## Model

The model isn't really that exciting, it is more a notion than a specific class, as it represents the data that the VM needs to drive the view.

So this could be a database, a web service etc but in most cases it will probably be represented in code as a POCO which represents the underlying data.

So for example you may have a `User` model which looks like:

```csharp
public class User
{
    public Guid Id {get;set;}
    public string Firstname {get;set;}
    public string Lastname {get;set;}
    public string Email {get;set;}
}
```

And this data may be applicable to a few different views so the view model would source its data from the user model but would wrap up the interactions at a higher level.

So ultimately the model can be seen as a form of data seed for the VM to function, and as mentioned it may not be a single model, it may be multiple from different sources. As if you had a user profile page you may want to know about the user, the groups they are in, friends they have etc and this could require data from lots of different places.

## View Model

I know this is out of order but its more important to know about the VM before the view, and we will end up covering part of the view in this section anyway.

So this is the brains of the operation, and this would source data from the model and then expose it generally via **actions** and **properties**. I am not sure if those are the real terms, but lets just roll with it for now and hopefully it will make sense.

### Actions

An action would be a method that is exposed to the view to call, so for example lets say you wanted someone to click a button in your view, and have that trigger some logic in the view model. You would want to do something like:

```csharp
public class HelloWorldViewModel : IViewModel
{
    public void SayHello() { ... }
}

public class HelloWorldView : IView<HelloWorldViewModel>
{
    public Button SayHelloButton {get;} // Imagine this is set

    public HelloWorldView(HelloWorldViewModel viewModel) {
        SayHelloButton.BindClickTo(viewModel.SayHello);
    }
}
```

I am making up the code here but hopefully at a high level this shows how the hello button would trigger an action on the VM when it is clicked.

### Properties

A property is more a value that you want to expose to the view, so for example if you wanted to display the current HP of the player you would want to do something like:

```csharp
public class HealthViewModel : IViewModel
{
    public ReactiveProperty<int> Health {get;}
}

public class HealthView : IView<HealthViewModel>
{
    public Text HealthUI {get;} // Imagine this is set

    public HelloWorldView(HealthViewModel viewModel) {
        HealthUI.BindTextTo(viewModel.Health);
    }
}
```

This way whenever the health updates progmatically it would update the textbox value. It is also worth noting that we are using `ReactiveProperty` here which is specifically an **rx** related class, so if you dont know about that it is well worth reading up on in the [Reactive Extensions](../data-patterns/reactive-extensions.md) chapter.

Also we are just making this up here, in real world MVVM frameworks they will probably have their own classes to create a reactive property, or may not require it to be reactive and can work off regular types, but its worth checking this on the specific frameworks.

#### Two way binding

So the above scenario covers us pushing data from the VM down to the view given a property contract, but part of the benefit of the MVVM approach is that data can be pushed from both VM and the view.

So lets change the above scenario to be a character generator, where you need to let the user randomize a name and/or input their own.

```csharp
public class CharacterCreatorViewModel : IViewModel
{
    public ReactiveProperty<string> CharacterName {get;}

    public void RandomizeName() {
        CharacterName.Value = RandomNameGenerator.Generate();
    }
}

public class CharacterCreatorView : IView<CharacterCreatorViewModel>
{
    public InputField NameUI {get;} // Imagine this is set

    public HelloWorldView(CharacterCreatorViewModel viewModel) {
        NameUI.BindTextTo(viewModel.CharacterName);
        NameUI.BindClickTo(viewModel.RandomizeName);
    }
}
```

Again pretending a bit with the syntax, but we can see that we have bound the text from the input field to the `NameUI` so when someone changes the value in there (be it in code or from UI) it will update the VM, and we have also got an **action** where we let the VM randomize the name, which would then filter down to the view.

## Views

The views are purely there to act as the visable/rendered UI for the user to interact with. So in the web world this would often be a **html** file, in Unity it would be a `MonoBehaviour` which acts as the binding layer for the Scene objects and the view model.

It is possible for there to be multiple views for each view model, as you may have a `PlayerViewModel` which has a view for displaying the players avatar in the game as well as a view for displaying the players stats/score on a UI.

As we covered a large chunk of the view in the view model examples and there isnt really much more special to add to the existing use cases. It depends upon a view model, which we represent as a type on the generic `IView` interface we made up, however it is important to look at the binding mechanisms which will often vary per framework.

### Binding actions/properties

This is completely up to the framework you are using, but almost all of them will have some convention for binding to and from UI components and other things in the scene.

For example in knockout js in the web world you would do something like:

```html
<input type="text" data-bind="text: someTextPropertyInViewModel" />
```

Whereas the same could be expressed in the C# world like:

```csharp
public class SomeView : IView<SomeViewModel>
{
    public InputField Text {get;} // Imagine this is set

    public SomeView(SomeViewModel viewModel) {
        Text.BindTextTo(viewModel.someTextPropertyInViewModel);
    }
}
```

They are both expressing the same intent and both views are saying that an input field should have a 2 way relationship with the `someTextPropertyInViewModel`, it just varys how they do it based upon the platform/framework you are using.

### Binding to non UI things

All the examples above have been binding properties to UI elements or binding buttons to actions, but you should be able to bind anything within reason, so if you wanted to bind a `Vector3 Position` to the characters position you should be able to, as you want these reactions to drive anything in the view regardless of if its a UI element or not.

## Benefits of this approach

### View Separation
This is a brilliant thing, imagine you wanted to simulate your game without having to render anything... well now you can, or what if you wanted to unit test the view models logic without needing to depend upon unity... you can do that too.

View separation makes it a lot easier to do a whole raft of better designs and use better design patterns as the view model is a purely data/logic construct and does not depend upon the render layer.

### Reactivity throughout
As this approach really builds upon reactivity at its core you can have a lot of your game notify you on changes, as the whole reactivity is not limited to just the view layer, you can have your VMs raise events when HP hits 0.

## Wrapping it up

As you can see MVVM is a bit complicated and hard to give solid examples on in the game dev world as there are not as many frameworks doing this approach these days. However it is still a great pattern and knowing about it is very useful if you want to do any web/app/ui related work as this pattern is perfect for managing UI states etc.
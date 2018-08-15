### Builder

The builder pattern has a small amount of overlap with the factory in that it will instantiate a type for you however it is slightly more intelligent in this regard. 

Rather than you just requesting an instance you build up the instance (usually in a fluent style manner) then when you are done setting it up you would call the build method and it would be created.

So assuming we use have a class like:

```csharp
public class Person
{
    public string Name {get;set;}
    public int Age {get;set;}
    public bool IsMale {get;set;}
    public List<string> Skills {get; set;}
    
    public Person()
    {
        Skills = new List<string>();
    }
}
```

Then we wanted to create it via a builder we could do:

```csharp
public class PersonBuilder
{
    private string _name;
    private int _age;
    private bool _isMale;
    private List<string> _skills;
    
    public PersonBuilder Create()
    {
        _name = string.Empty;
        _age = 0;
        _isMale = false;
        _skills = new List<string>();
        return this;
    }
    
    public PersonBuilder WithName(string name)
    {
        _name = name;
        return this;
    }
    
    public PersonBuilder WithAge(int age)
    {
        _age = age;
        return this;
    }
    
    public PersonBuilder IsMale(bool isMale)
    {
        _age = age;
        return this;
    }
    
    public PersonBuilder AddSkill(string skill)
    {
        _skills.Add(skill);
        return this;
    }
    
    public Person Build()
    {
        return new Person 
        {
            Name = _name,
            Age = _age,
            IsMale = _isMale,
            Skills = _skills
        };
    }
}
```

I am sure to begin with it looks like a lot of code, but basically we just expose every field in some logical way then populate what we care about and then build the last bit.

So here are a couple of examples:

```csharp
var personBuilder = new PersonBuilder();

var maleMathTeacher = personBuilder.Create()
	.WithName("Mr Green")
	.WithAge(26)
	.IsMale(true)
	.AddSkill("math")
	.AddSkill("teaching")
	.Build();

var femaleGeographyTeacher = personBuilder.Create()
	.WithName("Mrs Red")
	.WithAge(26)
	.AddSkill("geography")
	.AddSkill("teaching")
	.Build();
```

This shows some basic usage but you can also do some more interesting stuff where you could create a builder for a specific subset and keep building without resetting.

```csharp
var preSetBuilder = personBuilder.Create()
	.WithAge(18)
	.AddSkill("running")
	.AddSkill("healing");

var recruit1 = preSetBuilder.WithName("Bob")
	.IsMale(true)
	.Build();

var recruit2 = preSetBuilder.WithName("Jane")
	.IsMale(false)
	.Build();
```

It is a pretty useful pattern to know, it is very useful in automated tests or to wrap up lots of re-used instantiation logic as you can expose your data however you wish, be it a high level method which would populate the properties in a certain way. You can even add extension methods to allow you to have pre-generated versions.

One of the other things to take away from this pattern is the way they return the active object `return this`, which basically allows you to daisy chain the logic together and build up your object. This style of daisy chaining can be used elsewhere to great effect.
# Requirements & Quantifying Work

So we have gotten over the first hurdle of actually creating the notion of tasks so our team can work in parallel, however one area which is often overlooked is the quantifying of tasks. As it is all well and good having a task called *"Create Inventory System"*, but what does that inventory system need to do?

So here is where quantifying your tasks comes in handy, and will also force you to make hard decisions about your project without even having to waste time implementing it, which sounds harsh, but how many times have you or someone you know had a great idea, ploughed weeks/months into something with just a rough idea of what they want, only to eventually come to a dead end realising it doesnt work out. 

> Generally before you start rushing ahead you want to think about what you actually want to create and how to express that in a way everyone can understand, this will often raise technical and process questions ahead of time without you being lost in the detail.

## How people work on tasks

So when someone picks up a task they will look over the requirements for it and want to be able to answer these questions:

* **What is it I am doing?**
* **Why am I doing it?**
* **How will I know if it's done?**

So the *"What is it?"* can be simple or complex, you need to describe what work actually needs to be done, if it ends up being complex it will generally show that your task is probably too big and needs to become multiple smaller tasks. 

The *"Why am I doing it?"* should be pretty straight forward, what does this achieve? In the inventory system example it allows your player to hold items which is pretty important in most games. 

Then finally the *"How will I know if it's done"*, which is what most people struggle with. As you will always have more ideas the more you work on something, but sooner or later you have to say its finished and move on to something else, so how do you measure that completion of the task, how will you know when you have done everything that is required?

> I am sure a lot of you will be thinking that I am over-thinking this problem, and that this is all a big waste of time, which may be correct if you are some game design/developer/audio/modelling savant however I will let you decide for yourself after you have read the rest of this waffle.

## Creating useful tasks using BDD

So here is an example of a pretty whimsical requirement which provides very little actual information but at first glance looks like it may be enough.

```text
Inventory System

Summary:
Need to create an inventory GUI for the player to be able to equip items and trade their stuff.

Additional Information:
Here are some concepts I knocked up showing some examples of how it should look.

<insert some bad pictures and screenshots of other games here>
```

Now that tells us at the bare minimum that we will need to have a GUI display of some kind, and the player needs to be able to equip items and trade items, but it doesn't really quantify the HOW part of our questions above, and only just about covers the WHY section.

So lets look at another example of how to lay this task out in a far better way using a well known pattern for requirements gathering known as BDD (Behavioural/Business Driven Development/Design gotta love the permutations of these acronyms).

```gherkin
Inventory System

As a player
I want to be able to store my items
So that I can equip them
And I can trade/sell them

Given I have an equipable helmet in my inventory
And I have an equipment window with a head slot
When I drag and drop the helmet into the head slot
Then I should no longer have the helmet in my inventory
And should have the helmet equipped
```

> The above syntax is known as Gherkin and is a BDD standard across most technical industries, tools such as Cucumber/Specflow can use these requirements to automate tests, which we will touch on later.

Now the above example is pretty succinct, we follow a pattern to show our motivations for needing this (**"As a _, I want _, So that _"**) and we show some acceptance criteria which covers how we expect it to work (**"Given _, When _, Then _"**).

Now in the example shown the 2 ways, I have had to make some up front decisions now because I am being FORCED to measure completion. 

So I have had to specify that I will be using drag and drop functionality to move items around, and that when an item is equipped it is no longer in the inventory. I have not even covered the trading aspect, nor have I described how items make their way into the inventory, how a user drops items, if they can be stacked, if they are icons or 3d models, and there is no information there discussed about the design of them.

> This shows that we often think about tasks as big blocks of unknown, we associate some keywords to it, like **Inventory**, **Trade**, **Equipment**, **Use**,  **Drop** etc but we rarely think through the underlying functionality to achieve those high level concepts. 

## Breaking down requirements

One major point to touch upon here is that this *Inventory System* we keep mentioning is probably not just a 1 man/skill job. The logical implementation of the handling of items and equipping of items would probably be a programming job, but how the GUI for the inventory looks and feels is more of a UI designers job. 

At this point you can probably see that having just one task here to represent your inventory is not going to work, as it would require multiple people to work on it with varying skills and some may be complete before others, and this single task would have to be HUGE to contain all the information and acceptance criteria needed.

> When writing requirements if you have trouble quantifying some aspect of work done by someone outside of your skillset then just ask them their opinion. Most successful teams will often ask multiple team members with varying skillsets questions about requirements and acceptance criteria.

So if we were to look at splitting this single huge task out into smaller tasks it could be grouped like so:

* **Create logic for adding items to inventory**
* **Create logic for removing items from inventory**
* **Create logic for trading an item**
* **Create logic for selling an item**
* **Create logic for equipping an item**
* **Create design for inventory window**
* **Create design for item icons in inventory**
* **Create design for trading window**
* **Create design for selling window**
* **Create design for equipment window**

There are an awful lot of things which go into this one feature, and each one of those tasks would require quantifying so you know when you run your game each one of those use-cases will be working. Which may seem quite daunting at first, but would you rather spend months of your time churning out code with no clear guide on what you want and just going by gut feel (which also makes it difficult to collaborate) or would you rather spend a week or so making all the difficult decisions up front so you know before you start how much work there is to do, who can do what and when it will all be finished by?

> In most industries there is the notion of a designer who can mock up designs of how things should look to make it easier to think about how things will look and feel, as it can often be hard to make solid judgement calls ahead of time without some rough idea of how you want it to hang together. In those cases you can just ask a designer to rough up a few sketches ahead of time, or factor it into a preliminary sprint of some kind.

There are possibly even more scenarios there as we are not even touching on what should happen when trading and the trade fails or is rejected, or when you cannot equip an item in a slot because it doesn't match (i.e trying to equip a sword into your head slot).

## Stories/Features and Tasks

To take a step back and summarise, so far we have been discussing how to manage tasks, which is great and you can apply the above information in an agile way or just roll your own sensible approach to managing the design and functionality of your game.

If we were to do all this in an agile would though we would probably be looking at having a separation between the story and the tasks that make up the story. So a story should generally be a high level feature you would deploy as a whole, and a task would be a small part of work that can be done in parallel with other tasks to complete the story.

So for example if we change perspective we could create a story called "**Player Trading**" which would have some overlap with the tasks mentioned in the previous but would act as a grouping mechanism for some of them, so it could look like:

```gherkin
Player Trading

As a player
I want to be able to trade my items with players
So that I can swap them for new items or money

Scenario: Player wants to sell item to player
Given I have PlayerA with a BasicSword and no monies
And I have PlayerB with no items and 100 monies
When PlayerA initiates a trade with PlayerB
And PlayerB accepts the trade request
And PlayerA offers his BasicSword for 100 monies
And both players accept the trade
Then PlayerA should have 100 monies and no items
And PlayerB should have a BasicSword

Scenario: Player wants to gift item
Given I have PlayerA with a BasicSword
And I have PlayerB with no items
When PlayerA initiates a trade with PlayerB
And PlayerB accepts the trade request
And PlayerA offers his BasicSword
And both players accept the trade
Then PlayerA should have no items
And PlayerB should have a BasicSword

Scenario: Players wants to swap items
Given I have PlayerA with a BasicSword
And I have PlayerB with AdvancedSword, PotionOfAwesome
When PlayerA initiates a trade with PlayerB
And PlayerB accepts the trade request
And PlayerA offers his BasicSword
And PlayerB offers his AdvancedSword, PotionOfAwesome
And PlayerB accepts the trade
Then PlayerA should have an AdvancedSword, PotionOfAwesome
And PlayerB should have a BasicSword

Scenario: Player cancels trade request
Given I have PlayerA
And I have PlayerB
When PlayerA initiates a trade with PlayerB
And PlayerB rejects the trade request
Then PlayerAs inventory should not change
And PlayerBs inventory should not change

Scenario: Player rejects trade
Given I have PlayerA with a BasicSword and no monies
And I have PlayerB with no items and 100 monies
When PlayerA initiates a trade with PlayerB
And PlayerB accepts the trade request
And PlayerA offers his BasicSword for 100 monies
And PlayerB rejects the trade
Then PlayerA should have BasicSword and no monies
And PlayerB should have no items and 100 monies

Scenario: Game disconnects mid trade
Given I have PlayerA with a BasicSword and no monies
And I have PlayerB with no items and 100 monies
When PlayerA initiates a trade with PlayerB
And PlayerB accepts the trade request
And PlayerA offers his BasicSword for 100 monies
And PlayerA disconnects
Then the trade should be cancelled 
And PlayerA should have BasicSword and no monies
And PlayerB should have no items and 100 monies
```

This may look like a lot to take in, but this as a story goes is touching on just the high level process required for this feature to be complete in the game. 

Notice it is not touching on the technical implementation at all, it is just detailing the process flow and expected behaviour from the perspective of a few different scenarios.

> The first few scenarios which are resolved with intended outcomes are known as **Happy Path** scenarios as those are the paths you want your users to take. However you should also factor in other paths that you ideally dont want to occur but probably will.
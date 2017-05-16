# Some Agile Lingo

There will be lots of buzzwords that wil lbe mentioned throughout the agile chapters, and probably some future chapters. So here is a glossary of sorts as to what each term means.

## Story
A story is basically a high level requirement which details what you need to deliver, a story should contain acceptance criteria which will be covered in more detail later. As stories can be seen as high level features such as *"Create Login System"* this may actually be broken down into smaller **tasks** which cover all the dev tasks for this story, such as:

* Create Login UI
* Create Authentication API
* Store users login token

A story will also generally have an estimate that the team come up with.

## Estimate

This will probably be self explanitory, but the estimate in the agile world is generally an abstract number, often the measurement for an estimate is in effort/story points, such as:

* 1 - Not much effort
* 2 - Average effort
* 4 - A lot of effort
* 8 - This is way too much, break the story up please

So generally you would all estimate on how much effort you think each **story** would require. So our Login story above may be a **4** (A lot of effort) to implement.

> If you are unsure about how much effort is required a good thing to do is start with the easiest story that you think is fairly simple and use that as your baseline, be it a **1** or a **2**, then just work out if your next story is double the effort or half the effort etc.

If you are struggling to estimate at the story level try starting off estimating on tasks, as this is often easier for technical people. A lot of agile gurus will probably *Sigh* here but its better that you are productive with rough estimates rather than just winging it and hoping for the best. 

So when estimating at the task level making a login UI may be **2** (average effort) but making the Authentication Api may be a **4**, with the storing of the users login token being a **1**, so our total story points would be an effort of **7**.

> There are no hard and fast rules about how big or small stories should be, if you prefer to make your stories granular then make them smaller bitesized chunks, if your team knows the domain and technology being used you can often get away with larger stories which are more akin to features.

It is also worth noting that when working on stories people often use the term *play* as if a story/task were a card you were going to play from your hand.

## Sprint

A sprint is a period of time you have to work on a functional goal based upon the stories. Generally these are often 2 weeks in length and you will decide up front what you hope to achieve by the end of it.

## Backlog

This is basically all the stories for a given project to be done, so if you were going to be making **Super Mario Bros** you may have 100 stories to get through and this may take 10 sprints, so the backlog is the maintained list of stories that are needed and it will often be added to in an ongoing way as more stories are added for new sprints but some are not finished, or technical debt is added etc.

## Standup/Scrum

> **Scrum** is actually a flavour of agile but most people use the terms scrum and standup interchangeably.

This takes place every morning and the team on a sprint get together and quickly mention what they are working on and any blockers they have.

You ideally dont want to take up too much time here, and it only needs to be as formal as you want it to be, for teams of 4+ it may be better to quickly stand together and discuss but there is no hard and fast rules.

The MAIN thing to cover here is any **blockers**, as someone else in the team may be able to assist you here.

> A lot of people get hung up on what you worked on yesterday, which isn't important in most cases, as you can see that from the status of stories and previous standups, so if you can avoid talking about that and just focus on any blockers and what your looking at now it should speed things along.

## Blockers

This is something that stops you being able to continue on a story/task, this may be down to you needing some other story/task to be complete before you can proceed, or a third party library has a bug, or even you are just unable to technically solve a problem.

## Retrospective
At the end of a sprint generally you would have a retrospective to discuss what went well and how you can improve things for the next sprint. This can be as short or as long as you want, for larger teams there are small games you can play to help get people talking on subjects, but when its a couple of people this can be as simple as just raising a few points and attempting to work on them next sprint.

## Time boxing

This term is used generally in conjunction with a blocker or some technical investigation, where you basically specify you will spend **X** amount of time looking into something then discussing your findings and working out where to go next. 

So if I were blocked on not being able to get the 3rd party authentication framework to return the right data for my task, I could timebox 3 hours to go and try a different one and report back and see if the other framework solves the problem in a better way etc.

> At a high level this is a useful strategy for investigating better solutions for technical debt, as you can timebox a few hours with various solutions to a single problem then know which one is the best, rather than just arbitrarily picking one from the start and spending all your time on it. You have to be pragmatic but it at least lets everyone know how long you envision your investigation will take.

## Technical Debt

As you are carrying out stories/tasks you will no doubt have to take shortcuts and do some things that you are not happy about, such as hardcoding some config somewhere because you didn't have time to fully finish a config loader, or you may have had to leave an annoying bug in a release as you just didn't have time to solve it before the end of the sprint.

This is where technical debt comes in, rather than just remembering (or forgetting) what technical snafoos need to be improved you should create tasks in the backlog with a specific tag or convention to indicate that it's not a feature based story but a technical improvement that will need to be completed later.

> The notion of technical debt is very important as a lot of teams don't track this, and it gives the false impression of completion when really you may have a lot of problems under the covers. This isn't to say you will always play all the technical debt, but the more unplayed technical debt on a project the harder/slower it can be in some cases to add new features as you will be bolting on top of code which may change heavily.

## Velocity

Velocity is an average of the amount of story/effort points that a team can realistically complete in a sprint. This is more a reporting mechanism but is important to understand for estimating how much you can get through in future sprints.

So when you do your first ever sprint you won't know how much work you can get through (again assuming 2 week sprint). Lets say you assume you think you can do 10 story points worth of work in a 2 week period and by the end of the sprint you have managed to only complete 8 story points worth of work. This would mean that for the next sprint you know roughly how many story points you can complete.

> Velocity is a great indication of team throughput as well as gauging how much many sprints your project may take, it also helps you work out if you can address technical debt within sprints without impacting what you want to deliver.

# What is Agile?

Agile is just the latest in another long line of project methodologies, such as waterfall, prince2 and many others.

One thing that sets Agile apart from others is that its not really that complicated and is geared towards technical projects as it not only covers how to manage the project but how to try and improve and regularly verify the quality of your project.

> There are also many flavours of agile which dictate how the scrum/standup boards are managed, how much work is throughput at a given time and how individuals should work together etc. Really you can make up your own decision on how to work, as long as you are pragmatic and are making sure you plan and test your stuff frequently I think everyone will be happy.

At a high level agile wants more communication, team decisions, good requirements, frequent deliveries and tests of software, and reflection on how to improve things. This all should hopefully sound like common sense and we will touch on more of that as we continue.

> If you are a single developer or in a team <= 3 people then a lot of this stuff will be overkill for you, but some of the good practices in an agile environment (covered later) will hopefully still be applicable to you one-man-armies and smaller teams.

## Steps

So there is a whole agile manifesto and LOTS of information online about each stage and reams of text going into detail about how long you should take on each and who should be involved etc, but at a high level here are the steps.

> Don't worry if you get bored looking through all this and the related buzzwords etc, I appreciate most readers will want technical stuff and not project process guff, but it is useful to know, but feel free to just skip over to the **tooling** and **requirements** pages to see how to put these discussed steps into practice.

### Inception

The inception phase is where you work out what you want to achieve from the next sprint create your **stories** (see the buzzwords for more info) and **tasks**. In this phase you will often be adding new stories to the backlog as well as working out what stories you should be playing in the next sprint.

Some teams will have a form of **Analyst** (usually a Business Analyst in companies) who will generally take point on creating the stories, but depending on your size you may all have to chip in and assist with this, but you generally need to know ahead of time:

* How long is your sprint?
* What is your velocity?
* How many people are working in the sprint?

Once you know this you can get everyone together and work out what the goal of the sprint is and estimate on stories related to the goal, then see how much work you can get done towards that goal in a sprint.

> It is not the end of the world if you dont finish all the work in a sprint, sometimes you will do it all, sometimes you will do less or even more than thought. The main thing is to try and show what you have done at the end of each sprint so everyone can see how far along everything is, which is covered more in retrospectives.

### Construction/Development

The construction phase is where there will be standups every day and everyone is hammering out code to get the the stories complete. This is one of the simplest stages to explain and although there will be a lot of development, there will also be some other things happening in an ideal world like automated builds and deployments as well as testers verifying what has been created against acceptance criteria.

### Deployment/Restrospective

This stage is known as many things depending on the flavour of agile but if you ignore the words and look into what actually happens at this stage (or multiple smaller stages) its all the same thing. You deploy what you have worked on to whatever environments required, and have a retrospective on how the sprint went.

> This is where things like automated builds and technical debt tracking can pay off, as it will speed up your deployments and make it easier to bubble to other environments (more on this later). 

Retrospectives can be as formal or informal as you want, it is recommended that you get some points to try and improve upon next sprint but its really about people just raising any concerns about the way things are going and rectifying them sooner rather than later. So for example you may find that your requirements are lacking in a sprint, so you can raise for them to be improved in a future sprint, or that you have lots of technical debt and its causing issues, so in the next sprint you could choose to play 50% less backlog stories and try to tackle some technical debt etc.
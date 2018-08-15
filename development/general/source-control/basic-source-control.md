# Source control

These days there are plenty of version control systems out there for you to use, such as the extremely popular **git** which is a great all rounder, or more specific ones like **perforce** which handles binary files better out the box.

These days most people will recommend one of the following:

* **git**
* **svn**
* **mercurial**
* **tfs**

They are all great, but **git** and **mecurial** are slightly better because they are distributed version control systems which may not mean much to you, but it is far better as they have additional redundancy and a layer between your code and everyone else copy of the code held on the central repository.

Anyway we will focus on **git** and some of the reasons why you should be using some form of source control if you are not already doing so.

## Why should I use source control

### Backing up

Have you ever lost work off your hard disk, or heard someone else complaining they lost days worth of work because their hard disk has died or corrupted. Even worse there have been indie developers who have had their entire projects stolen because they didnt back it up anywhere.

This is one **MAJOR** reason why source control is important as it gives you a way to backup your current code to somewhere other than your hard disk. By using online providers for your code you can quite easily backup all your code at the click of a button and also retrieve it on any computer you want to continue working.

### Versioning

Every time you commit some code to a source control system you will basically be creating a snapshot of your code base at that point in time, so you can at any time go back to a previous version making it easy to go back in time and see if a bug exists on a specific version of your software.

### Branching

Branching is a step ahead of versioning, as you may want to quickly test something out without breaking your stable version. 

By default you will be on a `master` branch in your version control system of choice and this will contain all your changes until you create a branch off it. When you do make a branch you keep the old code as it was but have an isolated branch to play around and try new things.

If at the end of your work on that branch you want to keep the work you can merge it back into your `master` branch, if you don't want to keep it just delete the branch and return to the previous version of `master`. This is great for quick prototyping and also isolating features which we will get onto later.

> Branching is a huge topic and there is a whole book dedicated to how to use git which is a recommended read if you need more information on the subject [**The Git Handbook**](https://git-scm.com/book/en/v2).

## Using version control systems

When using distributed version control systems (**DVCS**) you will have a central repository (usually on a 3rd party server) and your local repository which knows about the central repository, this way your code is backed up locally AND on a server elsewhere.

This also means that others can access your code if you want to share it with them and they can create their own local repositories.

> When using non distributed VCS you will generally just have one repository on a remote machine that all code is directly comitted into, this means that every time you commit changes it will effect other people, whereas with a DVCS you can commit as many times as you want locally without effecting others until you push.

### Providers

Almost everyone these days will have heard of [**github**](http://github.com) which is a very popular git host. However for free it will only let you host public projects, which is great for people who want to do open source work, however those who want to keep their code private will have to look at other hosts such as [**bitbucket**](http://bitbucket.org) and [**gitlab**](http://gitlab.com) who allow free private repositories.

Some hosts will have file size limits depending on your account with them, also some have additional **git** plugins enabled so you can use **git LFS** which is very useful for large binary files.

## Git Tools

There are a lot of tools for all the various VCS, almost all of them come out of the box with a command line tool where you can do things like:

```sh
git add my-file.txt
git commit -m 'Added my file to git'
```

Some people like the command line, and some like visual tools (I prefer the latter) and there are plenty of them for git.

There are 2 main sorts of approach to managing git in a visual way, one is where you manage it from the file system, where you would be looking at tools like [**tortoise git**](https://tortoisegit.org/) which is free or the built in git shell tools. With these sort of tools you find your git repo on the file system and `right click -> commit` to start doing your stuff.

The other kind is the one where you manage git at the repository level ignoring the file system so you just get a list of all git repos on your machine and you pick which ones you want to carry out actions on. If you want these sort of tools then look at the free [**SourceTree**](https://www.sourcetreeapp.com/) or if you are happy to spend look at [**gitkraken**](https://www.gitkraken.com/).

It is also worth noting that most development IDEs these days come with built in tools for git so you can commit/push/pull etc all without leaving Visual Studio etc.

> I currently use GitKraken but have used tortoise for years as well as trying source tree for a while. They are all very much the same, and for a lot of people the command line will be fine, I very much like the ability to switch my profile in GitKraken though which is helpful when you need to work with git with many different accounts.

## Setting it all up

If you dont mind your projects being visible to anyone in the world then go make a **github** account, if you want your code to be private then I would recommend using **gitlab** and make an account there.

Once you have created your account go and create a repository on there (the sites have their own readmes on how to do this) and once done you will end up being provided a git url which looks like:

```
git@github.com:grofit/development-for-winners.git
```

Once you have this link load up your tool of choice, and you will want to **clone** your repository, this will create a local folder for your source code.

From here you can **commit** your code locally as often as you want and make as many local **branches** as you want, then when you want to make the central repository aware of it **push** your changes up, and if someone else makes changes and wants to share them with you, do a **pull** request.

> The previously mentioned [**Git Handbook**](https://git-scm.com/book/en/v2) details the flow and commands for git a lot better than I will delve into here so feel free to read that for more information on the subject.
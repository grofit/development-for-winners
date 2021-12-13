# Git Flow

Rather than just repeat what others far wiser than I have said, just go look at this link on the subject [**Git Flow**](http://nvie.com/posts/a-successful-git-branching-model/).

![Git Flow Diagram](http://nvie.com/img/git-model@2x.png)

This is probably the best approach to large projects that need to be well tested before going live, the main concept centers around a few key things which we can look at.

> At the heart of this pattern we separate the notion of "What's Live" and "What's In Development" so we can develop and hotfix independently of what is happening across various teams.

### Master & Development

So the main concept is that you have a branch for what is currently live (**master**), and a branch for what is in development ready to be released (**development**). These 2 branches are basically static and will always exist, however there will then be transient branches which are made off these branches for varying scenarios. 

### Hotfixes

So the first one would be creating **hotfix** branches off live which can be pushed directly back up to master, but also relayed into **development**, so both branches get the same fixes. 

### Release

There are also **release** branches which basically would contain all the features required for a given release, and would be a branch off **development**. Once the release has been tested it can be fed into **master** and put live.

One thing to mention here is that because of this separation off the **development** branch you can continue to develop new features and code against **development** without any worry as the **release** branch is isolated and only contains what was put into it at that time. In some cases you may want to do fixes on this branch and merge it back up to **development** as well as putting it into **master**.

### Feature Branches

These are super important, as these are where you will spend most of your time in these branches working on new features. The main idea being that you branch off **development** and make a contextual branch for your body of work such as **adding-profile-page**. Once your work is done the idea is that you can choose when to merge it into development so you could wait until the end of a sprint and merge all features that are complete into **development** then make a release branch off there. Once these branches have been merged into development they can be removed if needed as development is up to date.

It is also worth mentioning here as to why **feature branches** are a good idea, so imagine that you start your work day and are asked to add a new feature to the system, you are 5 hours into it when someone shouts that you need to go do a hotfix on live. Without feature branches where does this half made feature go? you could commit it into **development** but then its always going to be there, and if someone then decides later down the line that they dont need that feature, you would lose hours trying to undo all your commits for this feature from the **development** branch, so it makes far more sense to isolate this feature in its own branch so you can commit to it freely without worrying about effecting **development** or others.

## Other Info

There are other approaches, which we will cover in other chapters, but at the heart of almost all good git patterns there is the following:

* **Ability to isolate and swap work quickly**
* **Ability to track release versions**
* **Ability to not effect others with half done work**

So if you think git flow is a bit overkill then by all means simplify it by not having release branches, or by removing development and just working off feature branches on master. 

However you do it just make sure everyone knows what they are doing, and unless you are a lone wolf developer you will often hit a problem which could have been avoided if you had some separation between whats live and whats in development.
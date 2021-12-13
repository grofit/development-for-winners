# Source Control/Git Patterns

To start with just working on `master`/`main` branch is fine, but as you do more complex work and potentially work with others you will need to come up with some sane branching strategies to make sure you dont end up losing work when you merge in other peoples changes as well as making sure you can partition your experimental changes from your stable ones.

## Common Tips

Rather than go in depth here, we will dedicate a section to each sort of approach, as there are a few main ones and which is best depends on the scenario you are facing. This being said there is quite a lot of good practices you can employ regardless of the branching strategy you are using.

### Always Work On A Branch

So by this we mean if you are going to change anything, don't do it on `main` make a branch from that with a relevant name, then do your changes in that branch. If you decide your changes are fine and should go into `main` then merge it in and delete it.

> It is generally good practice to have a prefix for your branches based on the purpose, for example if I were doing a hotfix maybe my branch should be `hf-login-issue` which indicates that its a hotfix for the login issue, whereas a feature request may be `fr-add-logout-button`. In corporate/open source settings you will probably have a bug/issue number too which you can add in like `hf-399-login-issue`, which can assist in tracability of an issue.

The main reason why we want to work on a branch is that you can isolate your new features from your hotfixes. 

#### Some Example Of Why Its Useful

So if you were to be working on a new feature for your project, but then someone runs over all panicky telling you there is a hotfix that needs looking at. You need to go do a fix on `main` branch, but you are mid way through a new feature which cant be released yet.

If you were working solely on `main` for everything then you either have to bin off all your new feature changes (or if you want to be clever `stash` them), or commit them and do your hotfix on top... neither of these options is good.

So if we were to rewind and assume we made a `fr-522-add-logout-button` branch for our new work, we could commit that up locally, switch over to the `main` branch, create a new branch from there called `hf-666-oh-noes-the-sky-is-falling` and do our fix on there. Once the fix has been verified it can be merged into master and you can hop back onto your *feature branch*.

### Make Sure To Reverse Merge Often

Assuming you are working on *feature branches* (as mentioned in the previous paragraph) you will often run into the situation where someone has made a change and merged it into your source branch (the branch which you took your branch from, often `main`) which means you cannot merge back into `main` without resolving conflicts.

These conflicts happen because your source branch is more up to date than your *feature branch*, so one way to keep on top of this is whenever you do a commit locally, check if any changes have been made on your source branch, and if so merge your source branch into your feature branch.

> This is why its called `Reverse Merging` as rather than you merging INTO the source branch, you are reversing it and putting your source branch into your *feature branch*.

You will still possibly have merge conflicts with this, but this way they are lots of smaller ones that you can often get context on and discuss easier with team members. If you were to not do this and potentially wait until all your work is done then try to merge into your source branch, you could end up with **HUGE** merge conflicts.

> If this is over the span of weeks and there are tens of changes to merge in from various people, it could be really messy as people often forget their changes and move on with things, so the longer you leave it you may end up in a situation where you just end up starting again and manually redoing your changes.
# Distributed Version Control System Basics

So a `DVCS` (i.e git, mecurial) are version control systems where you have an `origin` repository that you clone, and then work locally on your **own** copy of that repository.

## Whats The Difference?

Non **distributed** version control systems, such as SVN, CVS, TFS etc do not keep your repository locally, just the current revision you are working on.

This means that if the SVN server goes down, you can no longer commit any of your work up or get updates from the server, it also means that any branches for the repository are all held on the server, so to swap branch you need to ask the server.

> This may seem like a trival thing, but as we get more into DVCS we will see why having your own repository locally rather than relying on a central VCS can be super beneficial for experimental branching and merging.

## DVCS Terms / Usage

Now we have covered the basic differences between `VCS` and `DVCS` lets cover some terms that are important and how they apply to common usage.

### `CLONE`

Cloning is when you take a cope of the `origin` repo (i.e github) and store it locally on your computer, this clone/copy comes with all information about branches and version history so even if the `origin` server goes down, you can still work on your branch and view the history, and even make new branches etc.

### `FETCH`

This is where you ask the `origin` server to tell you of any new changes to the repository, such as new branches or updates to existing branches.

This doesn't alter your existing branches, it just tells your local repository that there are new changes that you could do with *pulling* into your local branches.

### `PULL`

This is where the `origin` server has newer content on a branch than you do, and you want to `pull` those changes down into your local branch. This could cause merge conflicts if someone else is working on the same branch as you, which you may need to manually resolve, but once the `pull` is finished your branch will be back in sync with `origin`.

### `COMMIT`

So in regular `VCS` we have the notion of a `commit` but as `DVCS` are distributed the way a commit happens does not require the `origin` server (which it would do in SVN for example).

When we commit our code we are adding to our **LOCAL** branch, again this is **SUPER IMPORTANT** its **ONLY** doing this locally, you could be offline on a desert island without internet and you can commit as much as you want, as all you are doing is adding to your **LOCAL** repository, not the `origin` one.

### `PUSH`

This is where we take all our changes on our branches and `push` them up to the `origin` server. This may require you to do a `PULL` first if your branch is not in sync with the `origin` server.

Once you have resolved any conflicts and have *pushed* your code up, the `origin` server is now in sync with your local server, so your changes are both on the `origin` and local repositories.

---

That about covers it for DVCS use cases, there are other things to look into like `Rebasing` which is an alternative to `Merging` but thats more advanced and for most situations you will only need to know about `Merging`.
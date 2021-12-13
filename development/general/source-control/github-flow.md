# Github Flow

Rather than just repeat what others far wiser than I have said, just go look at this link on the subject [**Github Flow**](https://docs.github.com/en/get-started/quickstart/github-flow).

![Github Flow Diagram](https://user-images.githubusercontent.com/6351798/48032310-63842400-e114-11e8-8db0-06dc0504dcb5.png)

This approach is far more lightweight in comparison to the **Git Flow** pattern covered in the previous chapters, however to pull this off successfully you need to be verifying almost all your crucial workflows via automated tests before you go live, as there is little place for manual testing before the go live point.

> You can have additional gates to going live for manual testing but then you end up stifling the throughput and if something fails testing but has already gone into master it's going to go live with someone elses release anyway.

### Master Branch

So the main concept is that you have a branch for what is currently live (**master**), and like **Git Flow** we have feature branches, but they are taken directly from **master** be it for hotfix or new changes.

> One of the problems here is that to a lot of people this approach can seem a lot more enticing as there is less overhead than Git flow and its easier to mentally comprehend whats happening, also things go live quicker. However if you do not have tests in place to verify you are not breaking things all this throughput does is make your live system more buggy, so ALWAYS RUN AUTOMATED TESTS WITH THIS APPROACH.

### Releasing

In this scenario there is no need for varying environments or complex release schedules, as unlike **Git Flow** which generally would have a **release branch** go through `test -> uat -> staging -> live` in this world we just send our **feature branch** to live as soon as its pushed up to master (assuming its passed all automated tests).

**A LOT** of this all hinges on great automated tests and a build server of some sort which runs the tests before a release is signed off for release (be it done by this server or another service).

> We haven't gone into the topic of build servers or continious integration too much yet, so those topics are SUPER important before you can really pull off this approach.

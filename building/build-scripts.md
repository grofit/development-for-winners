# Build Scripts & Processes

Build scripts are a great way to manage your project and environment, it can automate a huge amount of steps you would normally need to carry out when setting up your project and building/testing it.

For example in most cases when starting a new game it is common practice to open up unity and make a new project, then start doing everything in there. This is fine, but what if you decide that you want to write some code in a separate c# class library, or you need to package some xml files into binary for processing in the game. This is where a build script can help arbitrate almost all of the general process behind what a build consists of, as well as setting up your builds for being run within the context of a build server if you wanted to do so.

### Build Script Frameworks

You can pick almost any platform and language you want, there is often a `*ake` style build framework available such as `CMake` for C/C++, `Rake` for Ruby, `Cake` for C#, `JakeJS` for JS `PSake` for Powershell etc. In most cases all these frameworks follow the same sort of approach, where you have a `default` task for your build script and you define other tasks in code for the build script to link/depend upon.

There are also more config driven build scripts which were made popular by Java, they use XML/JSON as the basis for its tasks such as `Ant/Nant`, `Grunt` and even `MSBuild`. In the past few years as NodeJS has become more popular there has been a new streaming based build system known as `Gulp` which has gained a lot of traction.

T> It doesn't really matter what framework you pick, as long as you feel comfortable with the syntax and know it has decent support for your desired platform. For example there is no point using `MSBuild` to write your build script if you are working on linux doing NodeJS.

### Why Use A Build Script?

As mentioned in the opening to this chapter we would use build scripts to automate environmental setup and builds. This means that you can consistently do the same actions and get the same result on every computer. This makes it easier for new team members to start working on your project, as all they need to do is check out your source code, run the build script and it will setup any 3rd party dependencies (i.e database, webservice, local services) template any files for their given environment (i.e change config files to use test servers not live ones) and build the projects and run any associated tests.

If you have a build script setup to do the above then a developer can check out, run it and know within seconds if the project is safe to start working on, and can also setup and teardown their environments as often as they want.

This becomes a lot more powerful when you hook it into a build server which we will get to in the next section but for now the benefits are more for the developer who wants to just get on and work without worrying about setting up configuration files and services etc.

### Using Gulp

`Gulp` is a javascript based build script which can run on all platforms and although it's one of the more complicated build frameworks it is simple enough to get going with and will use javascript syntax which you are probably already familiar with.

#### Setting It Up

There is a bit of setup involved but not much, if you do not know about NodeJS it is worth having a quick read as it is a very versatile platform which is being used more and more for web/app/mobile development as well as an integration layer between other platforms. We will be touching upon nodes package manager (known as `npm`) to manage all the dependencies of the build script, such as unit test runners, msbuild compilers etc.

I> Gulp is one of the more popular build systems at the moment, but it is quite difficult to develop new tasks for unless you are familiar with how node uses streams, we will try and cover the basics without going too off topic but if you feel that you would prefer one of the other frameworks the knowledge you learn here will carry over easily. So don't worry about being stuck with `Gulp` if you prefer another frameworks syntax or approach.

So to get setup:

- Head on over to https://nodejs.org and download the latest version
- Once installed create a `package.json` file in the root of your project
- Add the following content to your file

```
{
	"name" : "your-project-name",
	"private": true,
	"devDependencies": {
		"gulp" : "^3.9.0"
	}
}
```

- Open a command line in the folder with the `package.json` file 
- Run the command `npm install -g gulp`
- Run the command `npm install`

> This tells npm that you depend upon `gulp`, you can use `dependencies` opposed to `devDependencies`, the distinction only matters if you were to release this library to others to use. As we are not and it is a private project and wont be published anywhere we can just rely upon the development dependencies.

> The `npm install -g gulp` may seem redundant as we have gulp setup as a dependency however npm has the notion of global libraries and local ones, so if you want to use the gulp task runner on the command line (which we will) you will need to register it globally so the `bin` folder of gulp is associated with your path.

After following the above steps you should now have a `node_modules` folder which you can ignore from source control if you wish, and that should have a gulp folder in. This is the minimum required to get gulp running, you can test it if you want by creating a `gulpfile.js` and adding the following to it:

```
gulp.task('default', function() {
	console.log('Hello world!');
});
```

If you were to now run the gulp file using `gulp` on the command line it will automatically run that script and output `Hello world!`. 

I> As mentioned npm global installs register the `bin` folders of the libraries on the path, so when you call `gulp` it will automatically look for a `gulpfile.js` within the directory you are running it from and look for a `default` task within there and run it.

#### Something Useful With Gulp

So now we have Gulp setup we can setup a simple task to build a unity project and output it somewhere, this is useful for building consistently.


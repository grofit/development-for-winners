# Javascript

So Javascript is a bit awkward and has some history and baggage, for those of you who use it, you probably are now using Babel to make use of **ES6+** features, or have adopted **Typescript** (which we will cover more later). If you have not used it much before (i.e coming from game dev world) you may hear lots of bad things about it (some true) and instantly dislike it because its different to your normal statically typed languages.

> If you have not done much javascript its recommended you have a look up on it and its syntax as we will be jumping right in on patterns with JS etc

## State of JS in browsers

So almost all browsers currently support ES6+, most even support the mainstream ES7 features like async/await, however there will be a lot of more modern features which have been specced out by the **TC39** but not yet implemented by all browsers.

> Its worth noting that the versioning has recently changed to ES20**, so rather than using ES6/7/8 which was used historically its now ES2017, ES2018 etc, also TC39 is the name of the technical committee who manage the new proposals etc.

Luckily since ES5 almost all new features can be polyfilled if needed, so there was a phase where everyone would polyfill the various bits that were missing for their browsers, but these days it tends to be a case of most people use **Babel** to transpile their bleeding edge JS down to something that can be consumed in an ES5+ compliant browser (everything since IE11). If you were to look at other languages like **Typescript** and **Dart** they let you write your code in their language and then compile it down to JS, often polyfilling certain bits for you.

This then gets onto **Webpack**, **Rollup** and a myriad of other bundling frameworks which can take all your various bleeding edge JS files and convert them into a singular (or multiple) optimized files for consumption. It all gets a bit wacky as realistically you need to know a lot of tech to start making something sensible in the web world, but lets not worry about that right now, lets look at some common approaches to make regular JS a bit more sane and then from there move on to frameworks and the relevant tech.

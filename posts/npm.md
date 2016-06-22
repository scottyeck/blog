---
title: What happens on `npm install`...
date: 2016-06-18
author: Scotty Eckenthal
email: scott.eckenthal@gmail.com
template: article
issue: 1
---

So much of the front-end tooling ecosystem ends up feeling like magic. Open up any *nodejs* project, and you're all but guaranteed to be one `gulp`, `grunt`, `webpack`, or `nodemon` command away from the whole system up-and-running before you. As Front-End people, our job is focused primarily on using (and abusing) these tools rather than understanding them. Such is the end-result, and desired purpose, of a thoroughly moduralized and distributed open-source world, and this is so okay.

That said, we might benefit from having a greater understanding to how these things work or, more importantly, how they got there. They were all bestowed upon us from on high with some variety of an `npm install` command, but 

Package managers and open-source artifact repositories are nothing new. Awareness of tools such as `pip`, `gem`, and `maven` is far and wide and, despite a [few](http://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/) [potential](http://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm) [speedbumps](http://www.haneycodes.net/npm-left-pad-have-we-forgotten-how-to-program/) that might come into play when depending on code that someone else wrote and maintained, it's no secret that open-source code modularization is baked into the way today's web-technology companies build software and, as a result, do business.

Our friend `npm` is no different.



```bash
$ npm install foo --save-dev
```

How about a javascript code block...

```js
function foo(arg) {
	console.log(arg);
}
```

For what it's worth, this is also one of my favorite interview questions. It shows at least one (if not both) of the following:

**It is indicative of a curiosity about the way in which our tooling actually works.** For better or for worse, as front-end people, new tools come across our desks every day. The way in which we approach these tools - either as passive consumers or as interested deconstructors (or even potential contributors) is indicative of a genuine curiosity. 

If the person on the other end understands this system, either they went ahead to do the requisite research to get a deeper understanding as to how their tool worked, or they have played around with it enough that they've done a great deal of this discovery themselves. In either case, the underlying source of their systematic understanding of the tool *hopefully* corresponds to some level of thoughtfulness when deciding on the tooling behind a front-end architecture.
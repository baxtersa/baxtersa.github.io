---
layout: post
title: "Stopify: In-Browser Debugging Abstractions"
date: 2017-06-10 08:53:36
comments: true
---

This is a writeup of a talk I co-gave at
[NEPLS](http://www.nepls.org/Events/31/) on my current research project,
[Stopify](http://github.com/plasma-umass/Stopify). Stopify performs source-to-source
program transformations on the output of to-JavaScript compilers to allow
running code to be interrupted. These transformations provide the abstractions
necessary for us to build in-browser breakpointing and stepping debuggers for
arbitrary languages with compile-to-JS toolchains. Read along for the details!

<br />

Motivation
===

It turns out that portable tablets and Chromebooks are increasingly common
devices. Custom-built Linux desktops will always be there for those of us who
have braved those waters, but for the rest, these smaller devices give users
exactly what they need (an internet connection) and a much lower pricepoint and
mental overhead.

But Chromebooks and the rest offer a _non-native_ environment for installing and
running software. This means you can't run traditional desktop IDEs like
Eclipse, XCode, or MS Visual Studio - you can't even install compiler binaries
you'd need to run these programming environments.

So bringing programming environments _into the browser_ makes sense because it
addresses these issues.

<br />

Client/Server-side Decision
===

Now that we've decided to move programming into the browser itself, we have to
briefly discuss whether to run things client-side or server-side.

Server-side programming environments have downsides we can't overlook:
 - Cost of maintaining server infrastructure for providers of the web-service
 - Cost of server runtime for users of the web-service
 - Bound to the internet (i.e. no offline-mode)
 - The assumption of a reliable internet connection is flat-out unrealistic.

Web-based programming environments offer such a great story to educators that
academic contexts are one of our leading motivators. Especially in these
educational contexts, reliable internet connections are unavailable, so
server-side execution of our programming environment is a non-starter.

So we have settled on a client-side, in-browser experience for developing
programs.

<br />

Current Offerings
===

A number of client-side programming environments exist
([codepen.io](https://codepen.io), [repl.it](https://repl.it), to name a
couple), but they fall short of offering a real execution environment. Many
compile-to-JS languages also offer in-browser execution environments
([BuckleScript](http://bloomberg.github.io/bucklescript/js-demo/),
[Scala.js](https://scalafiddle.io/), [Elm](http://elm-lang.org/try), etc.), but
we're going to focus on the first subset for now. Let's look at CodePen as an
example.

CodePen let's us develop JS, HTML, and CSS animations all within the browser.
This is nice for rapid development, but what happens when your JS program
doesn't behave itself? Here's a screenshot of running an infinite loop:

![infinite-loop](/images/codepen-infinite-loop.png)

To keep the page responsive, CodePen just terminates the infinite loop and
returns execution to the browser. Already, this rules out writing certain types
of interactive programs in this browser environment. But it gets worse! Let's
try to run a _definitely-terminating_ but _long running_ program in CodePen:

![long-running](/images/codepen-long-running.png)

Again, it terminates what it detected to be an infinite loop (???), but it
resumes after the loop **with the wrong results**! This is crazy behavior.
`10000000` isn't even _too_ crazy of a loop bound.

Surprisingly, this is _better_ than how many browser-based environments handle
these types of programs. The language playgrounds mentioned above will happily
crash your browser tab (+1 for multi-process Firefox), and you'll lose all the
code you've written up until that point.

What's going on here? Why is it so hard to offer a simple **Stop** button like
native IDEs provide?

<br />

The Browser Runtime
===

Browsers process a single event at a time off of the JS Event Queue. User
interaction (`onclick`, `onmousemove`, etc.) registers events on this queue, to
be processed by any registered event handlers. Let's look at what happens when
an event is being processed. Below, we see some JS that increments a value `i`
in an infinite loop, and never reaches `foo`'s return statement.

```js
function foo() {
  let i = 0;
  while (true) {
    i++;
  }
  return i;
}

foo();
```

Let's assume this codes executes in the handler for some **Run** event, which
gets enqueued. When **Run** is handled, `foo()` begins executing, incrementing
`i` each iteration of the infinite loop. If the user pushes a **Stop** button,
enqueing more events to be processed, they are queued after the current **Run**
event. The Event Queue is _blocked_ on processing these **Stop** events until
**Run** completes, but this never happens. So while the webpage continues
queuing events on user interaction, they never get handled, so the page appears
unresponsive to user input.

| ------------------------ |
|   Event Queue  | Runtime |
| :------------: | :-----: |
|    **Stop**    | `i=147` |
| :------------: | :-----: |
|    **Stop**    |   ...   |
| :------------: | :-----: |
|  \_\_\_\_\_\_  |  `i=0`  |
| :------------: | :-----: |
|    **Run**     | `foo()` |
| ------------------------ |

<br />
Environments like CodePen try to detect this type of behavior, and terminate the
running event handler so that other events can be processed, maintaining the
page's responiveness.

This means simply providing a **Stop** button doesn't address the issue of
debugging in the browser. The JavaScript being executed within event handlers
must be _instrumented_ in some way, pause to allow other events to be processed,
and eventually resume.

<br />

Do or Do Not
===

Handling this properly is a large engineering effort. There are existing
solutions that don't punt on this work ([Pyret](http://code.pyret.org),
[WeScheme](http://wescheme.org), [Doppio](http://plasma-umass.github.io/doppio-demo/),
and others), and engineer these stopping mechanisms into the compiler and
runtime implementation. So these tools produce instrumented code, and a
knowledgable execution environment to properly handle long-running programs and
provide a stop-button-like abstraction for pausing programs.

**Stopify** is an alternative to these massive engineering efforts,
instrumenting code and offering a debuggable execution environment for
JavaScript emitted by unmodified compilers. Stopify composes with _unstoppable_
output from existing compile-to-JS toolchains to produce _stoppable_ programs.
Furthermore, if these compile-to-JS toolchains provide source maps between the
source language and the JS they produce, Stopify can preserve source locations
and allow breakpointing and stepping _through the source language program_.

Stopify
===

How does Stopify achieve this? There are a number of program transformations
that produce instrumented code enabling these debugging features.
 - **JavaScript generators** (a relatively new ES6 language feature) allow
   programs to _incrementally evaluate_ until the next `yield` point - Stopify
   implements a transformation to inject these `yield` points.
 - **Continuation-Passing Style** is a program transformation that turns all
   control-flow into function applications in tail-position. Functions are
   applied to an additional _continuation argument_, which is a function
   capturing the _rest of the program to be executed_ upon the calling-functions
   completion. Stopify implements this transformation as well, CPSing JavaScript
   directly.
 - **Building a shadow-stack** is similar to the type of engineering involved in
   Pyret, WeScheme, and Doppio mentioned above. This involves maintaining a copy
   of the stack at runtime, so that a running program can be suspended, and the
   running state can be restored from our copy. This is a transformation we are
   looking at implementing in the future.

Stopify builds the foundation of the in-browser **Paws IDE** we are developing.
Paws will be your typical split-pane editor, with Stopify as the special sauce
under the hood. Because of Stopify's composable nature, we can easily support
many (dozens?) of compile-to-JS languages with minimal extra effort. Stay tuned
as we continue development, and watch how the work evolves! We'll hopefully have
a public demo shortly, contact me if you have questions in the meantime!

<br />

---

[Here](/Stopify-NEPLS) are some slides from the talk I presented. Some of the
animations are wonky in the browser, and it turns out minimal text makes for a
good talk, but for not-so-informative slides on their own. Refer to the above
writeup for the details!

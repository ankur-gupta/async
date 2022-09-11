# Review

## There are others
There are many models for asynchronous programming. The following table is an non-exhaustive list
of some of these models.

| Model        | Language   | Comments |
|--------------|------------|----------|
| Callbacks    | Javascript | Very common in [Javascript](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)  |
|              | Python | Callbacks can be used in python, even if less popular than suspendables  |
| Greenlets    | Python     | [Gevent](https://sdiehl.github.io/gevent-tutorial/#real-world-applications) uses [greenlets](https://learn-gevent-socketio.readthedocs.io/en/latest/greenlets.html) |
| Goroutines   | Go         | Run a function synchronously or asynchronously as a [goroutine](https://gobyexample.com/goroutines)   |
| Suspendables | Javascript | [Javascript](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Async_await) inspired `async`/`await` [keywords](https://www.python.org/dev/peps/pep-0492/#abstract) in python |
|              | Kotlin     | Clean & modern implementation of [suspendables](https://www.slideshare.net/elizarov/introduction-to-coroutines-kotlinconf-2017)  |
|              | Python     | Incrementally added to Python 3 since [2001](https://www.python.org/dev/peps/pep-0255/) |

Even though python itself has at least three competing models, namely callbacks, greenlets, and
suspendables, this course only describes suspendables in detail. This is because at the time of
writing this course, suspendables are the newest, trendiest, and arguably (contentiously) the
best model of asynchronous programming in python, so much so that the phrase
*asynchronous programming* often defaults to mean *suspendables*.

Python is an older programming language which initially did not have native support for
asynchronous programming. This support was incrementally added later and as a result the syntax
for suspendables may not be as clean as in some of the newer languages.
For example, Kotlin has a dramatically
[clearer description and terminology](https://github.com/Kotlin/KEEP/blob/master/proposals/coroutines.md#terminology)
including the explicit use of the the word suspendable, suspendion point, and
continuation. They even have a suspending lambda.

Go provides its namesake *goroutine*. In Go, you can define a function and
decide to execute it synchronously like usual (therebey treating the function as a simple function)
or calling the same function asynchronously within a goroutine (thereby treating the function as an
extended function). All of this without requiring any special syntax in the definition of the
function.


## Python from here on
Before we move to discussing the exact details of asynchronous programming in python,
let's review what we discussed until now.

#### Suspendables can be categorized based on control transfer semantics
Suspendables can be categorized into

* Explicit Control Transfer Suspendables, and
* Implicit Control Transfer Suspendables

based on how the control is transferred away from the suspendable. Python provides
generators as an implementation of an explicit control transfer suspendable and coroutines as
an implementation of an implicit control transfer suspendable.

It may be argued, with some merit, that generators do not be even qualify as a model of
asynchronous programming. However, a counter argument is provided in our
[improved approach](/suspendables/control/#improved-approach) pseudocode example and in
David Beazley's [live coding](https://www.youtube.com/watch?v=MCs5OvhV9S4)
of a basic event loop in python from scratch using only generators.
In any case, it is impossible, if not immensely impractical, to discuss coroutines
(which are uncontentiously a model of asynchronous programming) without discussing generators.

#### Suspenables may be implemented as extended functions but not as simple functions
Since we [prefer](/suspendables/syntax/#verbosity-mental-model) suspendables to
have a functional form, suspendable functions cannot be implemented as simple functions.

Both generators and coroutines are implemented as extended functions. Calling
a generator or a coroutine does not execute the internal contents of the suspendable but
instead returns an object that can be used to execute the contnts.

In the rest of this course, we will study python's implementation of generators and
coroutines, while discussing their specific design, syntax, and control flow.

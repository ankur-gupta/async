# Introduction

Learning or teaching asynchronous Python programming is *unreasonably* difficult.
This is unusual for Python which is otherwise so easy to learn that it is often used as the
[first programming language](https://www.coursera.org/learn/learn-to-program?).
This presents an interesting pedagogical problem:

!!! Question
    Why is Python easy but async programming in Python difficult?

The difficulty stems from the following reasons.
*(Feel free to skip ahead and then come back to this page later.)*

### Async programming is *inherently* more complicated
A *regular program*[^1] has a single[^2]
[control flow](https://en.wikipedia.org/wiki/Control_flow) path that a programmer
can walk through mentally in order to reason about the program.
An asynchronous program often has multiple,
implicit control flow paths. A programmer has to think about all the possible paths in
order to properly reason about the program. This makes asynchronous programming inherently more
complicated than regular programming, no matter the choice of programming language.

### Python's troubled history
Python arrived at async programming in incremental steps spread out over roughly two decades.
*Simple generators* were proposed for Python 2.2 in 2001 via
[PEP 255](https://www.python.org/dev/peps/pep-0255/). Since then Python has constantly
updated the semantics and syntax associated with async programming. This trend
continues to the present day (2020, at the time of writing) when *generator-based coroutines* are
being [removed](https://docs.python.org/3.10/library/asyncio-task.html#generator-based-coroutines)
from Python 3.10.

Teaching resources that were once up-to-date became stale over time. Python
programmers had to unlearn the stale semantics and then learn the then-new semantics multiple
times. Unfortunately, this historical cruft is reflected in the teaching materials.
Async Python programming is very often taught in the same order as the incrementally-added
features -- starting with *iterators*, then *generators*, then *generator-based coroutines*,
and finally, *native coroutines*. This style of teaching may be acceptable for a
*History Of Python* lesson but it is unnecessarily disorienting for an async beginner even if
the async beginner is well-versed in Python otherwise.
For example, most teaching materials teach *generator-based coroutines* as an extension of
*generators* only to mention in the end that *generator-based coroutines* are scheduled for
removal.

### Outdated and insufficient terminology
Async Python programming is taught using *generators* (aka *semicoroutines*) and
*coroutines*. The terms *generators* and *coroutines* are old.
Generators first appeared in 1975, were a
[prominent feature](https://en.wikipedia.org/wiki/Generator_(computer_programming)#Timeline)
of the Icon programming language, and inspired
[PEP 255](https://www.python.org/dev/peps/pep-0255/#motivation) in 2001.
Coroutines were invented in 1958 and the first published
explanation appeared in a [1963 paper](https://en.wikipedia.org/wiki/Coroutine) by
Melvin Conway[^3] in context of the COBOL programming language and *punch cards*.
Since then, coroutines have been
[implemented](https://en.wikipedia.org/wiki/Coroutine#Implementations) in many languages with
the implementations differing from each other due to language-specific idiosyncrasies.
As a result, the modern definition of a coroutine is a generalization[^4] of all the different
coroutine implementations.

The general-purpose defintions of a generator and a coroutine are wholly insufficient
when it comes to Python because they fail to
properly distinguish between a Python generator and a Python (native) coroutine.
Generators are considered a [subset of coroutines](https://en.wikipedia.org/wiki/Coroutine#Comparison_with_generators) and coroutines are considered an
[evolution of generators](https://www.python.org/dev/peps/pep-0342/#introduction).
While the preceding statement is historically correct, it is no longer applicable because
Python generators and Python (native) coroutines have now diverged into completely
independent concepts.
Funnily, this failure of definitions eventually comes to light when the programmer
accidentally stumbles upon an [asynchronous generator](https://bugs.python.org/issue27243),
which acts as both a generator and a coroutine at the same time[^5].

More recent programming languages such as [Kotlin](https://kotlinlang.org/) (unveiled in
[2011](https://en.wikipedia.org/wiki/Kotlin_(programming_language))) and
[Go](https://golang.org/) (unveiled in
[2009](https://en.wikipedia.org/wiki/Go_(programming_language))) provide a dramatically
better learning experience compared to [Python](https://www.python.org/) (unveiled in
[1990](https://en.wikipedia.org/wiki/Python_(programming_language))).
For example, Kotlin uses a much clearer
[definition of a coroutine](https://github.com/Kotlin/KEEP/blob/master/proposals/coroutines.md#terminology)
and Go provides a [goroutine](https://gobyexample.com/goroutines) as a primary feature.

### Underhanded treatment of control flow and state
Coroutines and generators are [radical](https://en.wikipedia.org/wiki/Goto#Coroutines)[^6] control
flow devices when viewed from the point of view of the now well-accepted
[structured programming](https://en.wikipedia.org/wiki/Structured_programming) paradigm.
Both coroutines and generators retain their state while allowing the control to be trasferred away
from themselves. However, Python generators and Python (native) coroutines provide very different
types of control flow. A Python generator provides a more definitive transfer of control
compared to a Python (native) coroutine. This difference in control flow is a very
important distinction between a Python generator and a Python (native) coroutine that is rarely
mentioned, if at all[^7].


### Footnotes
[^1]:
    A regular computer program is synchronous, non-concurrent, single-threaded, and single-process.
[^2]:
    The use of (psuedo) random numbers without a known, pre-set
    [random number seed](https://en.wikipedia.org/wiki/Random_seed) can make the control paths
    appear random (or stochastic) to the programmer. As a result, the programmer may have to
    consider multiple control flow paths in order to properly reason about the program.
    We ignore this situation because (psuedo)randomness-induced control
    flow multiplicity is (1) easier to handle, and (2) orthogonal to async programming.
[^3]:
    **Conway**, Melvin E (1963). Design of a Separable Transition-diagram Compiler.
    *Communications of the ACM. ACM. 6 (7): 396–408.*
    [doi:10.1145/366663.366704](https://doi.org/10.1145%2F366663.366704)
    ([PDF](http://melconway.com/Home/pdf/compiler.pdf))
[^4]:
    This retroactive generalization is quite common in computer science where the specific
    implementation of a new concept arrives before the general concept. Once the specific
    implementation is found to be useful, it is generalized into an abstract concept.
[^5]:
    Author(s) of this course predict that the Python syntax will change once more to
    allow `yield from` within `async def` coroutine functions even though it isn't allowed as of
    [PEP 492](https://www.python.org/dev/peps/pep-0492/#new-coroutine-declaration-syntax). This is
    because (1) `yield` is allowed within an `async def` coroutine function
    in order to create an [*async generator*](https://www.python.org/dev/peps/pep-0525/), even
    though the [previous PEP](https://www.python.org/dev/peps/pep-0492/#new-coroutine-declaration-syntax)
    specifically forbade it, and (2) the refactoring concerns from
    [PEP 380](https://www.python.org/dev/peps/pep-0380/#motivation) still apply to
    *async generators*. This would formally complete the separation of generators and
    coroutines in Python.
[^6]:
    It can be argued that coroutines and generators are a form of
    [structured concurrency](https://en.wikipedia.org/wiki/Structured_concurrency) that is
    sufficiently close enough to structured programming and should not be called *radical*.
[^7]:
    It is much more common to find the following outdated distinction -- a Python generator
    only generates values but a Python coroutine can both generate and accept values.
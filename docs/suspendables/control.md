# Control
!!! summary "Control"
    Who gets the control when a suspendable entity suspends execution ?

In our opinion, this key question is often avoided by most other tutorials, which makes 
learning unncessarily difficult. We alluded to this in the 
[beginning](/why-is-it-difficult/#underhanded-treatment-of-control-flow-and-state).

In this course, we use new terminology to answer this question.
Specifically, we subcategorize suspendables on the basis of control flow:

1. *Explicit Control Transfer Suspendables*
2. *Implicit Control Transfer Suspendables*

We use python-like pseudocode instead of real python code throughout this page. 
We cannot use real python code yet because we haven't answered the question of
[syntax](/suspendables/syntax/).

## Explicit Control Transfer Suspendables
Let's re-visit our [Salad & Mashed Potatoes](/suspendables/cooking-is-like-programming/#salad-mashed-potatoes)
example from before. We previously noted that making `mashed_potatoes` a suspendable recipe
would be more efficient because we can execute `salad` while the potatoes are boiling.
In order to do so, we need to break up `boil_potatoes` into three pieces:

!!! note "`boil_potatoes`"
    `start_boiling_potatoes`
    : Put the potatoes and water in a saucepan and set the auto shutoff timer for 15 minutes

    `release control`
    : while the potatoes are boiling

    `finish_boiling_potatoes`
    : Drain the hot water from the saucepan and extract boiled potatoes

We can write our code in one of two ways as shown below.

=== "Example 1"
    <pre><code><strong>function</strong> salad
        chop_lettuce_into_the_bowl()
        chop_tomato_into_the_bowl()
        chop_cucumber_into_the_bowl()<br/>
    <strong>suspendable function</strong> mashed_potatoes
        peel_potatoes()
        cut_potatoes()
        start_boiling_potatoes(minutes=15, auto_shutoff=<strong>true</strong>)
        <strong>release control</strong>  # to the caller
        finish_boiling_potatoes()
        mash_potatoes()
        stir_potatoes_with_butter()<br/>
    <strong>function</strong> main
        mashed_potatoes()
        salad()
        mashed_potatoes()  # resume suspended run<br/>
    main()
    </code></pre>

=== "Example 2"
    <pre><code><strong>function</strong> salad
        chop_lettuce_into_the_bowl()
        chop_tomato_into_the_bowl()
        chop_cucumber_into_the_bowl()<br/>
    <strong>suspendable function</strong> mashed_potatoes
        peel_potatoes()
        cut_potatoes()
        start_boiling_potatoes(minutes=15, auto_shutoff=<strong>true</strong>)
        <strong>release control to</strong> salad()
        finish_boiling_potatoes()
        mash_potatoes()
        stir_potatoes_with_butter()<br/>
    <strong>function</strong> main
        mashed_potatoes()<br/>
    main()
    </code></pre>

In Example 1, control is transferred from `main` to `mashed_potatoes`, which transfers the
control back to `main` once the potatoes are set to boil. `main` then sends the control to
`salad` which finishes its job and returns control back to `main` like any traditional
function. `main` then calls `mashed_potatoes` once again to resume the suspended run[^1].

In Example 2, control is transferred from `main` to `mashed_potatoes`, which then transfers
the control to `salad` while the potatoes are boiling instead of returning control back to
`main`. `salad` finishes its job and returns control back to its caller
which happens to be `mashed_potatoes`. Finally, `mashed_potatoes` then finishes its job
and transfers control back to `main` at the end.

In both examples, the suspendable function transfers the control **explicitly** and
**deterministically** from itself to another known function. In Example 1, during suspension,
the control transfers from `mashed_potatoes` to its caller, `main`. In Example 2,
the control transfers from `mashed_potatoes` to `salad`. In both of these examples,
the programmer knows deterministically which line of the program would execute immediately after
the control is released during suspension without having to run the code at all.

The suspendable `mashed_potatoes` in both examples is an Explicit Control Transfer Suspendable.
The idea of explicit control transfer is best understood in contrast to the implicit control 
transfer.


## Implicit Control Transfer Suspendables
The [Salad & Mashed Potatoes](/suspendables/cooking-is-like-programming/#salad-mashed-potatoes) 
example is not suited to demonstrate implicit control transfer. 

Let's consider a more real-world use case for suspendables -- network calls. Suppose you play a
video game on three different servers -- Server Red, Server Blue, and Server Green. Each server
may have a slightly different copy of your saved game data based on when you last played.
Since you don't remember which copy of your saved game data is the latest, you decide to download
the data from each of the three servers and then inspect the data to determine which is the latest.
For this exercise, we will assume that the game data is simply a textual string and the latest
string is simply the longest one.

### Aside: why asynchronous programming shines
We could do this serially: request the string from one server, wait for it to reply, and
count the string length, then do the same for the other two servers, and then compare the three
sizes. Expectedly, serial operation would be inefficient.
It could happen that the first server we query takes 1 second to respond
while the second and third servers take only 300 and 400 milliseconds respectively. By serially
ordering them in the way we did, we will spend 1700 milliseconds waiting.

We could use three separate processes but that would be wasteful. Each process takes separate
memory to run an independent python interpreter. Each process also needs time to be setup.
We could use three threads within the same process which would solve both of these problems.
Threads share memory and can be started and ended much faster than processes. The problem becomes
worse for both the process and thread based solutions if we had thousands of network calls to
make instead of just three. Even though threads are less wasteful than processes, creating
thousands of threads is inefficient and even unachievable depending on the hardware.

Servers have internal latencies and transferring data over the network
takes time. We don't need to perform a lot of computation, we simply need to receive the data.
There is no inherent need to create threads/processes in order to handle the incoming data.
The reason why we have to create multiple threads/processes before asynchronous programming
was simply a structural restriction in programming constructs. A real world example of
how asynchronous programming can outperform multithreading is
[Nginx vs Apache](https://stackoverflow.com/questions/12924124/why-would-i-choose-a-threaded-process-based-approach-vs-asynchronous-web-server).

### Naive approach
Using the same constructs as our cooking example above, we begin with the following naive
approach.

<pre><code><strong>suspendable function</strong> get_length_from_server_red
    request = create_request(server=red)
    <strong>release control</strong>
    request.wait()
    <strong>return</strong> len(request.result)

<strong>suspendable function</strong> get_length_from_server_green
    request = create_request(server=green)
    <strong>release control</strong>
    request.wait()
    <strong>return</strong> len(request.result)

<strong>suspendable function</strong> get_length_from_server_blue
    request = create_request(server=blue)
    <strong>release control</strong>
    request.wait()
    <strong>return</strong> len(request.result)

# Initiate all requests
get_length_from_server_red()
get_length_from_server_green()
get_length_from_server_blue()

# Resume suspended functions
red_length = get_length_from_server_red()
green_length = get_length_from_server_green()
blue_length = get_length_from_server_blue()

# Find the server with the longest (latest) data
max([('red', red_length),
     ('green', green_length),
     ('blue', blue_length)],
    key=<strong>lambda</strong> x: x[1])
</code></pre>

We initiate requests using `create_request` function and then wait for the server to return
data using `request.wait`. Since the functions[^2] `get_length_from_server_red`,
`get_length_from_server_green`, `get_length_from_server_blue` are suspendable in our
pseudocode syntax, we first call them knowing that they would suspend just after making the
request but before waiting for the data to return. This suspension allows us to place all
three requests without having to wait for any of them finish. Then, we resume[^3] all suspended
functions serially in a particular, distinguished order: red, green, and then blue.

This naive approach is more efficient than serial approach in which we wait for the first server to
finish before we make a call to the second server and then to the third server. If the red,
green, and blue servers respectively take 1000, 300, and 400 milliseconds to finish, then by
the time we hear back from the red server, the green and blue servers have already sent back the
data. Thus, we have to wait only a 1000 milliseconds. Ability to release control reduces the
experienced wait time from 1700 milliseconds to 1000 milliseconds[^4].

#### Room for improvement
This naive approach is still inefficient. Just after the first 400 milliseconds or so,
both green and blue servers have returned data but red server has not. We have to wait another
600 milliseconds until red server returns data to compute the length of red server's result
before we can process the result of green and blue servers. This is because we ordered our
resumed calls to suspended functions in a particular, distinguished order: red, green, and then
blue. Had we ordered the resumed calls in the order green, blue, and red, then at the 400
milliseconds mark, we could've compared the length of green server data and blue server data
while still concurrently waiting for the red server to return. Out of the two comparisons needed
to find the maximum of 3 numbers, we could've performed one comparison (between green and blue)
while the red server took its time. This would've been more efficient.
This improvement would be barely noticeable for our 3-server example but could produce serious
gains if we had thousands of networks to query or if the post-network call task was more
complicated than simply finding the largest number.

How can we improve upon our naive approach? Reordering the resumed calls to be (green, blue, red)
is not a practical solution because we will never know the server wait times beforehand. Let's
see how we can improve in the next section.


### Improved approach
Let's say we have the ability to check whether a `request` has returned data or not. Let's assume
that `request.result` remains `None` while it's still waiting for the data and becomes a
`str` once the data has returned (with an empty string `''` indicating that request returned with
zero data).

<pre><code><strong>suspendable function</strong> get_length_from_server_red
    request = create_request(server=red)
    <strong>while</strong> request.result <strong>is None</strong>:
        <strong>release control</strong>
    <strong>return</strong> len(request.result)

<strong>suspendable function</strong> get_length_from_server_green
    request = create_request(server=green)
    <strong>while</strong> request.result <strong>is None</strong>:
        <strong>release control</strong>
    <strong>return</strong> len(request.result)

<strong>suspendable function</strong> get_length_from_server_blue
    request = create_request(server=blue)
    <strong>while</strong> request.result <strong>is None</strong>:
        <strong>release control</strong>
    <strong>return</strong> len(request.result)

# Suspend-resume loop until all requests are done
waiting = ['red', 'green', 'blue']
max_length = (<strong>None</strong>, 0)
<strong>while</strong> waiting:
    # Check to see if the server returned data
    server_name = waiting.pop()
    <strong>if</strong> server_name == 'red':
        length = get_length_from_server_red()
    <strong>elif</strong> server_name == 'green':
        length = get_length_from_server_green()
    <strong>elif</strong> server_name == 'blue':
        length = get_length_from_server_blue()

    <strong>if</strong> length <strong>is None</strong>:
        # If data was not returned yet, push the server name
        # to the queue for another check later.
        waiting.append(server_name)
    <strong>else</strong>:
        # If data was returned, compare with the maximum length up to now.
        <strong>if</strong> length > max_length[1]:
            max_length = (server_name, length)
</code></pre>

In each of the `get_length_from_server_<color>` functions, we first check to see if the request
returned data or not. If data was not returned, we simply release control back to the caller with
a `None` to indicate that the function is suspending/releasing control but is not finished.
If the data was returned, *i.e.*, `request.result` was not `None`, we return the from the
function for good with the length of the result.

In the main part of the program, we repeatedly check whether each of the three suspendable
functions have finished for good or not. If a suspendable function returns with a non-`None`
value, we interpret it as a length of the result and immediately compare it against the maximum
known length up to now. In doing so, we don't need a final call to `max` at all like we did in
the naive approach. Further, we process the result of a suspendable function as soon as it is
done.

As before, if the red, green, and blue servers respectively take 1000, 300, and 400
milliseconds to finish, then at around 300 millisecond mark, we will update `max_length` to be
`('green', green_length)`. Then, at around 400 millisecond mark, we will compare
`green_length` against `blue_length` without having to wait for red server to return data.
Finally, at around 1000 millisecond mark, the red server would return data and we would compare
the larger of `green_length` and `blue_length` against `red_length`.

#### Too common to leave unformalized
It turns out that the situation above is extremely common. In so many real-world situations, we
do not know, in an *a priori* manner, the order of completion of suspendable functions. Thus,
we cannot hardcode the order in which we resume calls to the suspendable functions. We must
loop over the suspendable function to repeatedly resume calls. This loop is called the
*event loop*. The `while waiting:` loop in the pseudocode above is a very rudimentary form of the
industrial-strength event loop that you would find in asynchronous programming packages such
as [asyncio](https://docs.python.org/3/library/asyncio.html) or
[uvloop](https://github.com/MagicStack/uvloop).

It is possible for every programmer to write their own custom loop. But in a real-world scenario,
suspendable functions call other suspendable functions which call other suspendable functions
and so on. Keeping track of all suspended calls and then having to resume them at the right time
is a very involved exercise which is best undertaken by the programming language itself. Python 3
provides, as a first class feature, the syntax and packages needed to run an event loop without
having to write a custom event loop yourself. That said, third party event loops are available,
such as [uvloop](https://github.com/MagicStack/uvloop) and
[trio](https://trio.readthedocs.io/en/stable/). You can even write a custom loop from scratch
as demonstrated in a [live coding session](https://www.youtube.com/watch?v=MCs5OvhV9S4)
at PyCon 2015 by David Beazley[^5].

### How does this relate to Implicit Control Transfer Suspendables?
The suspendable functions in the
[improved approach](/suspendables/control/#improved-approach) transfer control to the
`while waiting:` loop. In order for the event loop to coordinate the work, the
suspendable functions must transfer control back to the event loop and not to any other entity.
If we ignore that the `while waiting:` loop is a part of the end-user-written code, we could
argue that the control is transferred from one suspendable function to another suspendable
function, albeit implicitly via the event loop.
In other words, the `get_length_from_server_<color>` suspendable functions contain the end-user
code and the `while waiting:` loop is just a coordinator that exists just to help facilitate
efficient execution of the end-user code. In real python code, the ingredients necessary to run
an event loop are indeed provided by python syntax and either standard or third-party library,
which makes our claim of implicit control transfer more legitimate.
We can argue that the suspendable functions that run within an event loop,
**implicitly** and **non-deterministically** transfer control amongst each other.

You could reasonably argue that the pseudocode in the
[improved approach](/suspendables/control/#improved-approach) actually uses explicit control
transfer. This is indeed correct, both in the pseudocode example above as well as in real
python code, which we will see later in the couse. Implicit control transfer is indeed built using
explicit control transfer as a building block within an event loop.

## Explicit vs Implicit Control Transfer Suspendables
Some suspendable functions such as the
[Salad & Mashed Potatoes](/suspendables/control/#explicit-control-transfer-suspendables)
example do not benefit for an event loop[^6]. Thus, there is no need to force such functions to
run within an event loop.
Some other suspendable functions, such as those involving network requests or disk I/O,
almost always require an event loop to run efficiently. It is rare for these functions to be run
outside an event loop, unless it is for
[educational purposes](https://stackoverflow.com/questions/52783605/how-to-run-a-coroutine-outside-of-an-event-loop).

While there is a natural distinction between the type of suspendable functions that require
an event loop and those who don't, there appears to be no reason why the syntax for explicit
and implicit control transfer suspendables should be different. It is somewhat reasonable to
expect that a programming language provide only one type of suspendable function with the same
set of reserved keywords for both explicit and implicit control transfer and that a
suspendable function's control transfer can be made explicit or implicit based on whether
or not it interacts with an event loop.

This happens to *not* be the case with python, as of writing this course. The following table
describes how python 3.8.5+ bifurcates the concept of suspendables into two specialized
entities, each with its own set of keywords.

| Item                        | Pseudocode (Explicit & Implicit) | Python (Explicit) | Python (Implicit) |
|-----------------------------|----------------------------------|-------------------|-------------------|
| Name                        | Suspendable                      | Generator         | Coroutine         |
| Requires event loop         | Yes & No                         | No                | Yes               |
| Function definer            | `function`                       | `def`             | `def`             |
| Suspendable modifier        | `suspendable`                    | -                 | `async`           |
| Control transfer to caller  | `release control`                | `yield`           | -                 |
| Control transfer to another | `release control to`             | `yield from`      | `await`           |

Explicit control transfer suspendables become python generators and implicit control transfer
suspendables become coroutines[^7]. The reason for this bifurcation of suspendables into explicit
and implicit control transfer suspendables is complicated involving syntactical, historical,
and cleaner design considerations. Just after [PEP 342](https://www.python.org/dev/peps/pep-0342/),
the same `yield from` syntax was used for both generators (explicit) and coroutines (implicit).
But, using the same syntax for both generators and coroutines created
[problems](https://bugs.python.org/issue24400) and a lot of confusion, as mentioned in
[PEP 492](https://www.python.org/dev/peps/pep-0492/#rationale-and-goals). PEP 492 finally
separated coroutines from generators. We will discuss some of these syntactical, historical,
and design considerations for both generators and coroutines later in the course.

One weird artifact of this bifurcation is that python may allow a suspendable to be
both explicit and implicit at the same time. This is what python calls
asynchronous generators, as described in this [bug report](https://bugs.python.org/issue27243)
and discussed in
[PEP 492](https://www.python.org/dev/peps/pep-0492/#why-aiter-does-not-return-an-awaitable).
We will discuss this oddity later in the course.

## We're not done yet
Before we begin discussing python generators and coroutines, we need to solve the problem of
defining unambiguous syntax for suspendables.

## Footnotes
[^1]:
    Wary readers may already have noticed the ambiguity in the second call to
    `mashed_potatoes`. How would we differentiate between a resumption of a previously
    suspended run and a second independent run of a suspendable function? This is exactly why
    we had to use pseudocode for our examples. We tackle this question of unambiguous syntax
    and semantics [next](/suspendables/syntax/).

[^2]:
    You may be wondering why did we have to write three functions instead of just one function
    `get_length_from_server(server)` that can simply accept the server name. The reason is
    the same as the one in the previous footnote[^1]. If we had just one function, we would have
    even more trouble differentiating between the resumption of a previously suspended run versus
    a new, independent run.

[^3]:
    This is yet another syntax problem. The first call to `get_length_from_server_red` does not
    expect a returned value but the second call does. How would we know when to "catch" the
    returned value from a suspendable function if the suspendable function has multiple
    `release control` statements or if the function calls other suspendable functions which
    could call other suspendable functions. We need an unambiguous syntax to handle such
    issues as discussed [next](/suspendables/syntax/).

[^4]:
    We assume that all other processing performed on our side (including creating requests) is
    essentially instantaneous.

[^5]:
    David Beazley's [live coding of an event loop](https://www.youtube.com/watch?v=MCs5OvhV9S4)
    is highly recommended in spite of the following caveats. Please be aware that this video
    uses an alpha version of Python 3.5 and the old `yield from` syntax intended for
    [soon to be deprecated](https://docs.python.org/3.10/library/asyncio-task.html#generator-based-coroutines)
    generator-based coroutines. Generator-based coroutines are an enormous source of confusion
    as discussed in the [beginning](/why-is-it-difficult/#pythons-troubled-history) of this course
    and this course does not describe them for this reason. Further,
    [PEP 492](https://www.python.org/dev/peps/pep-0492/#new-coroutine-declaration-syntax) makes it
    illegal to have a `yield` or `yield from` within a native coroutine. See this
    [footnote](/why-is-it-difficult/#fn:5) to go one step further down into the deep hole.

[^6]:
    A memory-efficient, boundless iterator (such as Python 3's `range`) is a common real-world
    example of a suspendable function that does not benefit from an event loop.

[^7]:
    We are purposefully ignoring the existence of generator-based coroutines. See footnote
    [5](/suspendables/control/#fn:5) on this page.

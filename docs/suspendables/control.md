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
The [Plane Ticket](/suspendables/buying-a-plane-ticket/) example is the perfect example to 
demonstrate the need for implicit control transfer suspendables. We previously discussed this 
classic example to show the benefit of asynchronous programming over serial and parallel 
programming. We will now use this classic example to demonstrate the need for implicit control 
transfer suspendables as well.

The [pseudocode](/suspendables/buying-a-plane-ticket/#not-parallel-asynchronous) we prevoisly 
saw is simply the naive implementation shown below. We will now discuss the drawbacks of the 
naive implementation and then provide an improved implementation. 

As we discussed before, the pseudocode purposefully has janky syntax, as we will explain in 
the next section on [syntax](/suspendables/syntax/). For now, we will ignore the syntax issues and
focus on control transfer.

=== "Naive implementation"
    ```python title="Pseudocode"
    suspendable function get_price(vendor)
        request = create_request(vendor)
        release control
        request.wait()
        print(f'Received price from the vendor {vendor}!')
        return request.result
    
    # Initiate all requests
    get_price(red)
    get_price(green)
    get_price(blue)

    # Resume suspended functions and wait for them to complete
    red_price = get_price(red).wait()
    green_price = get_price(green).wait()
    blue_price = get_price(blue).wait()

    # Find the best price
    # Could we have started this computation with just two prices?
    min([red_price, green_price, blue_price])

    # Did you notice the janky syntax in this pseudocode? This is why we need 
    # to answer the syntax question in the next section.
    ```

=== "Improved implementation"
    ```python title="Pseudocode"
    suspendable function get_price(vendor)
        request = create_request(vendor)
        while request.result is None:
            release control
        print(f'Received price from the vendor {vendor}!')
        return request.result
        
    # Suspend-resume loop until all requests are done
    waiting = [red, green, blue]
    best_price = infinity
    while waiting:
        # Check to see if the query has finished
        vendor = waiting.pop()
        price = get_price(vendor)

        if price is None:  
            # If query didn't finish yet, add it to the waiting list again.
            # We'll check on it again.
            waiting.append(vendor)
        else:
            # If query returned, start comparing. No need to wait for other
            # queries to finish.
            if price < best_price:
                best_price = price
    
    # Did you notice the janky syntax in this pseudocode? This is why we need 
    # to answer the syntax question in the next section.
    ```

#### There is still room for improvement
Even though our naive implementation of the asychronous approach was much better than 
the serial and parallel approaches, the naive implementation is still inefficient. 

We need to perform a few comparison operations in order to compute the minimum of three numbers. 
These comparison operations also cost time. In the naive approach, we waited for all three 
asynchronous queries to finish before we computed the minimum price. This idle waiting is wasteful.

Just after the 400ms mark, we have the prices from Vendors Green and Blue but we don't have the
price from Vendor Red. We needed to wait another 600ms before we obtained the price from Vendor Red.
We could have computed the smaller of the blue and green prices during that 600ms.

However, the naive approach waits for the queries to finish in a very particular, distinguished 
order - it waits for red, then green, then blue. This forces us to idly wait during that 600ms 
period. If we re-order the vendors to wait for green first, then blue, and then red, we may be able 
to use that 600ms otherwise idle time to compare the blue and green prices. But, even 
re-ordering the queries is not a practical solution because we may not know the vendor latencies 
pre-emptively.

The benefit of utilizing the idle time to perform comparisons seems overkill for our toy example 
comprising of only three queries. But, imagine the real-world case of our own Kayak-type website.
We will to perform thousands of comparisons among thousands of prices in order to compute the 
mimimum price. It would be noticelably slow to perform the thousands of comparisons only after 
all the queries have finished.

### Improved implementation
The main difference between the naive and the improved implementation is that the improved 
implementation adds a `while` loop. Improved implementation does not wait for a query to 
complete. The `while` loop checks (but doesn't wait) to see if a query has completed. If it has 
completed, the loop updates the current minimum price. If the query has not yet completed, 
the loop puts back the query in the waiting list to be checked again at a later time.

The `while` loop runs really fast compared to the queries. Therefore, roughly around 300ms mark, 
the loop will discover that the query to the Vendor Green is finished and the loop will remove 
Vendor Green from the waiting list. Then roughly, at 400ms mark, the loop will remove Vendor Blue
from the waiting list and compare the prices from the Vendors Green and Blue to compute the current
minmum price. This comparison between the prices from the Vendors Green and Blue happens much 
before the query to Vendor Red finishes. Finally, at 1000ms, the loop will have gone through all 
the queries and will have computed the bets price.

### This is too common a scenario
Looping through events is an extremely common scenario and for the same reason -- in real life, we 
cannot know, in an *a priori* manner, the chronological order of task completion. Therefore, we 
need to loop over the tasks to determine which task needs to be assigned to the processor.
This loop is called the *event loop*.

The pseudocode `while` loop we just disucssed is called a rudimentary form of the
industrial-strength _event loop_ that you would find in asynchronous programming packages such
as [asyncio](https://docs.python.org/3/library/asyncio.html) or
[uvloop](https://github.com/MagicStack/uvloop). 

Every programmer can write their own custom loop; after all, we just did so in pseudocode. 
So, why have the programming language provide whole machinery to run suspendable tasks in an 
event loop? 

Because real-world scenarios are more complicated. For example, a suspendable function could call
another suspendable function, which could call another suspendable function and so on. An event 
loop needs to keep track of all these suspendable functions properly and give all of them some 
processing time. 

Since this scenario is too common, Python 3 provides, as a first class feature, the syntax and 
packages needed to run an event loop without having to write a custom event loop yourself. 
That said, third party event loops are available, such as 
[uvloop](https://github.com/MagicStack/uvloop) and [trio](https://trio.readthedocs.io/en/stable/). 
And, of course, you can write your own custom loop 
from scratch as demonstrated in a [live coding session](https://www.youtube.com/watch?v=MCs5OvhV9S4)
at PyCon 2015 by David Beazley[^2].

### How does this relate to Implicit Control Transfer Suspendables?
The suspendable functions in our
[improved implementation](/suspendables/control/#improved-implementation) transfer control to the
`while` loop. The event loop can only coordinate the work if the control is transferred from the 
suspendable function to the event loop. If you transfer the control to some other entity besides 
the event loop, then the event loop cannot coordinate the work properly. 

But, the event loop typically doesn't really do much work for itself. Its main job is to 
transfer the control from one suspendable function to another. We could argue that the event loop
facilitates control transfer between suspendable functions. In other words, the control is 
_implicitly_ transferred between suspendable functions via the event loop.
In real python code, the event loop is nearly invisible to the end user. As a result, the end user 
perceives the control being transferred between their user-written suspendable functions. 

It often not be possible to pre-emptively tell which suspendable function (among a whole
list of suspendable functions waiting for resumption) will finish first. Thus, it's not possible
to deterministically say which suspendable function will transfer the control to which other 
suspendable function, via the event loop. In other words, the control is transferred 
**non-deterministically** amongst the suspendable functions.

Finally, uou could reasonably argue that the pseudocode in the
[improved implementation](/suspendables/control/#improved-implementation) actually uses explicit 
control transfer. This is correct! Implicit control transfer mechanism is indeed built upon
the explicit control transfer mechanism with the event loop being the invisble control tranfer 
coordinator.

## Explicit _v._ Implicit Control Transfer
Some asynchronous tasks, such as the
[Salad & Mashed Potatoes](/suspendables/control/#explicit-control-transfer-suspendables)
example, do not benefit for an event loop[^3]. There is no need to force such functions to
run within an event loop.

Some asynchronous tasks, such as the [Plane Ticket](/suspendables/buying-a-plane-ticket/) 
example, almost always require an event loop to run efficiently. It is inefficient to not 
run these functions within an event loop[^4].

### One unified syntax for both explicit & implicit control tranfer?
While there is a natural distinction between the type of suspendable functions that require
an event loop and those who don't, there appears to be no reason why the syntax for explicit
and implicit control transfer suspendables should be different. It is somewhat reasonable to
expect that a programming language provide only one type of suspendable function with the same
set of reserved keywords for both explicit and implicit control transfer. Perhaps, a
suspendable function's control transfer can be made explicit or implicit automatically 
depending on whether or not it interacts with an event loop?

This happens to _not_ be the case with python, as of writing this course. The following table
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

In python, explicit control transfer suspendables are implemented as generators and 
implicit control transfer suspendables are implemented as coroutines[^5]. 

The reason for this bifurcation is complicated involving syntactical, historical,
and design considerations. Just after [PEP 342](https://www.python.org/dev/peps/pep-0342/),
the same `yield from` syntax was used for both (explicit) generators and (implicit) coroutines.
But, using the same syntax for both generators and coroutines created
[problems](https://github.com/python/cpython/issues/68588) and confusion, as mentioned in
[PEP 492](https://www.python.org/dev/peps/pep-0492/#rationale-and-goals). PEP 492 finally
separated coroutines from generators. We will discuss some of these syntactical, historical,
and design considerations later in the course.

One weird artifact of this bifurcation is that python may allow a suspendable to be
both explicit and implicit at the same time. This is what python calls
_asynchronous generators_, as described in this 
[bug report](https://github.com/python/cpython/issues/71430) and discussed in
[PEP 492](https://www.python.org/dev/peps/pep-0492/#why-aiter-does-not-return-an-awaitable).
We will discuss this illumintaing oddity later in the course.

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

[^3]:
    A memory-efficient, boundless iterator (such as Python 3's `range`) is a common real-world
    example of a suspendable function that does not benefit from an event loop.

[^4]:
    Unless it's for [educational purposes](https://stackoverflow.com/questions/52783605/how-to-run-a-coroutine-outside-of-an-event-loop).

[^5]:
    We are purposefully ignoring the existence of generator-based coroutines. See Footnote
    [2](/suspendables/control/#fn:2) on this page.


# `yield from`
We saw previously that presence or absence of a syntactically reachable `yield` within a function
can change the nature of the function completely. Adding a `yield` makes the function into a
generator function and removing the `yield` makes the function a simple function.
We also discussed that this `yield`-induced structural change can cause the entire downstream code
to be rewritten. This `yield`-induced structural change is a big problem when refactoring code.

Let's look at a toy[^real-world-example] example to understand this issue. The following example
generates random shades of red and random shades of blue as `(r, g, b)` tuples.

```python
import random

def generate_red_blue(n=2):
    while True:
        # Generate n shades of red
        for _ in range(n):
            r = random.randint(1, 255)
            gb = random.randint(0, r - 1)
            yield (r, gb, gb)

        # Generate n shades of blue
        for _ in range(n):
            b = random.randint(1, 255)
            rg = random.randint(0, b - 1)
            yield (rg, rg, b)

x = generate_red_blue()

next(x)
# (105, 75, 75)  # red

next(x)
# (205, 125, 125)  # red

next(x)
# (108, 108, 225)  # blue

next(x)
# (143, 143, 206)  # blue
```
You can execute the generator function `generate_red_blue` yourself to see how it works.
Consider the situation that you need another generator function to generate shades of red and
green. You could write another function, say, `generate_red_green` by copying the
red section of `generate_red_blue`. This would generally be a
[bad programming practice](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) for various
reasons. For example, if we wanted to replace the current red scheme with a better scheme,
we will have to edit two functions.

Expectedly, we should extract out the red section into a generator of its own. This does not
pose a problem, yet. Now, if we wanted yet another function to generate shades of `green` and
`blue`, we would need to extract the blue section into another generator function of its own.
Extracting out both red and blue sections would remove the two `yield` statements from
`generate_red_blue`, which would reduce it to a simple function. This would require
a rewrite of the downstream code, as we've discussed
[before](/generators/introduction/#brevity-has-its-costs).


## Naive Solution
We can solve the problem by instantiating the generator objects within `generate_red_blue`
and then yield ourselves, as shown below.

```python
import random

def generate_red(n=2):
    for _ in range(n):
        r = random.randint(1, 255)
        gb = random.randint(0, r - 1)
        yield (r, gb, gb)

def generate_blue(n=2):
    for _ in range(n):
        b = random.randint(1, 255)
        rg = random.randint(0, b - 1)
        yield (rg, rg, b)

def generate_red_blue_refactored_basic(n=2):
    while True:
        for red in generate_red(n):
            yield red
        for blue in generate_blue(n):
            yield blue

y = generate_red_blue_refactored_basic()

next(y)
# (159, 8, 8)  # red

next(y)
# (64, 40, 40)  # red

next(y)
# (54, 54, 157)  # blue

next(y)
# (0, 0, 198)  # blue
```
This works, as you can check yourself. But, the code above has a problem. For example, if we
send a value to `generate_red_blue_refactored_basic`, it would not automatically get sent to
`generate_red` or `generate_blue`; we will have to update `generate_red_blue_refactored_basic`
to send any value it receives to the subgenerators `generate_red` and `generate_blue`.
Another problem is the use of other generator functions, `throw` and `close`.


## Better Solution
Handling all of these cases is [non-trivial](https://www.python.org/dev/peps/pep-0380/#motivation)
yet boilerplate. This is why [PEP 380](https://www.python.org/dev/peps/pep-0380) added a new
syntax called `yield from`.

The `generate_red` and `generate_blue` are defined just like in the naive solution above.
The refactored, `yield from`-based generator function is below.

```python
def generate_red_blue_refactored(n=2):
    while True:
        yield from generate_red(n)
        yield from generate_blue(n)

z = generate_red_blue_refactored()

next(z)
# (60, 58, 58)  # red

next(z)
# (64, 43, 43)  # red

next(z)
#  (155, 155, 200)  # blue

next(z)
#  (56, 56, 59)  # blue
```

On its face, `yield from` seems to provide a marginal improvement in brevity. The generator
function `generate_red_blue_refactored_basic` was quite terse to begin with and
`generate_red_blue_refactored` only removes the simple `for` loops.

However, by using `yield from`, `generate_red_blue_refactored` allows sending values back to
the subgenerators and a proper handling of exceptions. If we were to add these benefits to
our basic solution in `generate_red_blue_refactored_basic`, we would have to write a lot of
code such as the one shown in [PEP 380](https://www.python.org/dev/peps/pep-0380/#formal-semantics).

!!! success "`yield from` still makes a generator function"
    As you may have noticed, a function containing a syntactically reachable `yield from` is
    still a generator function (which is an extended, not a simple function). The same
    [rules](/generators/mechanics-by-examples/) apply as that for `yield`.

## `yield from` is a QoL improvement
If you review [PEP 380](https://www.python.org/dev/peps/pep-0380/#formal-semantics), you'll find
that `yield from` is semantically equivalent to a small snippet of regular python code that
contains a lot of `try`-`catch` statements. Thus, `yield from` is simply a
*[quality of life](https://www.destructoid.com/stories/the-new-borderlands-3-patch-ushers-in-some-welcome-quality-of-life-changes-598543.phtml)*
improvement with only new python magic being the new keyword `yield from`.

## `yield from` to write generator-based coroutines
It is true that `yield from` has a special usage: it can be used to write generator-based
coroutines. We don't talk about generator-based coroutines in this course for many reasons.
Firstly, generator-based coroutines are deprecated and scheduled for
[removal](https://docs.python.org/3.10/library/asyncio-task.html#generator-based-coroutines)
in Python 3.10. It's no longer need to learn asynchronous programming in python.
Secondly, even the term *generator-based coroutines* causes so much
[confusion](https://www.python.org/dev/peps/pep-0492/#rationale-and-goals); is it a generator,
is it a coroutine, is it both, or is it neither? See the
[footnote in Control](/suspendables/control/#fn:5) and the
[footnote in Introduction](/why-is-it-difficult/#fn:5) for even more details.

We recommend beginners to forget that generator-based coroutines exist or that `yield from`
has a special usage to create them. There is nothing to lose by doing so.


## Footnotes
[^real-world-example]:
    This example is quite silly. Some would argue that the `(r, g, b)` tuples being returned
    are not even shades of red/blue and they would be right. Such a scheme to determine
    shades of red/blue is definitely not something to be used in real world code.
    For example,
    <a style="color:rgb(72, 218, 223)" href="https://www.google.com/search?q=rgb++(72%2C+218%2C+223)">
    `(72, 218, 223)`</a>
    can be considered <span style="color:rgb(72, 218, 223)">blue</span> but it won't be returned by
    our example.
    If you want a more real world example demonstrating the benefits of `yield from`, consider the
    [Fibonacci server](https://speakerdeck.com/dabeaz/concurrency-from-the-ground-up-live?slide=44)
    that David Beazley coded [live](https://youtu.be/MCs5OvhV9S4?t=2305).

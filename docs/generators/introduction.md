# Introduction
## Definition
A generator may be defined simply by including a syntactically reachable `yield` (or `yield from`)
keyword within a function. The following is an example of the simplest generator.

```python
def example_generator_function():
    yield 1
```
Notice how we used the word *function* to describe the above example. This is because
`example_generator_function` is indeed a function. You can call it by executing
`example_generator_function()` like any other python function, as shown below.

```python
example_generator_function()
```

However, `example_generator_function` is not a simple function. This means that calling
it will not execute the contents of the function. Feel free to check this out yourself.

```python
x = example_generator_function()
print(x)
# <generator object example_generator_function at 0x10be77740>
```
The result of calling `example_generator_function` is stored in `x` and it is not `1`,
or `None`, or anything you would've expected had it been a simple function.
Instead, calling `example_generator_function` returns a generator object.
`example_generator_function` is a suspendable and therefore an extended function.
This means that we require some other mechanism to execute the contents of
`example_generator_function`. One such mechanism is to call `next` on the generator object `x`.

```python
next(x)
# 1
```

Later on, we will look at `next` [in detail](/generators/a-better-way-to-drive/) and
discuss other ways of executing the contents of a generator function.

## Confusing nomenclature
The word *generator* can be ambiguous.

*Does it refer to `example_generator_function` or `x` ?*

Ideally, a function like `example_generator_function` would always be called
a *generator function*, an object like `x` would always be called a *generator object*,
and the word *generator* would always be used as an adjective instead of a noun. Sadly,
such carefulness in word choice is uncommon and the word *generator* is often used to refer
to both a generator function and a generator object.

## `yield` is like `return`
`yield` may be thought of as a free-spirited cousin of `return`.
Both `yield` and `return` optionally return a value back to the caller of the function they are
in. `yield` suspends the execution of the rest of the function leaving the possibility open
for a resumption of execution. In contrast, `return` ends the
execution of the rest of the function without any chance of resumption. This is best
demonstrated by the following pair of functions.

=== "Simple Function"
    ```python
    def print_return_and_print():
        print("Starting execution")
        return 1
        print("Ending execution")

    y = print_return_and_print()
    # Starting execution

    print(y)
    # 1

    next(y)
    # ---------------------------------------------------------------------------
    # TypeError                                 Traceback (most recent call last)
    # <ipython-input-42-cf9ac561a401> in <module>
    # ----> 1 next(y)
    #
    # TypeError: 'int' object is not an iterator
    ```

=== "Generator (Extended Function)"
    ```python
    def print_yield_and_print():
        print("Starting execution")
        yield 1
        print("Ending execution")

    z = print_yield_and_print()
    print(z)
    # <generator object print_yield_and_print at 0x10bfb0510>

    z_value = next(z)
    # Starting execution

    print(z_value)
    # 1

    next(z)
    # Ending execution
    # ---------------------------------------------------------------------------
    # StopIteration                             Traceback (most recent call last)
    # <ipython-input-38-81b9d2f0f16a> in <module>
    # ----> 1 next(z)
    #
    # StopIteration:
    ```

While the functions `print_return_and_print` and `print_yield_and_print` look very similar,
they are structurally very different.
`print_return_and_print` is a simple function and `print_yield_and_print`
is an extended function. When we call `print_return_and_print`, we see that `Starting execution`
is printed out and the value `1` is stored in `y`. The string `Ending execution` is never printed
out with no recourse besides changing the function itself[^1].
Finally, calling `next` on `y` gives us a `TypeError` error because `y` is just `1` and
`next` cannot be called on `1`.

In contrast, when we call `print_yield_and_print`, nothing gets printed at all and the value `1`
is *not* stored in `z`. Instead, `z` is a generator object. When we call `next` on `z` ,
the contents of `print_yield_and_print` are executed until the yield (or suspension) point,
`Starting execution` is printed, and the value `1` is stored in `z_value`. At this point,
the execution of the contents of `print_yield_and_print` is suspended and the control is
transferred back to the caller. The execution may be resumed by calling `next` again
on the *same* generator object `z` from immediately before. Calling a second `next` resumes
the execution from where it was suspended[^2], prints out `Ending execution`, and then
throws a `StopIteration` error, which, as we will discuss later, is expected and
does not mean that there is a mistake in our code.

## Brevity has its costs
Unprimed readers may underestimate how much of a structural difference a tiny change
like replacing `return` with `yield` can create.

Imagine that you've written a complicated generator function and accompanying downstream code that
executes the contents of the said generator function.
One day, you decide to refactor the generator function and comment out all the `yield` statements,
which reduces the extended function to a simple function.
As a result of this edit, the downstream code that was previously able to *drive* the generator
now throws an error, similar to the `TypeError` we got when we called `next` on `y`. All downstream
code that interacts with this function now needs to be rewritten.
On some other day, if you decide to revisit the function and uncomment any of the `yield`
statements, you will need to update the downstream code again, this time in reverse.

!!! Summary
    Even though they look alike, a simple function and a generator function are
    not interchangeable.

As a side note, newer languages may have a less cumbersome design. One example is a goroutine in
Go. Go allows you to [re-use](https://gobyexample.com/goroutines) the same function in two ways,
once, as a simple function that you call directly and execute, and second, asynchronously
in a goroutine. You don't need to write two variants of the same idea. You can write once and
decide whether you want to call it synchronously or asynchronously later.

## `yield` is a two way street
Perhaps, as an act of rebellion, `yield` can accept a value from the caller into the
generator function. This is best experienced in action.

```python
def receive_value():
    value = yield 1
    print('received value = {}'.format(value))
    yield 2

gen = receive_value()
print(gen)
# <generator object receive_value at 0x11106e6d0>

first = next(gen)
print(first)
# 1

second = gen.send('hello')
# received value = hello
print(second)
# 2
```

This time, our generator function `receive_value` has two `yield` expressions and the first
`yield` expression expects to receive a value. As usual, we can start to drive the generator
by calling `next` on the generator object `gen`. Once the execution is suspended
after the first `next` call, we call the `send` method of the generator object. This allows
us to send any value, such as `'hello'` in our case, back to the generator function. The
`send` method runs from the first `yield` expression until the second `yield` statement, which
is why `send` returns the value `2`, which is then stored in `second`.

## Behold the power
A generator is a very powerful construct because it is able to both return and receive values,
multiple times, without losing its state. A generator is bestowed with all the benefits of a
function, which means that a generator function can accept function arguments, maintain its own
independent scope, and contain arbitrarily complicated code. For example, we could return
multiple values from a generator function at different times. Or, we could choose which branch
of an `if-else` statement gets executed inside a live, running generator based on the value
it receives. Such tremendous freedom is not granted to a mere simple function. In some ways,
as we will see next, a generator is a somewhat complicated class unto itself.

Many readers may have more questions about the mechanics of `yield` such as how many `yield`
statements can we have, can a function have both `yield` and `return` statements, and
what is meant by syntactically reachable. We discuss such questions in the
[Mechanics by Examples](/generators/mechanics-by-examples/) section after we have studied the
underlying design of generators.

## Footnotes
[^1]:
    Calling `print_return_and_print` a second time would repeat the same behavior and will still
    not print out `Ending execution`.

[^2]:
    There are other ways besides `next` to resume a suspended execution.

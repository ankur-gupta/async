# Introduction
Let's forget that we ever learned generators. Let's continue 
from the [review](/suspendables/review/#suspendables-by-control-transfer) of suspendables.

|      Suspendable Type     | Python Implementation | Definition   | Control Transfer Point Keywords |
|:--------------------------|:----------------------|--------------|---------------------------------|
| Explicit Control Transfer | Generators            | `def`        | `yield`, `yield from`           |
| Implicit Control Transfer | Coroutines            | `async def`  | `await`                         |

Coroutines are completely independent from generators[^1]. We need not even know about generators 
to learn about coroutines. Generators require the use of `yield` and `yield from` keywords. 
Coroutines require the use of `async def` and optionally the use of `await` keyword.

Oddly enough, you can still use `yield` within a coroutine. This will create an object called 
_asynchronous generator_, which is both a generator and a coroutine hybrid generator. We will 
study the _asynchronous generator_ in the advanced section (LINK ME)
But, using  `yield from` within a coroutine is illegal (CITE THE PEP).

## Definition 
A coroutine may be defined simply by adding a `async` before `def`. The following is an example
of the simplest coroutine. The use of `await` keyword is 
[not needed](https://peps.python.org/pep-0492/#new-coroutine-declaration-syntax) 
to define a coroutine. However, it is a `SyntaxError` to use `await` outside of an 
`async def` function.

```python
async def example_coroutine_function():
    return 1

type(example_coroutine_function)
# function
```

The coroutine function `example_coroutine_function` is a suspendable and an extended function, 
analogous to a generator function. Calling `example_coroutine_function` does not execute 
the contents of the function. Instead, it returns a coroutine object.
We need some other mechanism to execute the contents of `example_coroutine_function`. 

```python
coro = example_coroutine_function()
print(coro)
# <coroutine object example_coroutine_function at 0x111b625c0>

type(coro)
# coroutine
```

### Coroutine does not implement the `Iterator` protocol
For [generators](/generators/a-better-way-to-drive/#__next__-is-a-part-of-iterator-protocol), 
we could use `next`, which is part of the 
[`Iterator` protocol](https://docs.python.org/3/glossary.html#term-iterator). 
Coroutine does not implement the `Iterator` protocol. Compare the following code 
with the corresponding code for a 
[generator](/generators/a-better-way-to-drive/#generator-implements-iterator-protocol).

```python
from collections.abc import Iterator

isinstance(coro, Iterator)
# False

coro.__iter__
# AttributeError: 'coroutine' object has no attribute '__iter__'

coro.__next__
# AttributeError: 'coroutine' object has no attribute '__next__'

next(coro)
# TypeError: 'coroutine' object is not an iterator
```

### Automatic drive
We cannot use `next` to drive coroutine objects. The coroutine equivalent to `next` 
is bit more complicated because imnplicit control transfer suspendables need an event loop.
The event loop invisibly and implicitly transfers the control between coroutines.
We have many choices for an event loop but we will use `asyncio`, which is the default 
event loop that comes with the python standard library.

```python
import asyncio
output = asyncio.run(example_coroutine_function())
print(output)
# 1
```

### Manual drive
We previously saw that for generators, `next(generator_object)` was equivalent to
`generator_object.send(None)`. For coroutines, `next` doesn't work but `send` does.
`send` method has the same semantics as with 
[generators](/generators/mechanics-by-examples/#drive-using-send):

- `send` requires one argument
- coroutine needs to be primed before we can send a non-`None` value
- `send(None)` works but the value is received by `await` (we'll study `await` [later](/coroutines/introduction/#await-is-like-yield))

```python
example_coroutine_function().send()
# TypeError: coroutine.send() takes exactly one argument (0 given)

example_coroutine_function().send('something')
# TypeError: can't send non-None value to a just-started coroutine

# There was no `await` in the coroutine function
example_coroutine_function().send(None)  
# StopIteration: 1
```

### Custom drive
Using a proper event loop such as the one provided by `asyncio` is the intended way to drive 
a coroutine. However, we could 
[write our own little event loop](https://stackoverflow.com/questions/52783605/how-to-run-a-coroutine-outside-of-an-event-loop) 
to drive the coroutine ourselves. 

```python
def drive(coroutine_object):
    while True:
        try:
            coroutine_object.send(None)
        except StopIteration as e:
            return e.value

output = drive(example_coroutine_function())
print(output)
# 1
```
There are two important things to note here:

- We used a `while` loop to make our event loop, just like we did in our pseudocode for the
[improved implementation](/suspendables/control/#improved-implementation) of the plane 
ticket example.
- The `.send` method and `StopIteration` work for a coroutine object, just like 
they did for the [generator object](/generators/mechanics-by-examples/#drive-using-send).

Finally, our event loop is very rudimentary and will not work even with slightly more 
complicated toy examples.

## Confusing nomenclature
Like the word _generator_, the word _coroutine_ is ambiguous.

???+ question "Question"
    Does _coroutine_ refer to `example_coroutine_function` or `example_coroutine_function()`?

The answer is the same as that for [generators](/generators/introduction/#confusing-nomenclature). 

| Object                          | Type Name             |
|:--------------------------------|:----------------------|
| `example_coroutine_function`    | Coroutine function    |
| `example_coroutine_function()`  | Coroutine object      |

It's best to use the word _coroutine_ as an adjective instead of a noun.

## `await` is like `yield`
Like `yield`, `await` is a control transfer point. You can suspend control at an `await`. 
`await` can also accept values. There is one difference — `yield` allowed us to yield 
any arbitrary value including 
[nothing](/generators/mechanics-by-examples/#must-we-return-something-with-yield) 
but `await` only allows us to await an `Awaitable` object.

=== "Fails"

    ```python
    async def print_await_print():
        print('Starting execution')
        await 1  # Can't await an int because it's not an Awaitable
        print('Ending execution')

    asyncio.run(print_await_print())
    # ...
    # TypeError: object int can't be used in 'await' expression
    ```

=== "Also Fails"

    ```python
    async def print_await_print():
        print('Starting execution')
        await  # await needs something to await
        print('Ending execution')
    # SyntaxError: invalid syntax

    ```

=== "Works"

    ```python
    import asyncio

    async def print_sleep_print():
        print('Starting execution')
        await asyncio.sleep(1)  # This is a coroutine (which is an Awaitable)
        print('Ending execution')

    asyncio.run(print_sleep_print())
    # Starting execution
    # Ending execution
    ```

???+ question "Question"
    What is an `Awaitable` object?

Quite simply, an `Awaitable` object is an object with an 
[`__await__` method](https://docs.python.org/3/library/collections.abc.html#collections.abc.Awaitable)[^2].
A coroutine is also an `Awaitable` object as you can see from the 
[definition](https://github.com/python/cpython/blob/d5d3249e8a37936d32266fa06ac20017307a1f70/Lib/_collections_abc.py#L114)
of a `Coroutine`.

## `await` is also a two-way street
=== "Fails"

    ```python
    import asyncio

    async def await_sleep():
        print('Starting execution')
        value1 = await asyncio.sleep(1, result=2)
        print(value1)
        value2 = await asyncio.sleep(1, result=3)
        print(value2)
        print('Ending execution')
    
    # This will fail because `asyncio.sleep(1)` needs an event loop!
    await_sleep().send(None)
    # RuntimeError: no running event loop
    ```

=== "Works"

```python
async def no_await(x):
    return x * x

async def await_no_await():
    print('Starting execution')
    value1 = await no_await(2)
    print(value1)
    value2 = await no_await(3)
    print(value2)
    print('Ending execution')

await_no_await().send(None)

```



From https://github.com/python/cpython/blob/d5d3249e8a37936d32266fa06ac20017307a1f70/Lib/_collections_abc.py#L57:
```python
## coroutine ##
async def _coro(): pass
_coro = _coro()
coroutine = type(_coro)
```



<!--

differences from generators - not just explicit vs implicit
https://www.python.org/dev/peps/pep-0492/#differences-from-generators
coroutines do not have `__iter__` or `__next__` methods, cannot be iterated over, and therefore
cannot be used in a for loop.
Coroutines are based on generators internally, thus they share the implementation. Similarly to generator objects, coroutines have throw(), send() and close() methods.


https://www.python.org/dev/peps/pep-3156/#coroutines
The object obtained by calling a coroutine function. ... If disambiguation is needed we will call it a coroutine object. -->



<!-- async generator:
"Using a yield expression in a function’s body causes that function to be a generator, and using it in an async def function’s body causes that coroutine function to be an asynchronous generator."
from: https://docs.python.org/3/reference/expressions.html#yield-expressions

Also from https://docs.python.org/3/reference/expressions.html#yield-expressions which contains the word suspended:
"All of this makes generator functions quite similar to coroutines; they yield multiple times, they have more than one entry point and their execution can be **suspended**."


https://docs.python.org/3/whatsnew/3.6.html?highlight=yield#pep-525-asynchronous-generators
"A notable limitation of the Python 3.5 implementation is that it was not possible to use await and yield in the same function body. In Python 3.6 this restriction has been lifted, making it possible to define asynchronous generators:" -->


<!-- Deals with async generators: important https://peps.python.org/pep-0492/#why-aiter-does-not-return-an-awaitable -->


## Footnotes
[^1]:
    We are purposefully ignoring the existence of generator-based coroutines which 
    are [defunct](https://docs.python.org/3.10/library/asyncio-task.html#generator-based-coroutines)
    as of Python 3.10. There is no benefit to learning generator-based coroutines except for 
    unnecessary confusion.

[^2]:
    Yet again, we're ignoring the existence of 
    [defunct](https://docs.python.org/3.10/library/asyncio-task.html#generator-based-coroutines)
    generator-based coroutines. This is a perfect example of how generator-based coroutines
    cause unnecessary confusion. **Feel free to skip the rest of this footnote to avoid confusion**
    or **continue at your own risk**. According to the 
    [official documentation](https://docs.python.org/3/library/collections.abc.html#collections.abc.Awaitable)
    on `Awaitable`, the defunct generator-based coroutines are considered awaitables even though 
    they don't have an `__await__` method. As a result,
    `isinstance(gencoro, collections.abc.Awaitable)` will return `False`; use 
    `inspect.isawaitable()` to properly detect generator-based coroutines as awaitables.
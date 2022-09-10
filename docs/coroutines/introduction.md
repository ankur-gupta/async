# Introduction
FIXME

Let's assume that we do not know anything about generators. We will not need to use the
keywords `yield` or `yield from` in coroutines at all. In fact, adding either of these may create
problems. Using `yield` within a coroutine may create a beast called asynchronous generator
which we will study later on. Using `yield from` is illegal (CITE THE PEP).

Let's begin from the end of
Suspendables, specifically, this
[table](/suspendables/control/#explicit-vs-implicit-control-transfer-suspendables)
and the [review](/suspendables/review/#python-from-here-on).

A coroutine may be defined simply by adding a `async` before `def`. The following is an example
of the simplest coroutine.

```python
async def example_coroutine_function():
    return 1
```

Can we drive the coroutine ourselves?

```python
import asyncio
asyncio.run(example_coroutine_function())
# 1
```

https://stackoverflow.com/questions/52783605/how-to-run-a-coroutine-outside-of-an-event-loop

Coroutine is derived from an Awaitable
https://github.com/python/cpython/blob/d5d3249e8a37936d32266fa06ac20017307a1f70/Lib/_collections_abc.py#L114
https://docs.python.org/3/library/collections.abc.html#collections-abstract-base-classes

Coroutine does not have a `__next__` method.

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

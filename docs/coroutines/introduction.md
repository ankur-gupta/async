# Coroutines




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

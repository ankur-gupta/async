# Coroutines




<!--

differences from generators - not just explicit vs implicit
https://www.python.org/dev/peps/pep-0492/#differences-from-generators
coroutines do not have `__iter__` or `__next__` methods, cannot be iterated over, and therefore
cannot be used in a for loop.
Coroutines are based on generators internally, thus they share the implementation. Similarly to generator objects, coroutines have throw(), send() and close() methods.


https://www.python.org/dev/peps/pep-3156/#coroutines
The object obtained by calling a coroutine function. ... If disambiguation is needed we will call it a coroutine object. -->

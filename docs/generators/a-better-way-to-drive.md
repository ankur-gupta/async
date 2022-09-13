# A better way to drive
Calling `next` repeatedly is tedious. There is a better way to drive a generator. But, first,
let's look at `next` in detail.

## `next` is not *that* special
It may appear that `next` is another special keyword like `yield` but it is not. As a quick test,
try running this code yourself.

```python
# Remember to exit and restart the terminal after you redefine `next`
# in order to avoid erroring out later examples.

next = 1
print(next)
# 1

yield = 2
#  File "<ipython-input-3-837215eb2b53>", line 1
#    yield = 2
#          ^
# SyntaxError: invalid syntax
```
`next` in not a keyword but a built-in function that is made available to you automatically
when you start a python session. `next` is actually like `len`. Just like calling `len` on
an object `x` simply calls `x.__len__()`, similarly, `next(x)` is nothing more than calling
`x.__next__()`. This means that `next` is not specific to generators. You can call `next` on
any class that has a `__next__` method. Try the following code out yourself.

```python
class NextDemo:
    def __next__(self):
        return 'You called next!'

x = NextDemo()
next(x)
# 'You called next!'

x.__next__()
# 'You called next!'
```

## `__next__` is a part of Iterator protocol
[Iterator protocol](https://docs.python.org/3/glossary.html#term-iterator)
requires the `__next__` method to be [implemented](https://docs.python.org/3/library/collections.abc.html#collections-abstract-base-classes), along with the `__iter__` method.

You can define an `Iterator` independently of any generator or asynchronous programming concepts
by simply defining the `__next__` and `__iter__` methods in a class. The following example
shows such a class that can produce arbitrarily many squares of sequential natural numbers, on
demand. All without using any generators or special keywords.

```python
class Squares:
    i = 1

    def __iter__(self):
        return self

    def __next__(self):
        out = self.i * self.i
        self.i = self.i + 1
        return out

for square in Squares():
    if square > 100:
        break
    print(square)
# 1
# 4
# 9
# 16
# 25
# 36
# 49
# 64
# 81
# 100
```
The benefit of an Iterator is that you can use it in a `for` loop without having to call `next`
yourself, making for a terse syntax.

Note that, in real-world code, you may want to inherit from `collections.abc.Iterator`. Also,
while you could call `next` on `NextDemo`, you cannot run it in a `for` loop because it
does not implement `__iter__`.


## Generator implements Iterator protocol
Generator implements the Iterator protocol as shown in the
[source](https://github.com/python/cpython/blob/23a567c11ca36eedde0e119443c85cc16075deaf/Lib/_collections_abc.py#L322).

Generators are
[based](https://docs.python.org/3/library/collections.abc.html#collections.abc.Iterator)
on iterators but iterators are not based on generators. So, a generator is an iterator but an
iterator may not necessarily be a generator.

```python
from collections.abc import Iterator

def example_generator_function():
    yield 1

gen = example_generator_function()

isinstance(gen, Iterator)
# True

gen.__iter__
# <method-wrapper '__iter__' of generator object at 0x105e58190>

gen.__next__
# <method-wrapper '__next__' of generator object at 0x105e58190>

next(gen)
# 1
```

Since generator is an iterator, we can drive a generator using a `for` loop instead of calling
`next` ourselves. The following simple implementation of a range function demonstrates this.

```python
def my_simple_range(start: int, stop:int):
    i = start
    while i < stop:
        yield i
        i = i + 1

for i in my_simple_range(0, 3):
    print(i)
# 0
# 1
# 2
```

You can use a generator in list comprehensions exactly like you would use a materialised `list` or
`tuple`.

```python
evens = [i for i in my_simple_range(0, 6) if i % 2 == 0]
print(evens)
# [0, 2, 4]
```

You could also drive a generator to exhaustion/completion (if it is exhaustible) by simply
calling `list` on it.

```python
three = list(my_simple_range(0, 3))
print(three)
# [0, 1, 2]
```

# Examples
We discuss some more real-world examples here. Examples may be added or removed from this
section later.

## range
A generator is the perfect tool to write a `range` function. `range` is often used only to
index or iterate over a list-like object. Even though `range` is in-built, we will rewrite
a simpler version of `range` ourselves to see how generators are perfectly suited for this task.

### Naive Approach
Let's first consider a naive, non-generator, simple function.

```python
def naive_range(start, stop):
    out = []
    i = start
    while i < stop:
        out.append(i)
        i = i + 1
    return out

print(naive_range(0, 10))
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# This may take a while and may run out of memory
naive_range(0, 10000000)

for i in naive_range(0, 10000000):
    # use i to index into another list-like object
```

The simple function `naive_range` materializes the entire range into a list and returns it.
This takes a lot of memory. Imagine you want to iterate over a 10M long list-like object.
Using `naive_range` to iterate over it would first create another 10M long list in memory.
This is a waste of memory because you only want one value of the iteration index `i` in memory
at a time.

### Generator approach
We [already saw](/generators/a-better-way-to-drive/#generator-implements-iterator-protocol)
a generator-based range-like function but let's see it again.
```python
def generator_range(start, stop):
    i = start
    while i < stop:
        yield i
        i = i + 1

print(list(generator_range(0, 10)))
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# This does not take a lot of memory and
# returns a generator object immediately.
generator_range(0, 10000000)

for i in generator_range(0, 10000000):
    # use i to index into another list-like object
```
You should try it out yourself to see the difference. Since generators follow the iterator
protocol, we don't need any special syntax when using `generator_range` in a `for` loop.

!!! tip "`range` in Python 2 and 3"
    The `range` function in Python 2 used lists much like our `naive_range` while the `range`
    function in Python 3 uses iterators much like our `generator_range`.
    See [this](https://stackoverflow.com/questions/44571718/python-3-range-vs-python-2-range).

### Unbounded `range`
Generator-based approach allows us to generate integers unboundedly. interestingly, this is
even simpler than our `generator_range` above. This was clearly not possible if we use a
naive list-based approach.

```python
def unbounded_range(start):
    i = start
    while True:
        yield i
        i = i + 1

r = unbounded_range(0)
next(r)
# 0

next(r)
# 1

next(r)
# 2

# Use in a for loop with care to avoid
# an infinitely running loop.
for i in unbounded_range(0):
    # do something with i
    print(i)
    if i > 10:
        break
# 0
# ...
# 11
```

## Random number generator
Another surprising perfect fit for generators is a random number generator.
Random number generators are typically state machines.

```python
def discrete_uniform_lgc(seed=894965, modulus=2**32,
                         coeff=1664525, constant=1013904223):
    x = seed
    while True:
        x = (coeff * x + constant) % modulus
        yield x
```

FILL ME

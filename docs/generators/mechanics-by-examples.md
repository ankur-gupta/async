# Mechanics by Examples
Curious readers may have lots of questions on how exactly `yield` behaves. Let's answer them with
examples.

## `yield` under an impossible branch
In [Introduction](/generators/introduction/), we said that a syntactically reachable `yield`
within a function makes it a generator function. The important word is *syntactically*. So,
even if a `yield` statement is inside a never-to-be-executed `if` branch, such as shown below,
the function would still be a generator function.

```python
def still_a_generator_function():
    print('Entering')
    if False:
        yield 1
    print('Exiting')

x = still_a_generator_function()
print(type(x))
# <class 'generator'>

next(x)
# Entering
# Exiting
# ---------------------------------------------------------------------------
# StopIteration                             Traceback (most recent call last)
# <ipython-input-4-92de4e9f6b1e> in <module>
# ----> 1 next(x)

# StopIteration:
```
The above generator function is indeed quite strange. The `yield` statment will never get to
execute because it is under an impossible `if` branch, and yet, presence of the `yield` keyword
makes for a generator function. Then, when, we call `next` on the generator function, it does not
find even a single `yield` statement (where the execution could've been suspended), reaches
the end of the function, and throws a `StopIteration`[^pick-a-side-yield].

Allowing an impossible to reach `yield` to create a generator is still a reasonable decision.
This is because in a more real-world example, the python interpreter can never know which way
an `if` statement would resolve. Consider the following snippet.

```python
def real_world_function():
    # ...
    if some_variable.some_method():
        yield something
    else:
        # Do anything but yield
```
There is no way of knowing whether `some_variable.some_method()` would be considered equivalent
to `True` or not. What if `some_variable.some_method()` uses pseudo random numbers and can return
a `True`-equivalent[^true-equivalent] during some runs and a `False`-equivalent during some other
runs? Even if `some_variable.some_method()` does not use random numbers, code changes in this
method would affect which branch of `if-else` is executed inside `real_world_function`.

Programmers would also face the same exact problem as the interpreter. How would you write the
downstream code to execute the contents of `real_world_function` ? We've already
[seen](/generators/introduction/#yield-is-like-return) that we require very different code
to drive a generator function versus running a simple function.

The same applies to other control flow constructs such as `while` and `do while` or even something
as evil as this.

```python
def evil_generator_function():
    while True:
        pass
    yield 1
```
This behavior can also be used to your advantage when you're editing code. Suppose that you have
a generator function called `frequently_edited_gen_fun`. You've written downstream code to
use `frequently_edited_gen_fun` as a generator function. Now, you need to edit it and as a result
of this edit, you have to remove all the `yield` statements. Removing all `yield` statements
would reduce `frequently_edited_gen_fun` to a simple function causing all the downstream code
to fail unless rewritten. You can salvage the situation by adding an impossible to reach `yield`
like the one shown in one of the examples above. This yield would never be reached but it will
still prevent `frequently_edited_gen_fun` from becoming a simple function.

MENTION THAT REJECTED PIP HERE (COFUNCTIONS, WAS IT?).

See yet another example in [`return` before a `yield`](#return-before-a-yield).


## Must we return something with `yield`?
No. We can yield without specifying a yield value.

```python
def no_yield_value():
    print('Entering')
    yield
    print('Exiting')

y = no_yield_value()
y_yield_value = next(y)
# Entering

print(y_yield_value)
# None
```
If we don't specify a yield value, `None` is yielded. This is similar to `return` in the following
simple function.

```python
def no_return_value():
    return

no_return_value() is None
# True
```

## `yield` is an expression
Since [PEP 342](https://www.python.org/dev/peps/pep-0342/#specification-summary), `yield` is
an expression rather than a statement, even if it can be used in a statement form.

```python
def yield_is_an_expression():
    product_value = (yield 1) * (yield 2)  # expression form
    yield 3  # statement form
```
The above code is legal syntax. You should try this out yourself because things will become
murky in a moment. We've [already](/generators/introduction/#yield-is-a-two-way-street) seen
that we can send value into a a generator via the `yield` keyword. The above generator function
seems to compute `product_value` as a sum of two `yield` expressions. Only difference is
that this time the expressions are wrapped in parentheses. This is also important. Thw following
statements list the rules of engagement when it comes to `yield`.

1. `yield` is always an expression, even if it can be written in a statement form
   (like a `return` statement)
2. `yield` expressions almost always require parentheses with two known exceptions. Overuse of
   parentheses is encouraged.
3. Every `yield` expression has a value, even if the value is `None`.

Let's look at an example that is easier to run than the one above. We'll discuss why this
example is easier than the one above in a moment.

```python
def fill_list_using_yields():
    list_value = [(yield), (yield)]
    print('list_value={}'.format(str(list_value)))
    yield
```

### Drive using `next`
We can drive the above generator in at least two ways, one is using `next` and the other using
`send`. Let's see `next` first.

```python
z = fill_list_using_yields()
z_value_1 = next(z)
z_value_2 = next(z)
z_value_3 = next(z)
# list_value=[None, None]
```
In the above snippet, we didn't explicitly send any value to the generator and it appears that
the generator assumed that we sent back `None`s. There is an important but simple reason for this.
Calling the `next` function calls the `__next__` method. If this `__next__` is the
[default method](https://github.com/python/cpython/blob/23a567c11ca36eedde0e119443c85cc16075deaf/Lib/_collections_abc.py#L326)
(which is [indeed](/generators/generator-class/) the case for `fill_list_using_yields`),
then `__next__` simply
[calls](https://www.python.org/dev/peps/pep-0342/#new-generator-method-send-value)
the `send` method with a `None` argument. Thus, calling `next(z)` is equivalent to calling
`z.__next__()`, which is equivalent to calling `z.send(None)`. This clarifies why `list_value`
contains `None`s. If we were to redefine `__next__`, perhaps by
[defining](/generators/generator-class/#minimal-example-from-scratch)
a generator class, then the outcome could've been different.

### Drive using `send`
Let's look at `send` now. When we use the `send` method, we explicitly send back a value
to the generator. Expectedly, trying to call `send` without any argument fails.

```python
fill_list_using_yields().send()
# TypeError: send() takes exactly one argument (0 given)
```
Let's try sending some value.

```python
fill_list_using_yields().send('hello, generator')
# TypeError: can't send non-None value to a just-started generator
```
This also failed! In a way, it's good that this failed. In the above snippet, we're calling
`send` on a fresh generator object `fill_list_using_yields()` which hasn't yet started
executing. This means that the execution hasn't yet reached the first `yield` point. Only
`yield` expressions can receive a value using `send`. So, the above snippet attempts to send
the value `'hello, generator'` even though the generator is not yet ready to receive the value.
This is why we must send a value that is completely useless and `None` uniquely
quailifies[^priming]. This is called *priming the generator*.

Let's try again. This time, we'll send `None` the first time, and some other values second and
third time.

```python
w = fill_list_using_yields()
w_value_1 = w.send(None)  # = next(w)
w_value_2 = w.send('alpha')
w_value_3 = w.send(3659)
# list_value=['alpha', 3659]

print(w_value_1)
# None
print(w_value_2)
# None
print(w_value_3)
# None
```
Voila! `list_value` was updated with the values we sent.

### Parentheses are important
Removing the parentheses from the first two `yield` expressions would result in an immediate
syntax error.

```python
def fill_list_using_yields_wrong():
    list_value = [yield, yield]
    print('list_value={}'.format(str(list_value)))
    yield
#   File "<ipython-input-60-985d7314e596>", line 2
#     list_value = [yield, yield]
#                   ^
# SyntaxError: invalid syntax
```

A `yield` expression must be enclosed within parentheses with only two exceptions, as shown
below.

```python
def yield_parentheses_exceptions():
    # Exception I: you can skip the parentheses
    value_1 = yield
    value_2 = yield 42

    # Exception II: you can skip the parentheses
    yield
    yield 42

    # Not Exceptions: you must use parentheses
    value_3 = 12 + (yield)
    value_3 = 12 + (yield 42)
    print((yield))  # This is what PEP 342 gets wrong
    isinstance((yield), dict)  # This is what PEP 342 gets wrong
```
Note the use of double parentheses in `print((yield))` above. This is necessary even though
[PEP 342](https://www.python.org/dev/peps/pep-0342/#new-syntax-yield-expressions) says otherwise.
Sadly, PEP 342 is outdated and contains some incorrect information. When in doubt, the
[documentation](https://docs.python.org/3/reference/expressions.html#yield-expressions)
for your python version should provide some guidance.

### Why was the first example more difficult to run?
This is simply because calling `next` on `yield_is_an_expression` would fail on the
third try.

```python
q = yield_is_an_expression()
next(q)
# 1
next(q)
# 2
next(q)
# TypeError: unsupported operand type(s) for *: 'NoneType' and 'NoneType'
```
Calling `next` is equivalent to calling `send(None)`, as we saw
previously. Eventually, the generator would attempt to compute `product_value` as the product
of two `None`s which is unsupported, regardless of generators. This problem is not just
for `None`s; we can recreate the problem if we sent back a `dict` and a `list`.

```python
e = yield_is_an_expression()
next(e)  # prime the generator
# 1
e.send({'a'})
# 2
e.send([1])
# TypeError: can't multiply sequence by non-int of type 'set'
```
So, in order to drive `yield_is_an_expression`, we must send back values that support the
product operation, such as `['a']` and `3`.

```python
f = yield_is_an_expression()
next(f)
# 1
f.send(['a'])
# 2
f.send(3)  # no error
```
The following is another working example that may be helpful because it shows the order in
which data is sent back and forth.

```python
def sum_of_yields():
    sum_value = (yield 'send first value') + (yield 'send second value')
    print('sum_value={}'.format(sum_value))
    yield

w = sum_of_yields()
w_value_1 = next(w)
print(w_value_1)
# send first value

w_value_2 = w.send(100)
print(w_value_2)
# send second value

w_value_3 = w.send(23)
# sum_value=123
print(w_value_3)
# None
```

## Can a function have both `yield` and `return`?
Yes, it can. Let's see two cases.

### `return` before a `yield`
Even if we put a `return` before the first `yield`, the resulting function is still a generator
function.
```python
def return_before_yield():
    print('Beginning')
    return
    print('After return but before yield')
    yield
    print('End')

weird_gen_object = return_before_yield()
print(weird_gen_object)
# <generator object return_before_yield at 0x10807a120>

next(weird_gen_object)
# Beginning
# ---------------------------------------------------------------------------
# StopIteration                             Traceback (most recent call last)
# <ipython-input-79-a5dbbac1bc45> in <module>
# ----> 1 next(weird_gen_object)

# StopIteration:
```
This is another case of
[yield under an impossible branch](#yield-under-an-impossible-branch).
Obviously, we will never to reach the `yield` point because the generator function will return
permanently when it reaches `return`. Thus, when `return` is before all the `yield`s, then the
function is a generator function and has to be *driven* like a generator but behaves partially
like a simple function by permanently returning when it hits `return`.

### `return` after all `yield`s
FILL ME IN

See https://www.python.org/dev/peps/pep-0380/:

`return value` = `raise StopIteration(value)`
## `StopIteration` error
FILL ME IN

## Can `yield` be outside a function?
No[^yield-outside-a-function]. This can be easily checked and is mentioned in the
[documentation](https://docs.python.org/3/reference/expressions.html#yield-expressions).

```python
yield 1
#   File "<ipython-input-13-9f4dce03671c>", line 1
#     yield 1
#     ^
# SyntaxError: 'yield' outside function

(yield 1)
#   File "<ipython-input-80-4cb37392add7>", line 1
#     (yield 1)
#      ^
# SyntaxError: 'yield' outside function

class A:
    (yield 1)
#   File "<ipython-input-81-f2ee79229f43>", line 2
#     (yield 1)
#      ^
# SyntaxError: 'yield' outside function
```

## Can a `lambda` contain a `yield`?
Yes. But, we must use `yield` within [parentheses](#yield-is-an-expression).

```python
# Fails
lambda x: yield 'whatever'
# SyntaxError: invalid syntax

# Works
gen_function_via_lambda = lambda: (yield 1)
print(type(gen_function_via_lambda()))
# <class 'generator'>
```
In fact,
[python source](https://github.com/python/cpython/blob/23a567c11ca36eedde0e119443c85cc16075deaf/Lib/_collections_abc.py#L62)
uses a `lambda` to obtain `generator` type for
[use](https://github.com/python/cpython/blob/23a567c11ca36eedde0e119443c85cc16075deaf/Lib/_collections_abc.py#L370)
for `collections.abc.Generator`.
```python
generator = type((lambda: (yield))())
print(generator)
# <class 'generator'>
```

## Can you put a `yield` within a simple function?
The answer to this question seems to be "no", given that we hammered
[this point](/generators/introduction/#brevity-has-its-costs) repeatedly.
But, this is a trick question, perhaps, for job interviews. Think about it and then click on
**Answer** below to check for one possible answer.

??? Answer
    Yes, just put the `yield` within another function inside a simple function.
    ```python
    def this_is_a_simple_function():
        print('Entering simple function')
        def this_is_a_generator_function():
            print('Entering generator function')
            yield 1
            print('Exiting generator function')
        print('Exiting simple function')
        return 'done'

    output = this_is_a_simple_function()
    # Entering simple function
    # Exiting simple function

    print(output)
    # done
    ```

## Can a simple function return a generator object?
Another trick question for job interviews. Think about it and then click on **Answer** below.

??? Answer
    Yes. See below.
    ```python
    def simple_function_returns_gen_object():
        print('Entering simple function')
        def generator_function():
            print('Entering generator function')
            yield 1
            print('Exiting generator function')
        print('Exiting simple function')
        return generator_function()

    obj = simple_function_returns_gen_object()
    # Entering simple function
    # Exiting simple function

    print(obj)
    # <generator object simple_function_returns_gen_object.<locals>.generator_function at 0x107fe19e0>
    ```

    This demonstrates an important point. Just because a function `f` returns a generator object
    does not mean that `f` is guaranteed to be a generator function.



## Can a simple function return a generator object *without using `yield` inside it*?
Another trick question for job interviews. Think about it and then click on **Answer** below.

??? Answer
    Yes. Just move the generator function out.
    ```python
    def some_generator_function():
        yield 1

    def simple_function_no_yield():
        print('Entering simple function')
        print('Exiting simple function')
        return some_generator_function()

    obj = simple_function_no_yield()
    # Entering simple function
    # Exiting simple function

    print(obj)
    # <generator object some_generator_function at 0x10810aba0>
    ```
## Can a simple function return a generator object without using `yield` *at all*?
Another trick question for job interviews. Think about it and then click on **Answer** below.

??? Answer
    Yes. Define a [generator class](/generators/generator-class/#improved-example-using-abc)
    which avoids the use of `yield`.
    ```python
    from collections.abc import Generator

    def simple_function_no_yield_self_contained():
        print('Entering simple function')
        class SomeGeneratorClass(Generator):
            def send(self, value):
                super().send(value)
            def throw(self, typ, val=None, tb=None):
                super().throw(typ, val, tb)

        print('Exiting simple function')
        return SomeGeneratorClass()

    obj = simple_function_no_yield_self_contained()
    # Entering simple function
    # Exiting simple function

    print(obj)
    # <__main__.simple_function_no_yield_self_contained.<locals>.SomeGeneratorClass object at 0x107f92d90>
    ```
## Footnotes
[^pick-a-side-yield]:
    It's like we can never win with `yield`. Pick a side, `yield`!

[^true-equivalent]:
    We don't always need something to evaluate to `True` to choose the `if` branch instead of the
    `else` branch. For example, non-zero `int`s or `float`s, non-empty lists, and non-empty
    strings all evaluate to `True`. Similarly, `0`, empty lists, and empty strings evaluate to
    `False`.

[^priming]:
    You could argue that we could've allowed sending `'hello, generator'` the first time and
    then just thrown it away. That would certainly be possible but it would make for a poorer
    design. First, every programmer could send something different leading to unnecessary
    and useless magic strings in the code. Second, anybody reading the code would be confused
    as to why a particular first, throwaway value was chosen and whether or not it had any
    significance. And, finally, unnecessarily sending an object only to be thrown away would waste
    computation.

[^yield-outside-a-function]:
    Think of `yield` as a fish. It cannot survive outside the waters of a `function`.

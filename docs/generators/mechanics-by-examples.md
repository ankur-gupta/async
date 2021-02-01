# Mechanics by Examples
Curious readers may have lots of questions on how exactly `yield` behaves. Let's answer them with
examples.

## `yield` under an impossible `if` branch
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
a `True`-equivalent[^2] during some runs and a `False`-equivalent during some other runs?
Even if `some_variable.some_method()` does not use random numbers, code changes in this
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

LINK TO `yield` after a `return`.


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
LINK TO THE PEP THAT SHOWS THIS

DIFFERENTIATE BETWEEN STATEMENT AND EXPRESSION. LINK TO LAMBDA LATER.

```python
def fill_list_using_yields():
    list_value = [(yield), (yield)]
    print('list_value={}'.format(str(list_value)))
    yield

z = fill_list_using_yields()
z_value_1 = next(z)
z_value_2 = next(z)
z_value_3 = next(z)
# list_value=[None, None]
```
We [previously](/generators/introduction/#yield-is-a-two-way-street) saw that `yield`
can accept value as well. FILL ME IN

Typically, we send back values to the generator using the `send` method. Calling the `next`
function calls the `__next__` method. If this `__next__` is the default method (LINK ME TO SOURCE),
then `__next__` simply calls the `send` method with `None`. Thus, calling `next(z)` is
equivalent to `z.__next__()` which is equivalent to calling `z.send(None)`. This explains
why `list_value` contains `None`s.



Note that `yield` must be surrounded with parentheses. For example, this would fail.
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

We could've chosen to send back values. Let's see this using another example. As before,
`yield` must be wrapped within parentheses.

```python
def sum_of_yields():
    sum_value = (yield 'send first value') + (yield 'send second value')
    print('sum_value={}'.format(sum_value))
    yield

w = sum_of_yields()
w_value_1 = next(w)  # See next question to see why we have to call next first.
# send first value
print(w_value_1)

w_value_2 = w.send(100)
print(w_value_2)
# send second value

w_value_3 = w.send(23)
# sum_value=123
print(w_value_3)
# None
```



## Can a function have multiple `yield`s?
Yes. We will use this section to describe how `next` and `send` work.

```python
def multiple_yields():
    print('Entering')
    yield 1
    print('After first yield')
    yield 2
    print('Exiting')
```

We can drive the above generator using `next`.

```python
gen_object_1 = multiple_yields()
value_1 = next(gen_object_1)
# Entering
print(value_1)
# 1

value_2 = next(gen_object_1)
# After first yield
print(value_2)
# 2

value_3 = next(gen_object_1)
# Exiting
# ---------------------------------------------------------------------------
# StopIteration                             Traceback (most recent call last)
# <ipython-input-8-56c83a1da170> in <module>
# ----> 1 value_3 = next(gen_object_1)

# StopIteration:
print(value_3)
# NameError: name 'value_3' is not defined
```
Calling `next` the first time executes the code
starting from the beginning of the generator function to the first yield point, returns the
yielded value, and suspends execution. A second `next` call on the same generator object,
resumes execution just after the first yield and executes until the second yield point while
returning the second yield value. The third `next` call resumes execution just after the second
yield point and then executes hoping to find yet another yield, which it does not and so throws
the `StopIteration` error.

LINK TO THE PEP THAT TALKS ABOUT THE StopIteration error.

We can also drive the same generator using the `send` method instead. We saw
[previously](/generators/generator-class/) that
the `next` function simply calls the object's `__next__` method, which in turn, simply calls
`send(None)`. Before we discuss the correct way to call the `send` method, let's see some of the
wrong ways.

```python
gen_object_2 = multiple_yields()
gen_object_2.send()
# TypeError: send() takes exactly one argument (0 given)
```

Send requires one argument. We must send something. Let's try sending `'hello'`.

```python
gen_object_3 = multiple_yields()
gen_object_3.send('hello')
# TypeError: can't send non-None value to a just-started generator
```
We can't send `'hello'` either. This is an important point.
In the above snippet, `gen_object_3` hasn't started execution and is not suspended anywhere.
A `send` call must correspond to
a suspended yield point. In the above snippet, `gen_object_3` hasn't been suspended yet (it
hasn't even begun yet) and therefore

At this point, curious readers may have lots of questions. For example, how many yield statements
can we have, can a function have both yield and return statements, what is meant by syntactically
reachable, what if we have a yield in only one of the `if-else` branches.


```python
def yield_and_return():
    print('Entering')
    yield 1
    print('After first yield')
    yield 2
    print('Exiting')
    return 3
```

## Can `yield` be outside a function?
No[^yield-outside-a-function]. This can be easily checked.

```python
yield 1
#   File "<ipython-input-13-9f4dce03671c>", line 1
#     yield 1
#     ^
# SyntaxError: 'yield' outside function

class A:
    yield 1
#   File "<ipython-input-14-3d7af3485611>", line 2
#     yield 1
#     ^
# SyntaxError: 'yield' outside function
```

## Can a `lambda` contain a `yield`?
Yes. But, we must use `yield` as an expression instead of a statement.

```python
gen_function_via_lambda = lambda: (yield 1)

print(type(gen_function_via_lambda()))
# <class 'generator'>
```

Link to this snippet:
https://github.com/python/cpython/blob/23a567c11ca36eedde0e119443c85cc16075deaf/Lib/_collections_abc.py#L62
```python
generator = type((lambda: (yield))())
```





EXAMPLES  - burniong questions



3. Multiple `yield`s
5. `yield` with `return`, `yield` after a `return`
6. `yield` within a function within a function
7. `yield` in a lambda. Mention Kotlin.
8. Trick interview Q1: write a function that returns a generator function without using yield within the function.
   Ans: Write a generator class and in another function return an object of the class.
9. Trick interview Q1: write a function that returns a generator function without using a class or yield within the function. Ans: Either define a a generator function separately (or use `range` which is a generator already)
    and return the generator function. CAUTION: `range` is not a generator.

(done) 2. Can a `yield` be outside a funtion?
(done) 1. `yield` under a `if False`, `yield` under a random-chanced `if` branch.
(done) 4. Must we return something with yield?








THIS SHOULD BE A NEW SUBSECTION IN THIS PAGE
The above class also shows us exactly how `send` works, even though we didn't actually use it.
Let's see another example.


When a generator function is written to receive a value from a yield expression but the caller
does not send any value, it is assumed that a None is sent.


also helps understand send, next, `StopIteration` etc.





## `send` mechanics
`send` is a very important feature of the generator. You can *drive* a generator by using `send`
alone, even when the generator is not written to receive a value from the caller.
Consider the following example.

```python
def drive_by_send():
    yield 1
    yield 2
    value = (yield) + (yield)
    print('sum of last two received values = {}'.format(value))
    yield 5
```

## Footnotes
[^yield-outside-a-function]:
    Think of `yield` as a fish. It cannot survive outside the waters of a `function`.

[^pick-a-side-yield]:
    It's like we can never win with `yield`. Pick a side, `yield`!

[^2]:
    We don't always need something to evaluate to `True` to choose the `if` branch instead of the
    `else` branch. For example, non-zero `int`s or `float`s, non-empty lists, and non-empty
    strings all evaluate to `True`. Similarly, `0`, empty lists, and empty strings evaluate to
    `False`.

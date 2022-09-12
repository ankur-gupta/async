# Syntax
???+ question "Syntax"
    What syntax is needed to correctly define a _suspendable_ entity ?

We have previously noticed that the syntax we used was janky, even for pseudocode. 

## Function
Let's begin with a relevant albeit non-traditional definition of a function.

!!! summary "Function"
    A function is a packaged unit of code that has at least two stages: <br/><br/>
    1. *defining stage* <br/>
    2. *calling stage* <br/>

These two stages are so trivial that one may wonder why are we even discussing it.
Hopefully, the necessity of this discussion will soon become clear.

### Defining stage
A function may be defined by executing code such as the following.
```python
def square(x):
    x_squared = x ** 2
    return x
```
Executing the above code doesn't actually execute the contents of `square`[^1]. In fact,
when we execute the above code, we don't even know which value of `x` needs to be squared. Thus,
it would not even make sense to execute the contents of `square` while defining it.

### Calling stage
The function `square` can be called like so:
```python
square(2)  # 4
```
*Calling* a function means executing a code snippet of the form `function_name(optional_args)`.
It just so happens that *calling* `square` executes the contents of `square`. This may sound
torturedly pedantic. After all, what else could we possibly mean by *calling a function*?
It turns out that there may be situations where calling an entity may not necessarily execute
the contents of the entity.

## Call &ne; Execute, necessarily
Let's broaden our notion of what it means to *call an entity*. In python, functions are not
the only [callable](https://docs.python.org/3/library/functions.html#callable) entities.

### Calling a class
Let's consider the following simple class.
```python
class HelloWorld():
    def __init__(self, data):
        self.data = data

    def __repr__(self):
        return f'{self.__class__.__name__}({self.data})'

    def __call__(self):
        return self.data

    def set(self, new_data):
        self.data = new_data
```

We can *call* the `HelloWorld` class just like we could *call* a simple function.
```python
y = HelloWorld(1)
```
Calling `HelloWorld` executes the default class constructor (`__init__`) but not the other
methods in the class. Thus, calling the class does not execute the entire content of
the class. In fact, executing the entire content of the class is not even a properly defined
operation[^2].

### Calling an object of the class
Since `HelloWorld` has a `__call__` method, an instance of the `HelloWorld` class is also a
[callable](https://docs.python.org/3/library/functions.html#callable).
```python
y()  # 1
```
Again, calling `y` only executes the contents of the `HelloWorld.__call__` method. It does not
execute the contents of `__init__` method (aka the class constructor) or any of the other
class methods.

The two examples above help disassociate the notion of *calling an entity* from
*executing the contents of an entity*. Calling an entity has multiple semantics.

## Simple Function
We need some terminology to distinguish functions that have different calling semantics.

!!! summary "Simple Function"
    A function is a *simple function* if calling the function executes the contents of
    the function. In other words, the *calling stage* is the same as the *execution stage* for
    a simple function.

A simple function is the ubiquitous function that every modern day programmer learns.
For example, `square` is a simple function.

## Suspendable function cannot be a simple function
We suspected that this might be the case since [Footnote 1](/suspendables/control/#fnref:1)
of the previous section. This is best described by a simpler counter example — let's assume that
a *suspendable function* can indeed be expressed as a *simple function* in the
following pseudocode:

```python title="Pseudocode"
suspendable function count_hello:
    count = 0
    while true:
        count = count + 1
        print string(count) + " hello"
        release control  # to caller

count_hello()  # 1 hello
count_hello()  # 2 hello
count_hello()  # 3 hello
```

The above pseudocode has a few problems:

1. A fourth call to `count_hello` would inevitably result in `4 hello`. There is no way to reset the state
back to `1 hello`. We're stuck!
2. What if we wanted to have two instances of `count_hello` simultaneously, possibly at different
states?
3. What if we wanted to receive `1 hello` as returned value instead of just being printed?

When we execute the contents of a *suspendable function*, we create an *internal state* for that
particular chain of execution. This internal state needs to be stored somewhere. If we then need to
initiate a second execution of the same *suspendable function*, we need to find a different
place to store the separate internal state of this second chain of execution.

A simple function can maintain at most one _internal state_[^3]. Unless global variables are involved,
every call to a simple function is independent of each other, and as a result simple functions
do not suffer from the problems mentioned above.

!!! summary "Suspendable function &ne; Simple function"
    Unlike a simple function, calling a suspendable function should not execute its contents.

Later in the course, we will see how exactly a suspendable function would come to differ from a
simple function.

## Suspendable function implemented as a class
Being able to independentally execute different instances of the same code is the hallmark of
object oriented programming. We could easily define `count_hello` as the following python
class.

```python
class CountHello:
    def __init__(self):
        self.count = 0

    def run(self):
        self.count = self.count + 1
        print(str(self.count) + " hello")
```

We can now instantiate multiple instances of `CountHello`, each with its own independent state.

```python
z = CountHello()
w = CountHello()
z.run() # 1 hello
z.run() # 2 hello
z.run() # 3 hello
w.run() # 1 hello
z.run() # 4 hello
```

The above code is proper Python code. We didn't need to use any fancy keywords to indicate
a transfer of control. Each call to `CountHello.run` performs one iteration that
increases the value of `count` by `1` and then returns the control back to the caller. The state
`count` is preserved between calls to `CountHello.run`.

The actual python code in the `CountHello` class performs the same work as the pseudocode
suspendable function `count_hello` but without needing any notion of suspendability. 

???+ question "Question"
    Why do we even need suspendable functions at all if we can just use classes?

The answer to this question is anti-climactic: *suspendable entities are not essential*.
We could do asynchronous programming using other constructs such as classes or callbacks.
In fact, the above `CountHello` class is a rudimentary implementation of a python generator,
which we will study [later](/generators/generator-class/) in the course.

The benefit of using a suspendable entity (over using a class) is readability, which is 
[highly valued](https://en.wikipedia.org/wiki/Zen_of_Python) in Python.

## Callbacks instead of suspendable functions
Callbacks provide an alternative approach to perform asynchronous programming, one that is
completely independent to suspendables. Callbacks are commonly used in Javascript but
also [available in python](https://www.python.org/dev/peps/pep-3156/#coroutines-and-the-scheduler).
Though, even in Javascript,
callbacks are sometimes considered [ugly](http://callbackhell.com/) and async/await is
considered
[easier](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Async_await).
These [slides](https://www.slideshare.net/elizarov/introduction-to-coroutines-kotlinconf-2017),
even though about Kotlin, provide an excellent comparison of the callback approach against the
coroutine approach.

## Verbosity & Natural Representation
Our pseudocode examples ignored syntax ambiguities. But when programming in python, we want
readable, clear, consistent, terse, and unambiguous syntax. Python is popular because of its 
wonderful syntax and it would be a shame to lose that quality simply to write an 
asynchronous program.

A class is much more verbose than a function. A class requires boilerplate code
(such as constructors) which is not needed for a function. For our toy example,
the class `CountHello` does not seem to be much more verbose than the function `count_hello` but
the class-equivalent for a moderate sized suspendable task could be quite verbose. 
A functional form is also a more [natural way](https://www.python.org/dev/peps/pep-0342/#motivation)
to express many forms of computation.  

Consider the suspendable psedudocode function `mashed_potatoes`, discussed
[previously](/suspendables/cooking-is-like-programming/#mashed-potatoes).
This function has one control transfer point. We can create a non-suspendable class out of the 
suspendable `mashed_potatoes` by splitting the code before and after the control transfer point.

=== "Suspendable Function Form"
    ```python title="Pseudocode"
    suspendable function mashed_potatoes(minutes, auto_shutoff):
        peel_potatoes()
        cut_potatoes()
        start_boiling_potatoes(minutes=minutes, auto_shutoff=auto_shutoff)

        release control  # to the caller
        
        finish_boiling_potatoes()
        mash_potatoes()
        stir_potatoes_with_butter()
    ```

=== "Non-suspendable Class Form"
    ```python
    class MashedPotatoes:
        def __init__(self, minutes, auto_shutoff):
            self.minutes = minutes
            self.auto_shutoff = auto_shutoff

        def before_control_transfer():
            peel_potatoes()
            cut_potatoes()
            start_boiling_potatoes(minutes=15, auto_shutoff=true)
        
        def after_control_transfer():
            finish_boiling_potatoes()
            mash_potatoes()
            stir_potatoes_with_butter()
    ```

It may be argued that adding a few new keywords to legalize the syntax for a suspendable function 
is better than having to create a class every time we want to suspend control.

| `mashed_potatoes`                                          | `MashedPotatoes`                                                                                         |
|----------------------------------------------------------  |----------------------------------------------------------------------------------------------------------|
| Very little boilerplate code                               | More boilerplate code (such as constructor)                                                              |
| Pseudocode                                                 | Proper python code                                                                                       |
| Needs `release` & `control` words                          | No special keywords needed                                                                               |
| Order of computation is specified by the function itself   | User needs to remember to order of computation: `before_control_transfer()` → `after_control_transfer()` |


## Extended Function
The above discussion may be summarized into the following two points.

- [x] A suspendable function cannot be a simple function
- [x] We prefer a suspendable function over classes or callbacks


This leads us to define a new type of function.

!!! summary "Extended Function"
    A function is an _extended function_ if calling the function does not execute the
    contents of the function. Executing the contents of the function requires an _execution stage_
    beyond the _calling stage_.

Allowing a function to have yet another stage (beyond the defining and calling stage) lets
us solve all of our problems albeit at the cost of increased complexity.
This concept of an extended function is best described via an example.

```python
# Stage 1: Defining stage
def example():
    print("Executing contents of example")
    yield 1

type(example)
# function

# Stage 2: Calling stage 
x = example()
type(x)
# generator

# Stage 3: Execution stage
y = next(x)
# Executing contents of example

print(y)
# 1
```

The above example requires a lot of explanation, which we will provide in the
[Generators](/generators/introduction/) section later in the course. For now, this example 
serves to demonstrate that calling `example` neither prints `Executing contents of example` nor provides
`1` as a return value.
Instead, calling `example` simply returns a `generator` object. Executing the contents of
`example` requires yet another step — calling `next` on the `generator` object `x`.
Evidently, the extended function `example` is very different from the simple function `square`.
The extended function `example` has one extra stage — the execution stage.

Wary readers may notice that calling `example` is reminiscent of calling the constructor of a
class. This is precisely true! Every invocation `example()` produces a separate,
independent `generator` object. The function `example` serves as a concise form of class
declaration. In other words, an extended function (such as `example`) may be thought of as
verbosity-reducing syntactic sugar over a class declaration.

This is not the only example of *trickery* that *hides complexity behind syntactical brevity*.
A very similar construct is a
[context manager decorator](https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager),
which allows us to define a context manager as a function instead of having to write a class
with boilerplate methods. 

_Arguably, hiding complexity behind syntactical brevity is a defining
feature of python itself._

## Footnotes
[^1]:
    Though, the syntax of the contents of the function is checked.
[^2]:
    In order to properly define what it means to execute the entire content of a class, we will
    need to first some answer questions, such as — what's the order in which various class methods are
    executed the arguments and which arguments need to be passed to the class methods.
[^3]:
    A simple function may use a global variable to save some _internal state_. This global variable
    may be accessible by future function invocations or even by other functions. Thus, a simple
    function using a global variable to maintain an _internal state_ suffers from the same
    problems outlined above. This is one of the reasons why global variables are considered a
    [bad programming practive](https://stackoverflow.com/questions/10525582/why-are-global-variables-considered-bad-practice).

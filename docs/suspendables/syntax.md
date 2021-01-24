# Syntax
*What syntax is needed to correctly define a suspendable entity ?*

Our cooking example does not worry about syntax at all. But when programming in python, we want
clear, consistent, terse, and unambiguous syntax. Python is popular because of its wonderful
syntax and it would be a shame to lose that quality simply to define a *suspendable entity*.
Requiring the maintenance of python's terse syntax is partially responsible for making
async programming difficult[^1].

Before we discuss the syntax of *suspendable entities*, it's necessary to revisit the syntax of a
*simple function*.

## Simple Function
It may be too obvious to notice but a *simple function* has two stages to it:

1. Defining stage
2. Calling stage

These two stages are so trivial that one may wonder why are we even discussing it.
Hopefully, the necessity of this discussion will soon become clear.

### Defining stage
The following example defines a simple function by executing the following python code:
```python
def square(x):
    x_squared = x ** 2
    return x
```
Executing the above code doesn't actually execute the contents of `square`[^2]. In fact,
when we execute the above code, we don't even know which value of `x` needs to be squared. Thus,
it would not even make sense to execute the contents of `square` while defining it.

### Calling stage
The function `square` can be called like so:
```python
square(2)  # 4
```
*Calling* a function means executing code that looks like this `function_name(optional_args)`.
It just so happens that *calling* `square` executes the contents of `square`. This may sound
torturedly pedantic. After all, what else could we possibly mean by *calling a function*?
But, this is exactly the point where a *simple function* would differ from a
*suspendable entity*.

### Call &ne; Execute, necessarily
Before we jump to the syntax of *suspendable entities*, let's discuss two more examples that
illustrate that *calling an entity*  and *executing the contents of the said entity* are not
necessarily the same thing. In python, functions are not the only
[callable](https://docs.python.org/3/library/functions.html#callable) entities.
Let's consider the following simple class:
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
methods in the class. Thus, calling a class is not the same as executing the entire content of
the class. In fact, executing the entire content of the class is not even a properly defined
operation[^3].

Since `HelloWorld` has a `__call__` method, an object `y` of the type `HelloWorld` is also a
[callable](https://docs.python.org/3/library/functions.html#callable). We can call `y` like so
```python
y()  # 1
```
Again, calling `y` only executes the contents of the `HelloWorld.__call__` method. It does not
execute the contents of `__init__` method (aka the class constructor) or any of the other methods.

Thus, we have demonstrated that calling an entity does not always execute the contents of the
said entity. Calling an entity has multiple semantics. This answers the question
*What else could we possibly mean by calling a function ?*
that we posed to ourselves [previously](#calling-a-regular-function).

This long discussion allows us to now provide a definition of the phrase *simple function*.

!!! summary "Simple Function"
    A python function is a *simple function* if <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (1) it has two stages, a *defining stage* and a *calling stage*, <br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (2) calling the function executes the contents of the function <br/>
    Thus, for a simple function, the *calling stage* is the same as the *execution stage*.

Python functions defined without using the keywords `async`, `await`, `yield`, and `yield from`
are simple functions. We will see examples of non simple functions later on.

## Suspendable Entity cannot be a Simple Function
This is best described by a counter example -- let's assume that a *suspendable entity* can indeed
be expressed as a *simple function* in the following pseudocode:

<pre><code><strong>function</strong> count_hello
    count = 0
    <strong>while true</strong>:
        count = count + 1
        print string(count) + " hello"
        <strong>suspend execution</strong>  # suspend instead of return
</code></pre>

What happens when we *call* this *suspendable entity* defined using a *simple function*
syntax[^4]?
<pre><code>count_hello()  # 1 hello
count_hello()  # 2 hello
count_hello()  # 3 hello
</code></pre>

The above pseudocode has two problems:

1. The fourth call would inevitably result in `4 hello`. There is no way to reset the state
back to `1 hello`. We're stuck.
2. What if we wanted to have two instances of `count_hello` simultaneously, possibly at different
states?

When we execute the contents of a *suspendable entity*, we gain an *internal state* for that
particular chain of execution. This internal state needs to be stored somewhere. If we want to
initiate a second execution of the same *suspendable entity*, we need to find a different
place to store the separate internal state of this second chain of execution.

*Why does a simple function not have the same problems?* A simple function does not maintain
internal state[^5]. Every call to a simple function is independent of each other.

Thus, we cannot design a suspendable entity to have the same invocation behavior
as function.

!!! summary "Suspendable &ne; Simple Function"
    Something must be different between a suspendable invocation and a simple function invocation.

Later in the course, we will see how exactly a suspendable entity would come to differ from a
simple function.

## Suspendable Entity could be a class
Being able to independentally execute different instances of the same code is a hallmark of
object orienter programming. We could easily define `count_hello` as the following python
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

The above code is runnable in Python. We didn't need to use any fancy keywords that would
indicate a suspension of control. Each call to `CountHello.run` performs one iteration that
increases the value of `count` by `1` and then returns the control back to the caller. The state
`count` is preserved between calls to `CountHello.run`.

The actual python code in the `CountHello` class performs the same work as the pseudocode
suspendable function `count_hello` but without needing any notion of suspendability. You may be
asking: *why do we even need suspendable entities at all if we can just use classes?*

The answer to this question is anti-climactic: *suspendable entities are not essential*.
We could do asynchronous programming using other constructs such as classes or callbacks.
In fact, the above `CountHello` class is a simple albeit non-standard

In order to reset the state, we may have to define a count_hellos2 with the same exact content. And, then count_hellos3, and so on.
Why doesn't this happen to a regular function? Because regular function does not maintain a state of its own. If there's no state, then there's nothing to reset.
By the way, this is how classes and objects work. We define a class once. Then we can instantiate an object of that class as many times as we want -- each object gets its own independent state.And, as we will see, calling a coroutine function in python is very much like calling a class's constructor to instantiate a new object with its own independent state.

## Footnotes
[^1]:
    This issue has been discussed many times. The most relevant discussion happens in the 2001
    [PEP 255](https://www.python.org/dev/peps/pep-0255/#bdfl-pronouncements)
    about generator syntax. The syntax adopted in PEP 255 *hides complexity behind syntactical brevity*,
    which may be considered a disservice to python beginners. However, many pythonistas
    (including the author(s) of this course) find the chosen syntax acceptable.
    A similar syntax "trickery" is commonly used to succinctly define
    [context managers](https://docs.python.org/3/library/contextlib.html). Arguably,
    hiding complexity behind syntax brevity is a defining feature of python itself.
    Finally, if none of this footnote makes sense, please ignore it; it's not
    necessary for the flow of the course. We discuss this topic in detail with examples in the
    main text.
[^2]:
    Though, the syntax of the contents of the function is checked.
[^3]:
    In order to properly define what it means to execute the entire content of a class, we will
    need to first answer questions like: what's the order in which various class methods are
    executed the arguments and which arguments need to be passed to the class methods.
[^4]:
    In this example, there is no other entity to assume control when `count_hello` suspends
    execution. In other words, when `count_hello` hits `suspend execution`, the control is sent
    back to the caller of `count_hello`. We tackle the question of control in the next page.
[^5]:
    A simple function may use a global variable to save some internal state.
    A function that can modify a global variable can indeed retain its state and use that state in another, future function invocation of itself. Technically, such a function does transfer control and data without losing its state. But, we don't consider the use of global variables an acceptable implementation of a coroutine for [general programming reasons](https://stackoverflow.com/questions/10525582/why-are-global-variables-considered-bad-practice).

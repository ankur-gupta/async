# Suspendables
On the previous page, we made a cooking *recipe* suspendable. A cooking *recipe* is
analogous to a python *function* and a *suspendable recipe* is analogous to
*suspendable function* in python. This is where programming becomes more complex than cooking.
A *suspendable function* requires a lot more design thought than a *suspendable recipe*.
We will see in the next few pages that a *suspendable function* cannot be the same
type of entity as a *simple function* in a programming language.

Let's not design our *suspendable function* just yet. Instead, let's think of an abstract python
entity called *suspendable* or *suspendable entity*, which may or may not be related to a
plain, vanilla python function. For now, let's keep our options open and remember that a
*suspendable entity* could be a python function or python class or something else entirely.

!!! summary "Suspendable (or Suspendable Entity)"
    A python entity containing an ordered sequence of instructions that can suspend and resume
    execution while maintaining its state.


The above definition of *suspendable entity* is quite loose and leaves a lot of room for
design. In other words, the above defintion does not fully nail down the specification of
the *suspendable entity*. We can design many distinct entities that can satisfy the above
definition. To phrase it yet another way, there can be many specific types of
*suspendable entities*.

In order to define specific types of *suspendable entities*, we will primarily look at
the following two features.

## Control
Who gets the control when the *suspendable entity* suspends execution ?

## Syntax
What syntax is needed to correctly define a *suspendable entity* ?

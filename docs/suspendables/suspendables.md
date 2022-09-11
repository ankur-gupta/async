# Suspendables
On the previous page, we made a cooking *recipe* suspendable. A cooking *recipe* is
analogous to a python *function* and a *suspendable recipe* is analogous to
*suspendable function* in python. 

This is where programming becomes more complex than cooking.
A *suspendable function* requires a lot more design thought than a *suspendable recipe*.
We will see in the next few pages that a *suspendable function* cannot be the same
type of entity as a *simple function*.

Let's not design our *suspendable function* just yet. Instead, let's think of an abstract python
entity called *suspendable* or *suspendable entity*, which may or may not be related to a
plain, vanilla python function. In other words, a *suspendable entity* could either be a 
python function, or a python class, or something else entirely.

!!! summary "Suspendable (or Suspendable Entity)"
    A python entity containing an ordered sequence of instructions that can suspend and resume
    execution while maintaining its state.


This definition of a _suspendable entity_ is purposefully vague and incomplete. It does not 
fully nail down the specification of the *suspendable entity*. This leaves us a lot of room 
for design. We can design many distinct entities that can satisfy the above
definition. To phrase it yet another way, there can be many different implementations of
a *suspendable entity*.

Any implementation of a *suspendable entity* needs to answer the following questions.
The rest of this chapter will try to answer these two questions.

### Control
Who gets the control when a *suspendable entity* suspends execution ?

### Syntax
What syntax is needed to correctly define a *suspendable entity* ?

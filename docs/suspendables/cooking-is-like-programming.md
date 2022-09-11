# Cooking is like programming
It is easier to understand async python programming if we temporarily forget about the
[structured control flow](https://en.wikipedia.org/wiki/Structured_programming) as it exists in
contemporary programming. 

_This is easier said than done._

Most of us are so well-versed with these common control flow constructs
(such as *functions*) that it is difficult for us to completely let go of these ideas.

_So, let's use a trick!_

Cooking is surprisingly similar to programming. We will use cooking as
an example to understand the need for a *suspendable*, which we will then define on the next page.

## Bear Crumpet
Let's say we invent a completely new baked good and we name it *bear crumpet*. Our first ten
attempts to make an bear crumpet required a lot of experimentation — we needed to adjust the
amount of ingredients and we had to play around with the oven temperature and timing.
After the first ten attempts, we perfected our invention — it's 50% tastier than a
*regular crumpet* and 75% more awesome than a *bear claw*!

We need to save the results from our experimentation so we can get the perfect bear crumpet every
time without having to redo the experiments. What we need is *an ordered sequence of instructions*,
_i.e._ a _recipe_ describing how to make an *bear crumpet*. A cooking recipe can be thought of as 
a _function_ in programming. But, we purposefully, choose the word _recipe_ instead of a 
_function_ because _recipe_ does not come with a predefined, hard-to-disassociate
connotation that comes with _function_.

Let's make two other food items using recipes — salad and mashed potatoes.

## Salad
The following pseudocode represents a *recipe* for a very simple salad:
<pre><code><strong>recipe</strong> salad
    chop_lettuce_into_the_bowl()
    chop_tomato_into_the_bowl()
    chop_cucumber_into_the_bowl()
</code></pre>

Let's say a single chef executes this recipe to make a salad. There are three steps to this 
simple recipe:

1. Chop lettuce
2. Chop tomato
3. Chop cucumber

For a salad, the order of the steps does not matter. We could, just as easily, have 
chopped tomato first, then lettuce, and then cucumber. Here is an alternative but somewhat crazier
recipe for the same salad:

1. Start chopping lettuce but stop midway
2. Start chopping cucumber but stop midway
3. Finish chopping lettuce
4. Finish chopping cucumber
5. Chop tomato

Not only does the order of the three steps not matter, we can even interleave the steps
and still have a salad[^1].

## Mashed Potatoes
Let's consider something more complex than a salad. The following pseudocode is a simple *recipe* 
for mashed potatoes:
<pre><code><strong>recipe</strong> mashed_potatoes
    peel_potatoes()
    cut_potatoes()
    boil_potatoes(minutes=15, auto_shutoff=<strong>true</strong>)  # Could we do other work here?
    mash_potatoes()
    stir_potatoes_with_butter()
</code></pre>
As before, let's assume that a single person executes this recipe. The first two steps are simple 
enough — peel and cut potatoes. The third step, boiling the potatoes, is where the complexity 
arises. 

Unlike the salad recipe, the order of steps is important for mashed potatoes. 
We must cut and peel the potatoes before we boil them[^2] and we need to finish boiling the 
potatoes before we can mash the boiled potatoes. It takes 15 minutes to boil the potatoes during 
which the chef is idle but cannot begin mashing the mashing the potatoes. 
The chef *must* wait until the potatoes are boiled, which is a waste of time[^3].

## Salad & Mashed Potatoes
Now, let's consider the scenario in which a single chef has to make the salad as well as the 
mashed potatoes. The chef can work serially — make the salad first and then make the mashed 
potatoes, but this is not an efficient use of time. The chef can save some time[^4] by
interleaving the steps like this:

1. Peel potatoes
2. Cut potatoes
3. Set the potatoes to boil
4. Temporarily suspend working on mashed potatoes
5. Make the salad
6. Resume making mashed potatoes

If we suspend the execution of `mashed_potatoes` just after starting the boiling step, 
we can use that time to execute `salad` instead of waiting by idly for the potatoes to boil. 
Thus, we can be more efficient with our time if we make `mashed_potatoes`a *suspendable* recipe.
The same motivation applies to *functions* in programming.


## Footnotes
[^1]:
    The `make_salad` example is, in fact, an ideal use-case for threads. Multithreading was available [before](https://en.wikipedia.org/wiki/Thread_(computing)) multicore processors were available. The primary motivation for multiple threads back then was to interleave unrelated tasks on the single CPU core. See [Thinking Outside the Synchronisation Quadrant - Kevlin Henney - YouTube](https://www.youtube.com/watch?v=2yXtZ8x7TXw&feature=emb_title).
[^2]:
    It is well-known that boiling uncut and unpeeled potatoes makes for a poorer recipe.
[^3]:
    Think of the chef as a _process_ that is forced to stay idle while waiting for some 
    computation to finish.
[^4]:
    This is how [Samwise Gamgee](https://media.giphy.com/media/q7kofYLObTVUk/giphy.gif) does it.

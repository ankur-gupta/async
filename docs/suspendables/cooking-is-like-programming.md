# Cooking is like programming
It is easier to understand async python programming if we temporarily forget about the
[structured control flow](https://en.wikipedia.org/wiki/Structured_programming) as it exists in
contemporary programming. This is easier said than done.
Most of us are so well-versed with these common control flow constructs
(such as *functions*) that it is difficult for us to completely let go of these ideas.
So, let's use a trick. Cooking is surprisingly similar to programming. We will use cooking as
an example to understand the need for the concept of a *suspendable*, which we will then define on the next page.

## Bear Crumpet
Let's say we invent a completely new baked good and we name it *bear crumpet*. Our first 10
attempts to make an bear crumpet required a lot of experimentation — we needed to adjust the
amount of ingredients and we had to play around with the oven temperature and timing.
After the first ten or so attempts, we perfected our invention — it's 50% tastier than a
*regular crumpet* and 75% more awesome than a *bear claw*!

We need to save the results from our experimentation so we don't have to repeat the experiments every time we want to make an bear crumpet.
What we need is *an ordered sequence of instructions* or a *recipe* describing how to make an *bear crumpet* perfectly at every future attempt. A recipe may be reminiscient of a
*function* in programming but the word *recipe* does not come with predefined, hard-to-disassociate
implications that come with using the word *function*.

Let's make two other food items using recipes — salad and mashed potatoes.

## Salad
We can write the *recipe* for a very simple salad using the following pseudocode:
<pre><code><strong>recipe</strong> make_salad
    chop_lettuce_into_the_bowl()
    chop_tomato_into_the_bowl()
    chop_cucumber_into_the_bowl()
</code></pre>
Let's say a single person executes this recipe to make a salad. There are three steps to this simple recipe — chopping lettuce, chopping tomato, and chopping cucumber. For a salad, the order of these three steps does not matter. We could just as easily have chopped tomato first, then lettuce, and then cucumber and still have a salad. This recipe is so simple that we could have something even crazier — start chopping lettuce, stop midway, start chopping cucumber, stop midway, finish chopping lettuce, finish chopping cucumber, and then finally chop tomato.

Not only does the order of the three steps not matter, we can even interleave the steps
and still have a salad[^1].

## Mashed Potatoes
Let's consider something more complex than a salad. The following is a simple *recipe* for
mashed potatoes written using pseudocode:
<pre><code><strong>recipe</strong> make_mashed_potatoes
    peel_potatoes()
    cut_potatoes()
    boil_potatoes(minutes=15, auto_shutoff=<strong>true</strong>)  # we can do other work here
    mash_potatoes()
    stir_potatoes_with_butter()
</code></pre>
As before, let's assume that a single person executes this recipe. The first two steps are simple enough — peel and cut potatoes. The third step, boiling the potatoes that have been cut and peeled, is where the complexity arises. Unlike the salad recipe, the order of steps is important for mashed potatoes. We must cut and peel the potatoes before we boil them[^2] and we need to finish boiling the potatoes before we can mash the boiled potatoes. It takes 15 minutes to boil the potatoes during which the cook is idle but cannot begin mashing the
mashing the potatoes. It appears that the cook *must* wait until the potatoes are boiled, which seems to be a waste of time.

## Salad & Mashed Potatoes
Now, let's consider the scenario in which a single person has to make both the salad and the mashed potatoes. The cook can work serially — make the salad first and then make the mashed potatoes, but this is not the most efficient use of time. The cook can save some time[^3] by
interleaving the steps like this:

1. Peel potatoes
2. Cut potatoes
3. Set the potatoes to boil
4. Temporarily suspend working on mashed potatoes
5. Make the salad
6. Resume making mashed potatoes

If we suspend the execution of `make_mashed_potatoes` recipe just after starting the boiling step, we can use that time to execute `make_salad` instead of waiting by idly for the potatoes to boil. Thus, we can be more efficient with our time if we make `make_mashed_potatoes`
a *suspendable* recipe. The same motivation applies to *functions* in programming.


## Footnotes
[^1]:
    The `make_salad` example is, in fact, an ideal use-case for threads. Multithreading was available [before](https://en.wikipedia.org/wiki/Thread_(computing)) multicore processors were available. The primary motivation for multiple threads back then was to interleave unrelated tasks on the single CPU core. See [Thinking Outside the Synchronisation Quadrant - Kevlin Henney - YouTube](https://www.youtube.com/watch?v=2yXtZ8x7TXw&feature=emb_title).
[^2]:
    It is well-known that boiling uncut and unpeeled potatoes makes for a poorer recipe.
[^3]:
    This is how [Samwise Gamgee](https://media.giphy.com/media/q7kofYLObTVUk/giphy.gif) does it.

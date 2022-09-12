## Yet Another Tutorial
Nearly every other tutorial, in our opinion[^1], has the same drawbacks:

- [ ] Study material is presented in the chronological order of invention
    1. Iterators
    2. Generators
    3. Generator-based coroutines
    4. Native coroutines
- [ ] Study material contains defunct concepts
- [ ] Study material uses insufficient terminology

<!-- Capitalize the P in Python a la https://www.python.org/ -->
For many students, it is actually easier to first learn the basics of asynchronous programming in a 
modern language, like Go or Kotlin, and then come back to asynchronous programming in Python. 
This is because Go and Kotlin were designed to have asynchronous programming support very early 
on and therefore have a more streamlined learning experience. 
On the contrary, Python has acquired asynchronous programming via a series of superseding PEPs. 
As a result, some concepts have now become defunct (such as, generator-based coroutines) 
and some ideas need to be redefined (such as, the difference between iterators and generators). 
Teaching defunct concepts and obsolete definitions makes learning unnecessarily difficult. 

## What this course does differently?
We aim to bring the streamlined learning experience to Python. In this course,

- [x] study material is presented in an increasing order of complexity instead of the 
chronological order of invention
- [x] study material omits defunct concepts
- [x] terminology is invented or borrowed from modern languages, as needed

## Course Requirements
- [x] Proficiency in Python syntax
- [x] Basic knowledge of Python data structures
- [x] Time and patience

## Time commitment
This course is divided into 4 chapters, shown as top-level tabs. Depending upon your previous
experience with async programming (in any language), it may take you anywhere between 4 hours to
4 days to complete the material. Feel free to skip and return back to the material as needed.

## Python version
This course describes **Python 3.8.5**. Earlier Python versions may not work and are not
recommended. Python async sytnax is still being refined. Specific examples from this course
may need to be updated but the general concepts should apply to all later versions of Python.

## Let's begin
We'll start with some commentary on why async Python programming is difficult to learn.

## Footnotes
[^1]: Apologies for the bluntness. We aim to be direct. Please take no offense.
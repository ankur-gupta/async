# Generators _v_ Coroutines
TBD

## Generators

1. Implements Iterator 
    - [Generators](/generators/a-better-way-to-drive/#generator-implements-iterator-protocol) yes
    - [Coroutines](/coroutines/introduction/#coroutine-does-not-implement-the-iterator-protocol) no
2. Can be used with a `for` loop
3. Have send, throw, close methods


## Coroutines

1. Implements Awaitable
2. Can be used with an event loop
3. Have send, throw, close methods

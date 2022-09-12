# Buying a plane ticket

Let's look at a classic example — network calls. 

Suppose we want to buy a plane ticket. The same plane ticket is available for purchase, albeit at 
different prices, from three online vendors — Vendor Red, Vendor Green, and Vendor Blue. We need to 
query each vendor to get the ticket price so as to find the cheapest ticket.

This classic example is often used to demonstrate the need for asynchronous programming itself.

## The need for asynchronous programming
### Serial & synchronous
We could do this task serially and synchronously — send a request to one server, wait for it to reply, and
send another request to another server and wait for it to reply and then the same for the third server.
Now, that we have the prices from all three servers, we can find the lowest price. 

```python title="Serial & synchronous — pseudocode"
function get_price(vendor)
    request = create_request(vendor)
    request.wait()
    print(f'Received price from the vendor {vendor}!')
    return request.result

red_price = get_price(red)  # Wait 1000ms
blue_price = get_price(blue)  # Wait another 300ms
green_price = get_price(green)  # Wait yet another 400ms

best_price = min([red_price, green_price, blue_price])
```

This approach is wasteful. The table below shows the times taken by each vendor to reply to 
our query. No matter how we order the three vendors, serial and synchronous approach is 
guaranteed to take 1700 ms, which is the sum of the three individual query times.

| Vendor | Query time | Total time |
|--------|------------|------------|
| Red    | 1000 ms    |            |
| Green  | 300 ms     |            |
| Blue   | 400 ms     | 1700 ms    |

### Parallel & synchronous
This approach is similar to the serial & synchronous approach except that we execute the three
queries parallely. But, each query is still synchronous in the sense that we wait for that 
query to finish before doing anything downstream.

Depending on the programming language, we can implement parallelism using either processes and 
threads. The pseudocode below uses processes. 

```python title="Parallel & synchronous — pseudocode (using processes)"
function get_price(vendor)
    request = create_request(vendor)
    request.wait()
    print(f'Received price from the vendor {vendor}!')
    return request.result

p_red = Process(get_price, args=(red, ))
p_green = Process(get_price, args=(green, ))
p_blue = Process(get_price, args=(blue, ))

# Start the processes
p_red.start()
p_green.start()
p_blue.start()

p_red.join()  # Waits 1000ms
p_green.join()  # No wait; we've already waited 300ms
p_blue.join()  # No wait; we've already waited 400ms

red_price, green_price, blue_price = p_red.result, p_green.result, p_blue.result
best_price = min([red_price, green_price, blue_price])
```

All three queries are dispatched at roughly the same time, when we call the corresponding 
`p_vendor>.start()` methods. We then wait for the query to Vendor Red to finish first, 
when we call `p_red.join()`, which takes 1000ms. During these 1000ms, the queries to Vendors
Blue and Green, which take 300ms and 400ms respectively, have already finished. So, we don't have 
to wait on the vendors when we call `p_green.join()` and `p_blue.join()`.

Overall, this approach takes a 1000ms, which is obviously better than the serial & synchronous 
approach. 

| Vendor | Query time | Total time |
|--------|------------|------------|
| Red    | 1000 ms    |            |
| Green  | 300 ms     |            |
| Blue   | 400 ms     | 1000 ms    |

But, this approach is still on resources. Each process takes separate memory to run. 
Each process also needs time to be setup. Depending on the machine, we may not have 3 processes to 
spare. And, in a real-world case, such as for our own Kayak-type website, we may have thousands 
of queries that need to be run parallely. We don't have a machine that has thousands of processors.
And, even if we did, creating thousands of processes would have trememdous overhead which would 
negate any multiprocessing gains. We could use a worker pool of three processes that rotates through
thousands of queries three at a time. This three-at-a-time parallelism would still take a long 
time to work through thousands of queries because only three queries would be active at a time.

Alternatively, we could use threads (within the main process) to implement the parallelism.
Threads share memory and can be started and ended much faster than processes.

Obviously, in the default CPython, this would not be truly parallel because the 
[Global Interpreter Lock](https://wiki.python.org/moin/GlobalInterpreterLock)
only allows one thread to have access to the interpreter at one time. 
There are different Python implementations that do not have a GIL. 
[IronPython](https://wiki.python.org/moin/IronPython) is one such example. 
In a GIL-free python implementation, the threads may be able to behave truly parallely. 

Our toy example only needs three threads and would definitely benefit from a GIL-free
implementation. However, our own Kayak-type website would require thousands of threads,
which is more efficient than thousands of processes, but is still not efficient enough. 
Thousands of threads may not even be possible on a machine. We could use a worker pool of 
three threads that rotates through thousands of queries three at a time. As with processes, 
this three-at-a-time GIL-free parallelism would still take a long time to work through 
thousands of queries because only three queries would be active at a time.


#### There is room for improvement
Servers have internal latencies and transferring data over the network
takes time. We don't need to perform a lot of computation, we simply need to receive the price data.
There is no inherent need to create threads or processes simply to handle the incoming data.
Before the advent of asychronous programming, we were forced to create multiple threads or 
processes because there was nothing better available.

### Not parallel & asynchronous
This approach uses a single thread within a single process. We don't have the overhead of 
creating new threads or processes. The magic here is that we made the `get_price` function
suspendable. The pseudocode below has janky syntax. This is purposeful as we will explain in 
the next section on [syntax](/suspendables/syntax/). For now, we will ignore the syntax issues and 
focus on how asychronous programming can solve our problem even more efficiently than parallelism.

```python title="Not Parallel & asynchronous — pseudocode"
suspendable function get_price(vendor)
    request = create_request(vendor)
    release control
    request.wait()
    print(f'Received price from the vendor {vendor}!')
    return request.result

# Initiate all requests but don't wait for them to finish
get_price(red)
get_price(green)
get_price(blue)

# Resume suspended functions and wait for them to complete
red_price = get_price(red).wait()  # Waits 1000ms
green_price = get_price(green).wait()  # No wait; we've already waited 300ms
blue_price = get_price(blue).wait()  # No wait; we've already waited 400ms

min([red_price, green_price, blue_price])

# Did you notice the janky syntax in this pseudocode? This is why we need 
# to answer the syntax question in the next few section.
```

We call `get_price` three times, once for each of the three vendors. This sends a query to each of 
the three vendor websites but doesn't immediately wait for the queries to be answered. 
Once all three queries have been sent, the pseudocode then waits for the queries to be answered 
in a particular, distinguished order -- it waits for red to finish, then green to finish, and 
then red to finish. Similar to the parallel & synchronous approach, we only have to wait a 1000ms
in all.

| Vendor | Query time | Total time |
|--------|------------|------------|
| Red    | 1000 ms    |            |
| Green  | 300 ms     |            |
| Blue   | 400 ms     | 1000 ms    |

Asynchronous approach is just as fast as the parallel & synchronous approach. But, asynchronous 
approach can handle thousands of queries within the same thread and the same process. 
Even if we had a thousand queries, we could start all of the queries nearly simultaneously
without waiting for them to finish.  Finally, let's not forget that we could still use
process-based parallelism along side asychronous programming.

### Not just theoretical
[Nginx vs Apache](https://stackoverflow.com/questions/12924124/why-would-i-choose-a-threaded-process-based-approach-vs-asynchronous-web-server)
provides a real-world, industrial-strength example where asynchronous programming outperforms 
parallelism via multithreading.



## Summary

| Approach                                   | Time       | Processes | Threads       | Limited by          |
|--------------------------------------------|------------|----------|---------------|---------------------|
| Serial & Synchronous                       | 1700 ms    | 1        | 1             | -                   |
| Parallel & Synchronous (Processes)         | 1000 ms    | 1 + 3    | 1 / process   | processor count     |
| Parallel & Synchronous (Threads, GIL-free) | 1000 ms    | 1        | 1 + 3         | processor count     |
| Asychronous                                | 1000 ms    | 1        | 1             | network bandwidth   |

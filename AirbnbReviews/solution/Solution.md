## "Airbnb Reviews Analyzer" - Solution

### What is wrong?
You probably noticed that our app is using very high number of threads.
If we would analyze more than 3K reviews we will hit `Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread`
(Don't do that. We tried. It will make your Mac restart :) )

Analyzing a thread dump with [](fastthread.io) shows the following unpleasent message:

![High number of threads](threads.png)

### Why?
 
The root cause is our definition of `ExecutionContext`, we currently use `Executors.newCachedThreadPool()`.

It creates a thread pool that creates new threads *as needed*. 
It will reuse previously constructed threads when they are available. Since we analyzing all of the reviews concurrently, 
and every analysis is quite slow (thanks to the `executeSuperComplexStuff` method, that sleeps for 3 seconds), we are exposing ourselves to using an unbounded number of threads.

A quick solution is to just use a fixed thread pool that limits the max numbers of threads in our pool.
```
   Executors.newFixedThreadPool(10)
```

If we'll want to use a thread pool that limits its thread count to the # of available processors, use this:
```
   Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors())
```

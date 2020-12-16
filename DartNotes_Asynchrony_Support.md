# Dart Asyncronous & Parallel Programming Support

Ordinarily, code executes synchronously - one operation blocks execution of the next until it completes. It can be particulary inefficient and damaging to application responsiveness if long running, indeterminate or compute intensive tasks are executed synchronously. Unlike synchronous operations, an asynchronous operation is non-blocking. Once initiated, the asynchronous operation *allows* other operations to be executed, before itself is completed. 

Dart's asyncrhronous programming support enables:

* Compute intensive tasks to be handled in background worker '*threads*'
* Indeterminate network and disk IO tasks and other external events (e.g. user gestures, etc.) to be processes asynchronously
 
The basic constructs for asynchronous programming in Dart are *Future*s and *Stream*s. 

A Future object represents an asynchronous computation with a **single** completion value, or error, available at some time in the future. Callbacks that handle the value or error, once it becomes available (i.e. once the async operation represented by this future is completed), may be registered. For example:

```dart
//getFuture method below will be executed asynchronously
//the return value from that computation will be of type int
Future<int> future = getFuture();
future.then((value) => handleValue(value))
      .catchError((error) => handleError(error));
```
* Future<int> is a future whose completion value type is of int.
* the `then()` method registers a callback to handle sucessful future completion with a value.
* the `catchError()` method registers a callback to handle failure with an error.

Note that the `then()` (and `catchError()`) methods above reurn future objects. So in effect, `then().catchError()` is chaining two asynchronous calls. This approach may be used to chain multiple async calls e.g. `then().then().then()...catchError();`

If the registered callback returns a value (or error) other than a Future<xxx>, the returned value is wrapped by Dart in a Future that completes with the returned value (or error), ensuring that `then()` or `catchError()` always returns a Future. This means that the final `catchError()` in a multi `then()` chain can handle the error from all the preceeding `then()` functions, including the original asynchronous call.

Futures are limited to delivering a single value/error asynchronously, at some point in the future. Conversely, a stream can deliver **zero or more** values/errors over time.

A Stream is a source of asynchronous events. Each event may be either a data (or value) event, in which case the value is referred to as an element of the stream, or an error event,  in which case it is a notification that some error has occured. We suscribe to the stream  (i.e. register callbacks that respond to the data events and optionally error events) to take advantage of the stream. Streams may be single-subscription - only allowing a single suscriber (i.e. listerner) during the entire life of the stream and not raising any events until it has a suscriber. Alternatively, streams may be multi-subscription type - i.e. broadcast streams - allowing zero or more subscribers and raising events regardless of the number of subscribers.

Stream subscription is via `listen()` method, allowing us register `onData`, `onError` and `onDone` handlers and to set the `cancelOnError` property. Note that the implementation of `foreach()` internally calls `listen`. The `StreamSubscription` object returned by listen may be used to control the event flow (cancel, pause, etc). Once we have subscribed to the stream, we can use/manipulate the data in elegant and interesting ways (e.g `map()`, `where()`, etc). These methods are similar to those available in Dart's Iterable class.  

The following table illustrates that, in Dart, Streams are the asynchronous counterparts of Iterables. Please note that Java Stream class represents something entirely different to Dart Stream. In Java, the concept of Stream is more concerned with describing a data source and the computational operations to be performed in aggregate on that data source, including how those operations are to be performed - sequentially or in parallel. The original Java collection classes (e.g. Lists, Maps, etc) are more focused on how to manage and directly access (get/put, etc) their elements. 

| Mode             |  Single Value  |  Zero or More Values  |
| ------------ | -------------- | --------------------- |
| Synchronous  |      int       | Iterable<int>         |
| Asynchronous | Future<int>    | Stream<int>           |


```
TODO Explain Creating Streams
```

Public Dart packages which provide advanced and interesting stream and ansynchrony convenience features include `async` and `stream_transform` packages.


Apart from  `then` and `catchError`, `listen` and `forEach` future and stream API methods which enable asynchronous handling of futures and streams, there are the `async`, `await` and `await for` keywords which allows asynchronous code to be written/used very much like synchronous code.

## Using `async` and `await` with Futures


An `await` expression causes execution to be suspended until the associated future completes. For example:

```dart
var version = lookUpVersion(); //lookUpVersion returns a future
print('Version = $version'); 
//unexpectedly, the above prints: Version = _Future
 .
 .
var version = await lookUpVersion(); //'await lookUpVersion()' is an "await expression"
print('Version = $version'); 
//as expected, the above prints: Version = 2020.2
```
In the above, without `await` execution does not wait, leading to unpredictable results (e.g. when toString is called on an as yet uncompleted future object).

To use `await`, the containing function/method must be marked as `async` and must also be declared as having a return value of type Future, e.g.:

```dart
// note async marker before method opening braces
Future<String> checkVersion() async {
    .
    .
    var version = await lookUpVersion();
    .
    .
    return version;  
    // Note you don't explicitly have to create Future<String> here to be returned
    // Dart will ensure what that the return value is wrapped in a Future as appropriate
    // per the method declaration (so as Future<String> in this case, with completion 
    // value equal to $version
}
```

Conveniently, `await` allows try-catch-finally as in normal synchronous usage, e.g.:
```dart

Future<String> checkVersion() async {
    .
    .
    try{
        var version = await lookUpVersion();
        return version;  
    }catch(e){
        //...
        return 'Unknown';
    }
}
```

The right hand part of the await expression usually resolves to a Future - if it doesn't, dart wraps is in a Future ensuring that await works as normal.

## Using `async` and `await for` with Streams

`async` and `await for` may be used to handle stream values. `await for` can only be used in an async function/method. The construct is used viz:

```dart
 .
 .
 .
 // inside async method
 await for (varOrType identifier in expression) {
     //can use break/return here to stop listening
     //to the stream
 }
```

## Understanding Dart Asynchrony and Parallelism

Streams in Dart are simply convenience constructs to handle data/error events whose arrival time is unknown. Futures are the equivalent construct for single events. It should be noted that the fact that a Dart program references Futures (or Streams) **does not necessarily imply that the program is multi-threaded**. 

By default, a Dart program consists of a single *isolate*. An *isolate* has a **single** thread, with its own **private** memory. This means that, by default, a Dart program is single threaded. Event handling, and the appearance of **concurrency** in single-threaded Dart programs is achieved using an asynchronous event despatch mechanism.  Every isolate runs an event loop as part of its single thread execution. The event loop allows the isolate to handle asynchronous tasks - these are tasks that have been scheduled for execution due to the occurence of an event. The tasks can be on one of two different task queues: the microtask queue or the event queue. Microtasks always run first, but they are mainly internal tasks that the developer doesn't need to worry about.

An operation that returns a future (e.g. async methods) schedules the computational task encapsulated by the future to be added to the event queue. The isolate's single thread continues to execute sequentially until it falls idle. At this point, the isolate executes the event loop - polling the event queue and executing queued tasks. The implemention of Future is such that when the task associated with a given future completes, the task encapsulated by the future's callback (`then()` or `catchError()`, as appropriate) is added to the event queue (i.e. scheduled for later execution on the event queue). 

Beyond the simplest asyncrony support case of single-thread Dart event handling, two other cases are important:

### Asynchronous IO Support

Disk and network IO operations are indeterminate and handled asynchronously in Dart. Typically, IO calls returns futures or streams. The computational task encapsulated by the Future/Stream generally involves a call to a native implementation (e.g. to read bytes from file). The underlying native implementation may or may not be multi-threaded, depending on the nature of the operation, the data and the system (single core vs multicore, etc). Nonetheless, when the system call completes (or partially completes), the associated Future (or Stream) triggers a completion (or data) event and the associated event handler task in scheduled for later execution. In the meantime, the isolate thread can continue to execute as normal, including events that pop up on the event queue.

As far as I understand it, where possible/advantageous, the underlying native IO implementation used by Dart usually rely on thread pools.

### Parallel Processing Support

An interesting dicussion of the difference between concurrency and paralellism may be found at stackoverflow.com/questions/1050222.

To achieve true **parallel** processing (as opposed to 'virtual' concurrency) within a Dart program, multiple isolates are required. If you have an expensive computation, you need to run it on a separate isolate. Unlike Java threads, isolates don't share any memory - inter-isolate communication and data exchange is via message passing over ports (more later).

```
TODO Explain How to create Isolates + Examples
```

# Dart Asyncronous & Parallel Programming Support

Ordinarily, code executes synchronously - one operation blocks execution of the next until it completes. It can be particulary inefficient and damaging to application responsiveness if long running, indeterminate or compute intensive tasks are executed synchronously. An asynchronous operation is non-blocking. Once initiated, it *allows* other operations to be executed, before the asynchronous operation is completed. Dart's asyncrhronous programming support enables:

* Compute intensive tasks to be handled in background worker '*threads*'
* Indeterminate IO tasks and other external events (e.g. user gestures, etc.) to be processes asynchronously
 
The basic constructs for asynchronous programming in Dart *Future*s and *Stream*s. 

A Future object represents an asynchronous computation with a **single** completion value, or error, available at some time in the future. Callbacks that handle the value or error once it becomes available (i.e. once the async operation represented by this future is completed) may be registered, for example:

```dart
Future<int> future = getFuture();
future.then((value) => handleValue(value))
      .catchError((error) => handleError(error));
```
* Future<int> is a future whose completion value type is of int.
* the `then()` method registers a callback to handle sucessful future completion with a value.
* the ```catchError()``` method registers a callback to handle failure with an error.

Note that the '```then()```' (and '```catchError()```') methods above reurn future objects. So in effect, ```then().catchError()``` is chaining two asynchronous calls. This approach may be used to chain multiple async calls e.g. ```then().then().then()...catchError();```

If the registered callback returns a value (or error) other than a Future<xxx>, the returned value is wrapped by Dart in a Future that completes in the returned value(or error), ensuring that ```then()``` or ```catchError()``` always returns a Future. This means that the final ```catchError()``` in a multi ```then()``` chain can handle the error from all the preceeding ```then()``` functions, including the original asynchronous call.

Futures are limited to limited to delivering a single value/error asynchronously, at some point in the future. Conversely, a stream can deliver **zero or more** values/errors over time.

A Stream is a source of asynchronous events. Each event may be either a data (or value) event - in which case it is referred to as an element of the stream - or an error event - in which case it is a notification that some error has occured. We suscribe to the stream  (i.e. register callbacks that respond to the data events (and optionally error events) to take advantage of the stream. Streams may be single-subscription - only allowing a single suscriber (i.e. listerner) during the entire life of the stream and not raising any events until it has a suscriber. Streams may also be multi-subscription type - i.e. broadcast streams - allwing zero or more subscribers and raising events regardless of the number of subscribers.

Stream subscription is via ```listen()``` method, allwing us register for ```onData```, ```onError```, ```onDone``` and to set the ```cancelOnError``` property. Note that the implementation of ```foreach()``` internally calls ```listen```. The ```StreamSubscription``` object returned by listen may be used to control the event flow (cancel, pause, etc). Once we have subscribed to the stream, we can use/manipulate the data in elegant and interesting ways (e.g map, where, etc). These methods are similar to those available for Dart Iterable.  

The following table illustrates that, in Dart, Streams are the asynchronous counterparts of Iterables. Please note that Java Stream class represents something entirely different to Dart Stream. Dart Streams are more akin to Queue's in Java. In Java, the concept of Stream is more concerned with describing a data source and the computational operations to be performed in aggregate on that data source, including how those operations are to be performed - sequentially or in parallel. The original Java collection classes (e.g. Lists, Maps, etc) are more focused on how to manage and directly access (get/put, etc) their elements. 

| Mode             |  Single Value  |  Zero or More Values  |
| ------------ | -------------- | --------------------- |
| Synchronous  |      int       | Iterable<int>         |
| Asynchronous | Future<int>    | Stream<int>           |


```
TODO Explain Creating Streams
```


Interesting Stream packages are: ```async``` and ```stream_transform```.



Apart from  ```then``` and ```catchError```, ```listen``` and ```forEach``` future and stream API methods which enable asynchronous handling of futures and streams, there is the ```async``` and ```await``` keywords which allows asynchronous code to be written/used very much like synchronous code.

## Using ```async``` and ```await``` with Futures


An ```await``` expression causes execution to be suspended until the associated future completes. For example:

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
In the above, without ```await``` execution does not wait, leading to unpredictable results (e.g. when toString is called on an as yet uncompleted future object).

To use ```await```, the containing function/method must be marked as ```async``` and must also be declared as having a return value of type Future, e.g.:

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

Conveniently, ```await``` allows try-catch-finally as in normal synchronous usage, e.g.:
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

```async``` and ```await for``` may be used to handle stream values. ```await for``` can only be used in an async function/method. The construct is used viz:

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

Streams in Dart are simply convenience constructs to handle data/error events whose arrival time is unknown. Futures are the equivalent construct for single events. It should be noted that the existence of Futures (or Streams) **does not necessarily imply the existence of  separate corresponding background threads**. 

By default, a Dart program is single threaded and consists of a single main *isolate*. An *isolate* has a **single** thread, with its own **private** memory. Event handling, and the appearance of **concurrency** in single-threaded Dart programs is achieved using an asynchronous event despatch mechanism. Unlike Java threads, isolates don't share any memory and they only communicate via message passing over ports (more later). Each isolate runs an event loop as part of its single thread execution. The event loop allows the isolate to handle asynchronous tasks - these are tasks that have been scheduled for execution due to the occurence of an event. The tasks can be on one of two different task queues: the microtask queue or the event queue. Microtasks always run first, but they are mainly internal tasks that the developer doesn't need to worry about.

An operation that returns a future (e.g. async methods) schedules the computational task encapsulated by the future to be added to the event queue. The isolate's single thread continues to execute sequentially until it falls idle. At this point is polls the event queue and executes and queued tasks. The implementions of Future is such that when the task associated with a given future completes, the task encapsulated by the future's callback is added to the event queue (i.e. scheduled for later execution when the isolate thread is idle). 

Beyond the simplest asyncrony support case of single-thread Dart event handling, two other cases are important:

### Asynchronous IO Support

Disk and network IO operations are indeterminate and handled asynchronously in Dart. Typically, IO calls returns futures or streams. The computational task encapsulated by the future/stream is generally a call to a native implementation. The underlying native implementation may or may not be multi-threaded (usually via thread pools) - depending on the nature of the operation, the data and the system (single core vs multicore, etc). Nonetheless, when the system call completes (or partially completes), the associated future (or Stream) triggers a completion (or data) event and the associated event handler task in scheduled for later execution. In the meantime, the isolate thread can continue to execute as normal, including events that pop up on the event queue.

### Parallel Processing Support

To achieve true parallel processing (as opposed to 'virtual' concurrency) within a Dart program, multiple isolates are required. If you have an expensive computation, you need to run it on a separate isolate.

```
TODO Explain How to create Isolates + Examples
```

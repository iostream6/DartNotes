# Flutter Rouitng and Navigation

Flutters navigation and routing support consists of two parts:

* An imperative API (a.k.a. Navigator 1.0).

* A new declarative API (a.k.a. Navigator 2.0).

The imperative API is centered around two classes: 

* `Navigator` -  a widget that manages a stack of `Route` objects.

* `Route` - an abstraction for an entity managed by a `Navigator` - concrete implementations, like MaterailPageRoute, typically represent a  page that renders over the entire screen.


To use the imperative API, we can simply call any of the appropriate static Navigator methods (e.g. Navigator.pop(), Navigator.push(), etc). The methods are discussed briefly below. Note that behind the scenes,the navigator widget state is actually managed by a `NavigatorState` object. All the navigator static methods that alter navigator widget state actually utilize `NavigatorState` (via Navigator.of()) in their implementation. We can directly refrence/use NavigatorState by calling Navigator.of)_ directly - this offeres some flexibility, for example allowing you to control which navigator state reference (e.g. the root vs nearest on widget tree) you obtain and alter.. In general the static methods on Navigator should be fine except you need better behaviour control.


## Navigator Methods

* push - pushes the given route onto the navigator that most tightly enclosed the given context. This will place the widget encapsulated by the given route above all others routes/widgets over the entire screen. The navigator route stack size is also increased by 1. This method returns a `Future` that completes to the result of the value (if any) passed to the pop call on the target route, allowing return values/results to be passed back from this route.

* pop - removes/pops the current top-most route off the navigator that most tightly enclosed the given context. Note that there are some lifecycle events that are triggered that allow you to respond to or block a pop request of be notified when it completes. See docs for more details.

* popUntil - call `pop()` repeatedly on the navigator that most tightly enclosed the given context, until the given predicate returns true (e.g ModalRoute.withName() allows to pop until a route with a certain name is reached).

* pushReplacement | pushAndRemoveUntil | replace

Also please note the named variants of the above methods as well.

The routes defines/declared when they are needed are referred to as anonymous routes. Flutter also supports **"named routes"** - these are defined at the widget app level. 

In one 'named routes' approach, we use the routes parameter of the widget app viz:

```dart
// top-level widget class passed to runApp()
Widget build(BuildContext context){
    return MaterialApp(
        ..
        ..
        routes: {
            '/': (context) => HomeScreen(), //first named route. Name = '/'
            '/details': (context) => DetailsScreen(), //second named route. Name = '/details'
        },
        ..
        ..
    );
}

// in another widget, to perform navigation to the named route
Widget build(BuildContext context){
  ..
  ..
  onPressed: (){
      Navigator.pushNamed(context, '/details');
  },
  ..
  ..
}
```
The above named route approach (via *routes* parameter) has a limitation - you can't parse path parameters/args from a route e.g. /details/**:id**. Another 'named routes' approach, (using *onGenerate* parameter, instead of *routes*,) avoid this limitation as shown below:

```dart
// top-level widget class passed to runApp()
Widget build(BuildContext context){
    return MaterialApp(
        ..
        ..
        onGenerate: (settings){
            //handle '/' route
            if(settings == '/'){
                return MaterailPageRoute(
                    builder: (context) => HomeSCreen()
                );
            }

            //handle '/details/:id route
            var uri = Uri.parse(settings.name);
            if(uri.pathSegments.lenght == 2 && uri.pathSegments.first== 'details'){
                var id = uri.pathSegments[1];
                return MaterialPageRoute(builder: (context) => DetailScreen(id: id));
            }

        },
        ..
        ..
    );
}

// in another widget, to perform navigation to the named route
Widget build(BuildContext context){
  ..
  ..
  onPressed: (){
      Navigator.pushNamed(context, '/details/1');
  },
  ..
  ..
}
```

# Declarative Navigation API (Navigator 2.0) - TODO
Flutter is a software development framework for building modern software UIs, based on the Dart programming language. The building blocks of Flutter UIs are reffered to as widgets. Widgets, typically, describe what their view should look like, given their current configuration and state. If and when the state changes, the widget rebuilds its description. Internally, the framework diffs the current description against the prior description in order to determine the minimal changes/updates to be effected by the underlying rendering system.

# Flutter Widgets

In Flutter, everything is a widget.Everything in your Flutter app UI is a widget and/or consists of widgets - the structure, layouts, animation, navigation, styles, etc are described by widgets. Widgets are Dart classes. Flutter comes with a host of built-in widgets - some of these are disccused in detail later. A large part of developing flutter apps involves using these built-in widgets to create/compose other custom widgets that describe the app views as part of a widget tree. The custom widgets you create will typically be one of two types: StatelessWidget and StatefulWidget. 

## StatelessWidget

A `StatelessWidget` is a widget that does not **require** a mutable state - all of the widget's configuration and data is provided to it at creation time. The widget describes part of the user interface by building a constellation of other widgets that describe the user interface more concretely. This build process is effected via the `build()` method. This method returns returns `Widget` and is called once by the framework for a given StatelessWidget when it is required to build that widget.


## StatefulWidget

A `StatefulWidget` is a widget that has/requires mutable state. They are useful when the part of the UI you are describing can change dynamically. To realize a stateful widget, we need to create two classes:

* A `StatefulWidget`sub-class
* A `State`sub-class associated with the StatefulWidget`sub-class

The implementation is such that all the `StatefulWidget` sub-class does is to create its associated `State` object(by calling `createState()` method). The `State` object is then responsible for state management and actual build out of the widget's description. It is the responsibility of the widget implementer to ensure that the state object is prompty notified whenever state changes, by calling `setState()` - internally, this notification will eventually trigger widget rebuild.

Key steps in `StatefulWidget` lifecycle are(**TODO see Begining Flutter fig 1.2 to improve**):

(a) createState   (b) mounted == true   (c) initState   (d) build   (e) setState   (f) deactivate   (g) dispose   (h) mounted == false.

# App Structure

Every Flutter app must have a main method which defines its entry point. Within the main method, Flutter's widget library's `runApp()` function is typically called with an 'app widget' as argument. An 'app widget' is a convenience `Widget` that wraps a number of widgets that are commonly required for an application. Two concrete 'app widgets' are the `MaterialApp` and `CupertinoApp` - these include/provide specific functionality for Material and Cupertino design respectively.

The runApp() function initializes, inflates the 'app widget' and attaches it to the screen. The 'app widget' typically defines app routes, theme, home widget, etc. For material-design apps, the `Scaffold` widget offers a basic visual layout structure  - including appBar, body, etc. Hence, a typical functional material app would be as follows:

**TODO**

flutter create --org com.munoor -i swift -a java --description 'Yet another TODO app' TODOFlutter
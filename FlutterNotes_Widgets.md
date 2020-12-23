# Flutter Widgets

In Flutter, everything is a widget.Everything in your Flutter app UI is a widget and/or consists of widgets - the structure, layouts, animation, navigation, styles, etc are described by widgets. Widgets are Dart classes. In general, widgets will be derived from one of two base classes: StatelessWidget and StatefulWidget. 

## StatelessWidget

A `StatelessWidget` is an immutable widget that does not maintain its own state information internally. Its sole purpose is to display data and this data is passed to it, by the framework, when it is created. If the data that it displays is changed (externally or somewhere else in the app), the framework knows to destroy the `StatelessWidget` and replace it with a new one created with the updated data.

`StatelessWidget` have a build method which returns `Widget` - this method is called once by the framework for a given StatelessWidget when it is required to build that widget.

## StatefulWidget

A `StatefulWidget` is a widget that maintains its own state information in a separate `State` object. The `State` object is created by the `createState()` method of the `StatefulWidget` and allows changes to state to be triggered by the `setState()` method. The `State` object must implement a `build()` method which returns the Widget - this method is called everytime the state change is triggered.

flutter create --org com.munoor -i swift -a java --description 'Yet another TODO app' TODOFlutter
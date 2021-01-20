
# Flutter State Management

State info that is local to or relevant to a single widget,e.g. the current page in a `PageView` widget, is called ephemeral state. Ephemeral state can be effectively handled using stateful widgets and does not require additional elaborate state management methods.

On the other hand, special measures/methods are needed to effectively manage app state - i.e state that is not ephemeral, but that is instead shared across many parts of your app. Examples of app state include login info (logged in status, logged in user, etc), shopping cart content, etc.

There are various options for state management in Flutter, including:

(a) BLoC   (b) MobX   (c) Redux   (d) InheritedWidget   (e) Provider   (f) others

We focus on the Provider method based on the *`provider`* package since this is one of the currently recommended methods for app state management in Flutter.

## App State Management with *provider*

App state management with the provider package is based on the Publisher-Suscribers pattern. There are a couple of classes that are of interest:

### `Provider`

A provider *is a widget* that makes some value available to wdgets below it on the widget tree.

### `ChangeNotifier`

A ChangeNotifier is a standard Flutter SDK class that provides change notification to its listeners. To use ChangeNotifier in provider based state management, we extend (or mixin) ChangeNotifier as a model class and call `notifyListeners()` method anytime the model changes e.g:

```dart
class ShoppingCartModel extends ChangeNotifier{
    final List<Item> _items = [];
    UnmodifiableListView<Item> getItems() => UnmodifiableListView<Item>(_items);

    void add(Item item){
        _items.add(item);
        notifyListeners();//Publish change notice!
    }

    void removeAll(){
        _items.clear();
         notifyListeners();//Publish change notice!
    }
}
```

The idea then is to attach this change notifier model onto the widget tree at the tree node **above all the widgets that depend** on this model's state.  For this, we turn to ChangeNotifierProvider and MultiProvider.


### `ChangeNotifierProvider` and `MultiProvider`

A ChangeNotifierProvider is a *`provider`* package widget that provides a **single instance** of a ChangeNotifier to its sub-ordinate nodes on the widget tree. For example, to attach the shopping cart change notifier model previously shown, above the 'app widget' node, we may do the following:

```dart
void main(){
    runApp(
        ChangeNotifierProvider(
            create: (context) => ShoppingCartModel(),
            child: MyApp() // the app widget
        )
    );
}
```

If we have more than one change notifiers (e.g. for different model types), then we can use `MultiProvider`. MultiProvider allows us to host multiple providers (e.g. multiple `ChangeNotifierProvider`s at the same node level) in a succint and elegant way:

```dart
void main(){
    runApp(
        MultiProvider(
            providers: [
                ChangeNotifierProvider(create: (context) => ShoppingCartModel()),
                ChangeNotifierProvider(create: (context) =>UserModel()),
            ],
            child: MyApp() // an app widget
        ),
    );
}
```


Once the ChangeNotifier model has been attached to the widget tree, we need:

* to be able to get a reference to the model, to read the model state or to call a model method that changes the model state. For this we use the static method `Provider.of<T>(context, listen: false).XXX`. 

* to be able to listen to the model state and react to changes in the model state, whenever they occur. For this we can use the static method `Provider.of<T>(context, listen: true).XXX`. We can also use `Consumer` and `Selector` as described below. 

The `Provider.of<T>` method obtains the nearest `Provider<T>` up the widget tree. So from the above snippet, using `Provider.of<ShoppingCartModel>(context, ...)` should find the ShoppingCartModel change notifier provided by the ChangeNotifierProvider up the widget tree refrenced by *`context`*. Note that apart from Provider.of, Consumer and Selector, we can also use context.watch, context.read and context.select - more details in the provider package documentation.


### `Consumer` and `Selector`

`Consumer` is a widget from the *`provider`* package. It offers a simple alternative to the explicit use of `Provider.of<T>(...)`. Behind the scenes, Consumer actually uses Provider.of.  The Consumer constructor requires a "builder" parameter - a function that takes a BuildContext, object value, and child widget. `Consumer<T>` brings the following advantages (compared with explicit calls to `Provider.of<T>`):

#### (a) Performance Optimization

Use of Consumer helps with performance optimization by providing more granular UI rebuilds:

Example 1:

```dart
Widget build(BuildContext context){
    return FooWidget(
        child: BarWidget(
            bar: Provider.of<Bar>(context)
        )
    );
}
```
In the above, each time Bar changes, both BarWidget and FooWidget will need to be rebuilt, even though only BarWidget depends on Bar. By using Consumer (as shown in next code snippet below), only BarWidget will be rebuilt if bar changes:

```dart
Widget build(BuildContext context){
    return FooWidget(
        child: Consumer<Bar>(
            builder: (_, bar, __) => BarWidget(bar: bar)
        )
    );
}
```

Example 2:

```dart
Widget build(BuildContext context){
    return FooWidget(
        foo: Provider.of<Foo>(context)
        child: BarWidget()
    );
}
```
In the above, each time foo changes, both FooWidget and BarWidget will need to be rebuilt, even though BarWidget does not depend on foo. By using Consumer (as shown in next code snippet below), BarWidget will not be rebuilt if foo changes:


```dart
Widget build(BuildContext context){
    return Consumer<Foo>()(
        builder: (_, foo, child) => FooWidget(foo: foo, child, child)
        child: BarWidget()
    );
}
```

#### (b) Avoid/Minimise Context Issues

Use of Consumer allows us to obtain a value from a provider when we dont have a BuildContext that is a descendant of said provider - where Provider.of would otherwise throw a ProviderNotFoundException:

Example 3:

```dart
Widget build(BuildContext context){
    return ChangeNotifierProvider(
        create: (_) => Foo(),
        //Next line will cause ProviderNotFoundException since context (ancestor widget tree) 
        //does not yet have Provider.of<Foo>
        child: Text(Provider.of<Foo>(context).value)
        )
    );
}
```

We can use Consumer to avoid the problem in the above as shown in the snippet below. It works cos the context ("_") passed on to the builder contains the needed change notifier.

```dart
Widget build(BuildContext context){
    return ChangeNotifierProvider(
        create: (_) => Foo(),
        child: Consumer<Foo>(builder: (_, foo, __)=>  Text(foo.value))
        )
    );
}
```

Note that to use more than one ChangeNotifierProvider in a Consumer, there are consumer variants like Consumer2, Consumer3, Consumer4, etc, that expose 2, 3, 4, etc, change notifiers simultaneously. 


A `Selector` is similar to a Consumer. The main difference is that it allows you to define explicitly which properties of a model you care about and can prevent widget rebuild if these properties did not change.


Example 4:

```dart
//model class
class User with ChangeNotifier{
    String _firstname, _lastname;
    double _balance;
    
    String get firstname => _firstname;

    set firstname(String fname){
        _firstname = fname;
        notifyListener();
    }

    String get lastname => _lastname;

    set lastname(String lname){
        _lastname = lname;
        notifyListener();
    }

    double get balance => _balance;

    set balance(double bal){
        _balance = bal;
        notifyListener();
    }
}

// widget class | build method
Widget build(BuildContext context){
    return Column(
        children: [
            //could also use Provider.of instead of 
            Consumer<User>(
                builder: (_, user, __) => Text(user.firstname)
            ),
            Consumer<User>(
                builder: (_, user, __) => Text(user.lastname)
            ),
            Consumer<User>(
                builder: (_, user, __) => Text(user.balance)
            )
        ]
    );
}
```

In the above, use of either Consumer or Provider.of will trigger a rebuild of all the text widgets anytime a notifyListener() is issued due to a change in the User model, regardless of whether the change affect the User model property that is being displayed by the Text widget. For example, if the user balance is changed, the text widgets for firstname, lastname and balance will all be rebuilt! We can avoid this by using the `Selector` widget. The `Selector` widget constructor has a *`selector`* parameter which is a callback that returns an object containing the value(s) needed by the  *`builder`* callback. Internally, by default, the `Selector` determines if its builder callback actually needs to be called (to rebuild the widget) by comparing the value returned by the *`selector`* to the value it previously returned. If the values are "equal", based on ==, the `Selector` assumes that this latest notifyListener() event did not affect a model property that this particular `Selector` is interested in and the builder is not called). This behaviour can be customised.

Re-writing the widget snippet in the previous example, we would have something like the following:

```dart
//model class - unchanged!

// widget class | build method now uses Selector rather than Consumer/Provider.of
Widget build(BuildContext context){
    return Column(
        children: [
            Selector<User, String>(
                selector: (_, user) => user.firstname,
                //user.firstname will get passed to the builder as value
                builder: (_, value, __) => Text(value)
            ),
            Selector<User, String>(
                selector: (_, user) => user.lastname,
                builder: (_, value, __) => Text(value)
            ),
           Selector<User, double>(
                selector: (_, user) => user.balance,
                builder: (_, value, __) => Text(value.toString())
            ),
        ]
    );
}
```

Note that it is possible to select more than one property/values from the model object - the easiest way of doing so without the need for a custom sub-model (for data consolidation and single return) class with overriden '==' method is to use a tuple from the *`tuple`* package - see docs for more details. Also, there are `Selector2`, `Selector3`, `Selector4`, etc classes with similar use case to `Consumer2`, `Consumer3`, `Consumer4`, etc.


# Flutter State Management

State info that is local to or relevant to a single widget is called ephemeral state. For example, the current page in a `PageView` widget, etc. Ephemeral state can be effectively handled using stateful widgets and do not require additional elaborate state management methods.

On the other hand,special measures/methods are needed to effectively manage app state, i.e state that is not ephemeral, but instead is shared across many parts of your app. Examples of app state include login info (logged in status, logged in user, etc), shopping cart content, etc.

There are various options for state management in Flutter, including:

(a) BLoC   (b) MobX   (c) Redux   (d) InheritedWidget   (e) Provider   (f) others

We focus on the Provider method based on the 'provider' package as this is one of the currently recommended methods for app state management in Flutter.

## App State Management with *provider*

App state management with the provider package is based on the Publisher-Suscribers pattern. There are a couple of classes that are of interest:

### `Provider`

A provider *is a widget* that makes some value available to wdgets below it on the widget tree.

### `ChangeNotifier`

A ChangeNotifier is a standard Flutter SDK class that provides change notification to its listeners. To use ChangeNotifier in *provider* based state management, we extend (or mixin) ChangeNotifier as a model class and call `notifyListeners()` method anytime the model changes e.g:

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

The idea then is to attach this change notifier model into the widget tree at the tree node **above all the widgets that depend** on this model's state.  For this, we turn to ChangeNotifierProvider and MultiProvider.


### `ChangeNotifierProvider` and `MultiProvider`

A ChangeNotifierProvider is a `provider` package widget that provides a single instance of a ChangeNotifier to its sub-ordinate nodes on the widget tree. For example, to attach the shopping cart change notifier model previously shown, above the 'app widget' node, we may do the following:

```
void main(){
    runApp(
        ChangeNotifierProvider(
            create: (context) => ShoppingCartModel(),
            child: MyApp() // an app widget
            )
    );
}
```

If we have more than one change notifiers (e.g. for different model types), then we can use `MultiProvider`. MultiProvider allows us to host multiple providers (e.g. multiple `ChangeNotifierProvider`s at the same node level) in a succint and elegant way:

```
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

The `Provider.of<T>` method obtains the nearest `Provider<T>` up the widget tree. Note that apart from Provider.of, Consumer and Selector, we can also use context.watch, context.read and context.select - more details in the provider package documentation.


### `Consumer` and `Selector`

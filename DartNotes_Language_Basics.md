# Dart Language Basics

## Language Introduction

Dart is a modern, strongly typed object-oriented language. Some important Dart characteristics/tidbits are outlined below:

* Dart code may be AOT compiled to native code providing good performance on workstations and mobile (x86 and ARM). It may also be JIT compiled - this is advantageous for speeding up development cycle. Lastly, Dart code can be compiled to Javascript to target web browsers.

* Create a new Dart program/project using the `dart` tool with 'create' argument.

* Although Dart is strongly typed, it has type inference, meaning that types can be ommited. For example:

```dart
int x = 42; // explicitly declare 'number' to be int type

var y = 42; // compiler infers that 'y' is of type int
y = 'abcd'; // compile error as compiler knows 'y' is of type int 
```

* Each Dart programmust have a `main` method with zero or more arguments. Examples are below.
```dart
//zero args main method
void main(){
    // code here
}
```

```dart
// args reference may be an empty list
void main(List<String> args){
    // code here
}
```
* The basic or built-in types include int, double, bool, String, List (a.k.a arrays), Set, Map and Symbol. These will be detailed later.

* Unlike Java, Dart does not have explicit access modifiers (public, protected and private). If a variable or method name begins with an underscore, it is private to its **library**, else it is public. An alternative to this is to use the `@protected` annotation available from the *meta* package - this makes it possible to use a variable or method only inside a class or its sub-classes
```
TODO confirm meta package annotation impact
```
* Declare a variable as `final` if it is to be set only once. Declare a variable as `const` if it is a compile time constant.
```
TODO Constant collections
``` 

## Built-in Types

### Numbers
Number types in Dart are subclasses of `num`. and may be `int` or `double`. `num` and its subclasses provide various methofs for operating on numbers including `toString()`, `ceil()`, `abs()`, etc. Alternatively, the `dart.math` library offers some other useful methods (e.g. `sqrt()`, `pow()`, etc).

### Strings

A String is a sequence of UTF-16 code units - they may be defined by single or double quotes:

```dart
String alpha = "Hello World";
String beta = 'Hello World';
```

Expressions (and variables) may be interpolated into strings as below:
```dart
String title = 'Software Engineer';
int level = 10;
//interpolate into string
print("Hello, $title"); // no need for curly braces for variables
print("Hello, ${user.name}"); // use curly braces for expressions
```

String concatenation may be achieved by placing string literals adjacent to each other. Multiline string literals may be created with tripple quotes. Examples of these are as below:
```dart
// String concatenation
String myString = 'Hello World,' ' my name is Vader!';

// multiline String - can use ''' ''' or """ """
String anotherString = '''Good Morning!
Welcome to the Dashboard''' 
```

### Lists
A Dart list is a resizable ordered array of objects of the same type. 
```dart
//create empty list
//
// new object instance, not the recommended way
List<int> aa = new List<int>();
//
// collection literal way - recommended
List<int> bb = [];

// create a list with some elements
List<int> cc = [1, 2, 3];
print('The second element is $cc[1]');
// add const to RHS to make const
List<int> dd = const [1, 2, 3];
dd[1] = -2; //compile error! 

// we can populate the list manually, with add/addAll 
List<int> ee = [1];
dd.addAll(anotherList.take(..));

// or with spread/null-aware-spread operators
List<int> ff = [1, ...anotherList]; //spread operator

List<int> gg = [1, ...?anotherList]; //null-aware-spread operator
// the null aware spread avoids throwing exceptions if spread target is null
```

List provides methods that may be used to operate on the contained data (e.g join, map, foreach, etc). **Note that foreach() loops through all elements of iterables (e.g Lists), regardless of break/return being encountered**. If you want to break/return midway during looping over all elements of an iterable, use `for  in`.

### Sets
A Dart list is an ordered collection of unique objects of the same type. We create a set in the following ways:

```dart
// A. 
Set<String> aa = new Set<String>();

// B. uses type inference, ommits 'new' keyword
var bb = Set<String>(); 

// C. Set literal way - RECOMMENDED WAY
Set<String> cc = {}; 

//D. 
var dd = Set<String>{};
```
Note that unlike Lists which use square brackets, for sets we use curly braces. Also unlike maps (discussed below) which may use **no OR two** parameter type info, with sets we always use just one (e.g. <String>).

Dart provides shorthand methds to create and populate a set viz:

```dart
//create and populate a set
Set<String> aa = {'Apple', 'Mango', 'Guava'};

//create and populate a set as a compile time constant
Set<String> bb = const {'Apple', 'Mango', 'Guava'};


//create and populate a set using spread operator (null aware may also be used)
Set<String> bb = const {'Apple', 'Mango', 'Guava', ...otherFruits};
```

### Maps
A Dart map is a collection of values of a given type, each associated with a unique key of a given type. We can create an empty map of int keys and String values viz:

```dart
// CREATE EMPTY MAP
// literal method - recommended.
Map<int,String> cc = {};

// another way.
var dd = <int,String>{};
// some other alternatives excluded here - use the recommended!

// CREATE AND POPULATE MAP
Map<int, String> countryRankings = {1: 'Finland', 2: 'Denmark', 3: 'Sweden'};
```

There are various ways to add entries to an existing map (e.g addEntries, addAll, putIfAbsent, etc). You can also add one item at a time using `map[key] = value;` You may also use the spread and null-aware-spread operator.

## Functions

A Dart function or method can have any number of **required** 'positional' parameters. They are required and positional because, when present in the function declaration, the function caller must provide these parameters in exactly the order that they have been declared. The required positional parameters **may** be followed by optional positional parameters (like Java) **OR** named parameters (like Python), but not both.

Optional positional parameters are declared by enclosing in square braces viz:

```dart
// userID and username are required positional parameters
// greeting and location are optional positional parameters
void greet(String userID, String username, [String greeting, String location]){
    if(greeting != null){
        // do something here
    }else{
        // do something else here
    }
}

```
Named parameters are optional, unless they are specifically marked wth the @required annotation. Named parameters are declared by enclosing in curly braces. Examples are as follows:

```dart
void greetAlpha(String username, {String greeting, String location}){
    //username is required, greeting and location are optional
}

void greetBeta({String greeting, String location}){
    //no required parameters, all are optional
}

void greetGamma({@required String greeting, String location}){
    //greeting is required, location is optional
}

void main(List<String> args){
    User user = getUser();

    // Examples of calling the above functions
    //
    // provide only the require parameter
    greetAlpha(user.username);
    greetGamma(greeting: user.greeting);
    //
    // provide required and optional parameters
    greetGamma(greeting: user.greeting, location: user.location);  
}
```
Optional parameters (positional or named) may be declared with default values - these default values will be used if none was provided by the function call. An example is below:

```dart


void greetTheta({@required String greeting, String location = 'Castle Black'}){
    //no required parameters, all are optional
}

void main(List<String> args){
    User user = getUser();

    // Example of calling the above functions
    //
    // provide required parameter, optional parameter will use default value
    greetTheta(greeting: user.greeting);  
    //
    // provide required and optional parameters
    greetTheta(greeting: user.greeting, location: user.location);  
}
```

Dart functions are first class objects. This means, amongst other things, that they may be assigned to variables and/or passed in as parameters to other functions. For example:

```dart
void pf(int e){

}

void main(List<String> args){
    // pf2 is a variable that references a function
    var pf2 = (int e){
        print('Number is $e');
    };
    // pf3 is a variable that references a function
    // the function is written in a shorthand way (more later)
    var pf3 = (int e) => print('Number is $e'); 

    var xx = [1, 2, 3]; // list literal, type infered

    //The following lines are exactly equivalent
    xx.foreach(pf); // foreach takes a function parameter
    xx.foreach(pf2);
    xx.foreach(pf3);
}
```
In the above, `pf2` and `pf3` are variables that references anonymous functions. The general structure of anonymous function in Dart is:
```
([[type] param1 [, ..]]){
    //codeblock
};
```
In the above, 
### TODO!!
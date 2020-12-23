# Dart Classes and OOP Support

## Classes

In Dart, unlike Java, multiple first-level public classes may be defined in a single file and the file name in Dart does not have anything to do with the name of the contained class(es).

Classes generally consist of fields and methods and optionally constructors and getters/setters. Unlike Java, all instance variables generate an implicit getter. All non-final instance variables also generate an implicit setter. For example:

```dart
class Car{
    int model;
    String name;
}

void main(List<String> args){
   Car car = Car(); //or Car car = new Car();
   car.model = 1975; //uses implicit setter
   print('Model is ${car.model}'); // use implicit getter
}
```

Explicit getters/setters may be written, for example for calculated fields. The getter structure is:
```dart
ReturnType get propertyName {
    .
    .
    .
    return xxx; 
    //xxx may be an instance variabe or an expression (from one or more variables)
}
//or with short hand notatin we may write
ReturnType get propertyName => xxx;

//In either case, note that there is no paralist after propertyName!!
```

The setter structure is:

```dart
set propertyName(ParameterType paramName, . . . ){
    // do actual setting here
}
```

### Constructors
A constructor is a special function with zero or more parameters, no return type, same name as the enclosing class (optionally with a suffix/identifier). It is used to create new instances of a class.

If no constructor is explicitly declared, a default no-arg constructor is provided by Dart - this constructor invokes the no-arg constructor in the superclass.

Unlike Java, Dart does not support constructor (or more generally method) overloading. Nonetheless, the following special *tricks* may be used to achive multiple constructors in Dart:

* Optional args [either named or positional]:
```dart
class BadCar{
    int model;
    String name;

    BadCar(int model, String name){...}
    BadCar(int model){...} //ERROR in Dart, works in Java
    BadCar(String name){...} //ERROR in Dart, works in Java
    //class BadCar will have errors in Dart as method overloading is not supported  
}

class GoodCar{
    int model;
    String name;

    GoodCar([int model, String name]){...}
    //class GoodCar will work in Dart 
    //method overloading achieved with optional parameters     
}
```

* Named constructors - this is a constructor that has a suffix or identifier tag apended. For example:
```dart
class Car{
    int model;
    String name;

    Car(int model, String name){...}

    //fromOther is the constructor name, allowing this constructor 
    //to be distinguished from the previous one
    Car.fromOther(Car car){...} 
}
```

If a constructors sole purpose is assigning the constructor args to instace variables, Dart provides syntactic sugar shorthand:
```dart
class Car{
    int model;
    String name;

    Car(int model, String name){
        this.model = model;
        this.name = name;
    }

    //the above constructor may be written as
    Car(this.model, this.name);     
}
```

During the process of creating a new instance of a class from a constructor, the superclass constructor is called. By default, an identifier-less, no -arg constructor  (i.e. the provided by default , or an explictly declared no-arg constructor with parameter set `()` or `([...])` or `({...})`, without an identifier) in the superclass is called. If such a constructor does not exist in the superclass or if it is desired to instead call some other superclass constructor then superclass *constructor direction* must be provided. For example:

```dart
class Person{
    int age;
    String name;

    Person(this.age, this.name);

    Person.fromOther(Person person){
        .
        .
        .
    }
}

class Student extends Person{
    int rank;

    //Student(this.rank);
    //the commented constructor is invalid as it will try to call
    //the identifier-less no-arg superclass constructor which
    //does not exist.

    Student(this.rank) : super(0, '');


}
```
In certain cases, it is useful to have constructors that delegate to other constructors (in the same class). In such cases, you can use an empty constructor body with the *this* constructor redirection as exemplified below:
```dart
class Person{
    int age;
    String name;
    bool vip;

    Person(this.age, this.name, this.vip);

    Person.Vip(this.age, this.name) : this(age, name, true);
}
```
In Dart, we can also make use of factory constructors. A factory constructor is a constructor which does not always create a new instance of a class. For example, it might return an existing instance from a cache. Such constructors require the `factory` keyword prefix (see example in Language Tour docs.).

Dart classes also support operators. Operators are class methods, with special names, that may be applied on operands. Operators are defined in classes with the `operator` keyword. This allows operator feature on the class instances. The Language Tour docs have further examples. I recommend to stick with general named methods like in java and forego object instance operator features.

Similar to Java, Dart support abstract classes. Like Java, an abstract class (i.e. one containing at least one method that is not implemented - i.e. without a body - such that the class itself may not be instantiated) requires the `abstract` keyword in the class declaration. **Unlike** Java, the abstract methods do not include an `abstract` keyword.

Unlike Java, all Dart classes have implicit interfaces(containing all instance members of the class and any interface it implements). As such, Dart does not have an `interface` keyword. All you need to do is define the interface as a class, XXX, and declare that another class, YYY, implements XXX. It is recommended that XXX be declared as abstract. The `implements` keyword allows the interface (of XXX) to be supported without inheriting any (if present) of the implementation declared in XXX. It also allows multiple interfaces to be supported. `extends` keyword inherits the implementation declared in XXX and does not allow multiple inheritance.

```
TODO| Mixins, 
```

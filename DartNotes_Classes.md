# Dart Classes and OOP Support

## Classes

In Dart, unlike Java, multiple first-level public classes may be defined in a single file and the file name in Dart does not have anything to do with the name of the contained class(es).

Classesgenerally consist of fields and methods and optionally constructors and getters/setters. Unlike Java, all instance variables generate an implicit getter. All non-final instance variables also generate an implicit setter. For example:

```dart
class Car{
    int model;
    String name;
}

void main(List<String> args){
   Car car = Car(); //or Car car = new Car();
   car.model = 1975; //uses implicit setter
}
```

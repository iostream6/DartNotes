
A library is a collection of resources, including pre-written codes, interfaces, configurations, documentation, etc, that may be shared & reused for a general purpose during software development. Libraries allow for modular code design & code reuse. Additionally, in Dart, a library is the unit of privacy.

Every Dart program is a library, even if this is not explicitly indicated by the use of the `library` directive. A Dart program consist of one or more libraries. 

Dart comes with a set of built-in libraries that provide essential functionality common to most programming use cases - e.g. the `core` library. Other Dart libraries (apart from the built-in libs) are managed and distributed using packages and the package management system referred to as `pub`.

Note that packages in Java are simply a namespacing/privacy/organization unit for classes. In Dart, a package is a construct to manage and distribute Dart libraries or programs. At a minimum, a Dart package consists of a directory containing a `pubsec.yaml` file. This file contains the packages metadata and a list of dependencies. Within the directory, zero or more libraries/programs may be created.

## Using Libraries

The *import* directive allows resources from one library to be used in another. It may take three forms:


```dart
import 'dart:xxx'; // for use with built-in libraries
```

```dart
import '*relative path*'; 
// for lib file from your own package when the library and the client file are both inside 
// of '/lib' or both outside of '/lib' folder
```

```dart
import 'package:xx-yy/zzz'; 
// for when importing from another package or from across first level folders of your own 
// package (e.g. between '/lib' and '/bin')
```

In the above, xxx-yyy would be the package name and the zzz part would be anything in an entrypoint folder (i.e. /lib', '/web', 'bin', etc folders).

```
TODO Draw tree in pp 14.
```

The import directive has some useful options:

* "as" e.g `import 'aaaa-bbbb/cccc' as  ddd`: this prefixes /namespaces the imported resources and is useful in avoiding name clashes.

* "show|hide" e.g. `import 'xxxx' show yyy`: allows selective importing:-
```dart
import 'package:lib_alpha/alpha.dart' show foo; //imports only foo
import 'package:lib_alpha/alpha.dart' hide foo; //imports all except foo
```

* "deferred"

### Conditional Import/Export

Libraries may be conditionally imported or exported based on criteria define by the Dart VM viz:

```dart
/*
 * the below will export 'src/hw-none.dart' except if dart.io library exists
 * in which case it will export 'src/hw_io.dart'. A similar construct can 
 * be used for imports
 */ 
export 'src/hw-none.dart' if (dart.library.io) 'src/hw_io.dart';

// Multiple if's may be used viz:
export 'xxxx' 
   if (<vm condition 1>) 'yyyy';
   if (<vm condition 2>) 'zzzz';
```

## Implementing Libraries

The package entry point for libraries in Dart is '/lib'. This means that all dart files placed in '/lib' and sub-folders are treated, by default, as representing separate libraries.  That is, one file = one library, by default. The only exception to this is:

* files in '/lib/src/..' and subordinate folders, which by convention are implementation code and are considered as private.

* when the '/lib/...' (ex src) files have "library", "part" and "part of" directives since these may be used to define a single library consisting of multiple files (in '/lib')

The current advice is not to use "library", "part" and "part of" directives. Instead, let each '/lib/..' file represent one library. If additional files are required to fully define the library, place these in '/lib/src/..' and export to the main library file. A sketch is provided below:

**package sphinx:**
/lib/task.dart
/lib/utils.dart

/lib/src/task/windows.dart
/lib/src/task/linux.dart

```dart
//task.dart
  export 'src/task/windows.dart';
  export 'src/task/linux.dart';
  .
  .
  .
```

**package my_app:**
/bin/main.dart


```dart
//main.dart
  import 'package:sphinx/task.dart';//use the multi-file 'task' lib from sphinx package
  import 'package:sphinx/utils.dart';//use the single file 'utils' lib from sphinx package
  .
  .
  .
```
**Note:** To include any library level documentation, you must use the "library" directive in the main library file.

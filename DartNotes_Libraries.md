
A library is a collection of resources, including pre-written codes, interfaces, configurations, documentation, etc, that may be shared & reused for a general purpose during software development. Libraries allow for modular code design & code reuse. Additionally, in Dart, a library is the unit of privacy.

Every Dart program is a library, even if this is not explicitly indicated by the use of the `library` directive. A Dart program consist of one or more libraries. 

Dart comes with a set of built-in libraries that provide essential functionality common to most programming use cases - e.g. the `core` library. Other Dart libraries (appart from the built-in libs) are managed and distributed using packages and the package management system referred to as `pub`.

Note that packages in Java are simply a namespacing/privacy/organization unit for classes. In Dart, a package is a construct to manage and distribute Dart libraries or programs. At a minimum, a Dart package consists of a directory containing a `pubsec.yaml` file. This file contains the packages metadata and a list of dependencies. Within the directory, zero or more libraries/programs may be created.

## Using Libraries

The *import* directive allows resources from one library to be used in another. It may take three forms:

* import 'dart:xxx'; // for use with built-in libraries




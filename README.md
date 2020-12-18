# :lizard: calang

### A simple and familiar C-like programming language optmized for happiness 

##### Strongly inspired by [wren](https://wren.io), some inspiration from [vlang](https://vlang.io/), [ziglang](https://ziglang.io)

###### Drivers

- C-like syntax
- Strong modularization, packaging and versioning story. Tightly integrated into the language
- Compiled to bytecode, but compilation step may be implicit. 
- Binary package distribution is a first-class citzen
- Tooling should evolve hand in hand with the compiler/runtime. Tooling should not be an after-thought. The language should be as simple as the tooling can support.

###### Non-goals

- Native compilation targets
- Being simple to embed the runtime is a "nice to have", but not a driver
- Being a pure functional language

##### Initial ideas

The ideas listed below are classified in a staged approach, where stage 0 would be the MVP. 

###### 1 - Minimal program 

**stage** :zero:
```c#
require std;
namespace foo/bar;

import {system};

fn main() {
   Console.println("Hello, world");
}

```

- `require` specifies which modules this program *depends on*. More on _modules_ on (add reference). A requirement to module `std` is implict and may be omitted
- `namespace` defines the fully qualified name for every function/type defined. It has no relation to directory structure
- `import` defines which _namespaces_ are imported to the compilation context. See _namespaces_ (add reference). The namespace `system` is always imported implicitly. This statement may be omitted
- A function named `main` is the entry point of the application
- `Console` is a class defined in the namespace `system`
- `println` is static method of the class `Console`

When the implicit definitions are removed the minimal program is:

```c#
namespace foo/bar;

fn main() {
   Console.println("Hello, world");
}

```

**stage** :one:

- If namespace definition is omitted then all classes and functions defined in the class belong to the _root_ namespace.
- It is possible to import all the static definitions of a class. All its static members can be referred without the class name
- The compiler will implicitly wrap all top-level statements into a generated `main` method

```c#
import {system/Console}
println("Hello, world");
```

###### 2 - Namespaces

**stage** :zero:

- Functions and classes belong to a `namespace`. Namespaces exist to avoid name clashes and have no relation to directory structure of source files
- Namespaces are declared by the `namespace` keyword and its parts are separated by `/`
- Classes and functions may be referred by its fully qualified names

```c#
//file: utilities.ca
namespace utilities/console;

fn salute() {
   Console.println("Hello, world");
}
```

```c#
//file: animals.ca
namespace animals/reptlies;

class Lizard() {}
```

```c#
//file: main.ca
import {utilities/console}

salute();
const lizard = animals/reptiles/Lizard();

```

**stage** :one:

- More than one namespace may be defined in the same file

```c#
namespace utilities/console;

fn salute() {
   Console.println("Hello, world");
}

namespace core/messages;

class Message {}
```

###### 3 - Modules


**stage** :zero:

- The minimal compilation unit are *modules*. The compiler is aware of modules and versions
- Dependencies between modules are defined by the keywork `require`
- The `requires` keyword is followed by a _module definition_. A module definition is part of the langugage syntax and it is formed as `<publisher_id>:<module_id>:<version>`
- The version definition follows _SemVer_ and always required. No wildcards are allowed.
- The module `std` is the standard module and does not require `publisher_id` and `version`. The module `std` is always implicitly required
- The `requires` keyword can appear in *any* source file and must be the first statement. The required module is available for all the source files of that same module

```c#
requires acme:motd:1.2.1

import {acme/messages, system/Console};

println(mtod());
```

- If more the one source file requires the same module, a compile error will be generated. 

```
main.ca:7:8: error: The module acme:motd is required more the once

      | main.ca:7:4
    7 |     requires acme:motd:1.2.1
      |              ~~~~~~~~~~~~~~~~
    8 |     import {acme/messages, system/Console};
    

      | messages.ca:3:4
    3 |     requires acme:motd:1.3.0
      |              ~~~~~~~~~~~~~~~
    4 |     import {acme/messages};    
    
```

- Specifying dependencies in the source files is a good feature for single-file programs. For projects with multiple files it is recommended to utilize the special source file `module.ca` that accepts only certain keywords (`module`, `requires`)
- The keywork `module` specifies a _module definition_ for the project. 

```c#
//file: module.ca
module acme:core:1.0.0

requires acme:motd:1.2.1
requires megacorp:utilities:0.4.1
```


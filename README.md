# :lizard: calang

### A simple and familiar C-like programming language optmized for happiness 

##### Strongly inspired by [wren](https://wren.io), some inspiration from [vlang](https://vlang.io/), [ziglang](https://ziglang.io)

##### Drivers

- C-like syntax
- Strong modularization, packaging and versioning story. Tightly integrated into the language
- Compiled to bytecode, but compilation step may be implicit. 
- Binary package distribution is a first-class citzen
- Tooling should evolve hand in hand with the compiler/runtime. Tooling should not be an after-thought. The language should be as simple as the tooling can support.

##### Non-goals

- Native compilation targets
- Being simple to embed the runtime is a "nice to have", but not a driver
- Being a pure functional language

#### Initial ideas

The ideas listed below are classified in a staged approach, where stage 0 would be the MVP. 

##### 1 - Minimal program 

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

##### 2 - Namespaces

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

##### 3 - Modules


**stage** :zero:

- The minimal compilation unit are *modules*. The compiler is aware of modules and versions
- Modules are a collection of classes and functions compiled to bytecodes. They are grouped in a single binary file (_cam_ files) that are loaded by the calang VM. Calang modules are not a ZIP files.
- The bytecodes specs should be stable and evolve in backwards compatible way. It is expected that modules are distributed in binary form. 
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

- Any program without the keywork `module` is considered an _anonymous module_. Anonymous modules cannot be compiled to CAM files and cannot be referred by other modules. They can only be executed from source. 
- Anonymous module can require other modules

```c#
//file: main.ca
// This is an anonymous module. It can be only executed from source
require acme:motd:1.2.1
Console.println(acme/messages/motd());
```

```bash
>calang main.ca
Message of the day.
```

- Even a single source file can become a module if it contains the keywork `module`. In this case it can be compile to binary distributable module. 

```c#
//file: main.ca
module megacorp:core:1.0.0

require acme:motd:1.2.1
Console.println(acme/messages/motd());
```

```bash
>calang build && ll
megacorp.core.1.0.0.cam
>calang megacorp.core.1.0.0.cam
Message of the day.
```

- The contents of a module are defined by the contents of the current directory. It defines the _compilation scope_

```bash
>tree .
.
├── core
│   ├── date.ca
│   └── utilities.ca
├── main.ca
└── README.md
```

The command `calang main.ca` will result in an implicit compilation step whose scope contains `main.ca`, `core/date.ca`, `core/utilites.ca`

The command `calang core/data.ca` will result in implicit compilation step whose scope contains `core/date.ca` and `core/utilites.ca`

The command `calang build` executed in the root directory will result in an explict  compilation whose scope contains `main.ca`, `core/date.ca`, `core/utilites.ca`. 

* If none of the files includes a `module` definition the compilation will fail :red_circle:
* If more than one file includes a `module` definition the compilation will fail :red_circle:
* If a single file in the compilation scope contains a `module` definition the compilation will succeed and the _cam_ file will be generated :green_circle:

- Even a multi-file project can be executed from source. `calang main.ca` will trigger an implicit compilation step and look for top-level statements or a `main` function in `main.ca`. 

```c#
//file: core/utilities.ca
require acme:motd:1.2.1
fn salute() { 
   Console.println(acme/messages/motd());
}
```

```c#
//file: main.ca
fn main() {
   salute(); // no need to import, function is defined in the root namespace  
}
```

```bash
>calang main.ca
Message of the day.
```

- Usually multi-file modules will make use of `module.ca` file

```bash
>tree .
.
├── core
│   ├── date.ca
│   └── utilities.ca
├── main.ca
└── module.ca
```

```c#
//file: module.ca
module megacorp:core:1.0.0

requires acme:motd:1.2.1
```

```c#
//file: core/utilities.ca
fn salute() { 
   Console.println(acme/messages/motd());
}
```

```c#
//file: main.ca
fn main() {
   salute(); // no need to import, function is defined in the root namespace  
}
```

```bash
>calang main.ca
Message of the day.
>calang build && ll
megacorp:core:1.0.0.cam
>calang megacorp:core:1.0.0.cam
Message of the day.
```






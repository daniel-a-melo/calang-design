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

###### Minimal program 

**stage** :zero:
```c#
require "std";
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


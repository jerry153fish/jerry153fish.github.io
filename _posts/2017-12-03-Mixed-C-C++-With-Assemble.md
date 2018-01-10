---
layout: post
title: Mixed C/C++ With Assemble
key: 20171203
tags: assemble c c++ gas
---

Recently, I was learning how to write a simple operation system in c++. There are many codes which mixed c/c++ withe assembles, either call c/c++ from in assemble codes or jump to assemble code piece in c/c++. This article records my finding in how to switch codes between them.

### GCC Compilation Process

There are four steps for gcc to compile a higher language into machine codes.

1. Pre-processing: includes the headers and expands the macros
2. Compilation: Compile the pre-processed code into assemble code
3. Assembley: converts the assemble code into machine code in object file format
4. Linker: link the object code with library code and produce executable file

As figure[^1] shows:

![gcc compile process](https://bharathisubramanian.files.wordpress.com/2010/03/compile.png?w=500&h=185&zoom=2)

### Name mangle

According to the GCC compilation process above, we need to understand how function name are mangled by gcc during difference - name mangling.

> Name mangling (also called name decoration) is a technique used to solve various problems caused by the need to resolve unique names for programming entities in many modern programming languages.[^2]

#### C

There is no name mangling for C language. Thus we can call C function directly in assemble. eg:

test was defined in c file

```c
void funcInC() {
    printf("test \n");
}
```

it can be directly called in assemble code

```s
.extern funcInC

# some codes

call funcInC

# other
```

![cmangle](/assets/img/mixedcands/cmangle.png)

#### C++

C++ name mangling following rules below:

* Start with _Z
* If nested add append N
* Followed by numberOfCharsInNamespace and namespace if has namespace as well same rule of classes
* Followed by parameter_names_encoded

eg:

```cpp
int func(int);
float func(float, int);
class Cfirst {
    int func(int);
    class Csecond {
        int func(int);
    };
};

namespace N {
    class CNfirst {
        int func(int);
    };
};
```

* int func(int) => _Z4funci
* float func(int) => _Z4funcfi
* int Cfirst::func(int) => _ZN6Cfirst4funcEi
* int Cfirst::Csecond::func(int) => _ZN6Cfirst7Csecond4funcEi
* int N::CNfirst::func(int) => _ZN1N7CNfirst4funcEi

![cmangle](/assets/img/mixedcands/cnamemangle2.png)


### Conclusion

As long as knowing the mangled name in object code, we can directly call c/c++ function with mangled name in the assemble code.


### Reference

[^1]: Bharathi Subramanian 2010, *GCC Compliation Process*, Blog, https://bharathisubramanian.wordpress.com/2010/03/28/gcc-compilation-process/

[^2]: Wiki 2017, *Name Mangling*, website, https://en.wikipedia.org/wiki/Name_mangling

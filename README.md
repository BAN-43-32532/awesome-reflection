# Awesome Reflection
A collection of C++26 reflection examples.

Unless otherwise specified, all examples are compiled with the latest [clang-p2996](https://github.com/bloomberg/clang-p2996/tree/p2996) implementation, with compile flags `-std=c++26 -freflection-latest -fexpansion-statements`. 

## From [C++ meetup Brno | Herb Sutter: Reflection - C++´s Decade-Defining Rocket Engine](https://youtu.be/H4NiR9xe1XA):

- [List data members with template-for](https://godbolt.org/z/WeaKzWc56)

- [Compile-time JSON parser](https://godbolt.org/z/3jzehW1bv)
  Use `#embed` with compile flag `-Wno-c23-extensions`.

- [Polymorphism code generation](https://godbolt.org/z/aP4dKfqjo)
  Pre-C++26 implementation: <https://godbolt.org/z/MEPvEMbqP>

- [Class interface code generation](https://godbolt.org/z/Y75Wr3nGe)

- [Class interface code injection](https://godbolt.org/z/7jv9aj8Mh)

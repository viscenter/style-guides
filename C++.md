Seales Research C++ Style Guide
===============================

These are some simple rules to follow during C++ development of the Volume Cartographer project.

### Naming Conventions
Consider these naming convenctions hard requirements.

* `lowerCamelCase` for local variables, member functions
* `UpperCamelCase` for:
    * Class names
    * using-declarations
    * Enums and enum class's
    * `static` functions and member functions
* `_memberVar`: lowerCamelCase'd leading underscore for member variables
* `SCREAMING_SNAKE_CASE` for any `#define`'s, `const static`, or `const` member variable
* File names should be UpperCamelCase'd and be named after the class they contain or after the functionality they contain
    * For class `UVMap` the header and source file should be named UVMap.h and UVMap.cpp, respectively
    * For a file contianing some helper functions, it should be named something like Util.h/Util.cpp. Use your best judgement here

### Best Practices
Consider these strongly-encouraged suggestions. There will be times where these guidelines won't apply, and that's okay. Use your best judgement.

* The STL. Use it. It has tons of really helpful and useful functionality that many smart people have spent a long time on making really fast.
    * If you need an algorithm, check first if it already exists in `<algorithm>`.
    * `std::vector`: unless you are sure you only need a fixed-size array, you should be using this
    * `std::array`: if you need a fixed-size array, use this one instead of C-style arrays. This allows you to more easily interface with the STL.
* Pointers
    * You basically shouldn't have to call `new`/`delete` in the code we're writing.
    * Furthermore, you _probably_ won't need to use raw pointers in any of our code with the exception of when writing Qt5 GUI code
    * If you _do_ need a pointer, use `std::unique_ptr`/`std::shared_ptr` instead. There are helpful functions in `<memory>` called `std::make_unique` and `std::make_shared` that can construct them for you directly.
* `auto`
    * Use auto where it makes sense.
    * If the type is really, really long, you should definitely use it, e.g. `std::vector<int>::const_iterator it = std::begin(myVec)` can be more succinctly written as `auto it = std::begin(myVec)`.
    * If not, e.g. `double f = 1.0`, prefer to use the named type directly.
* Range-based for-loops
    * Use these when iterating over a collection, e.g. a `std::vector`, `std::map`, `volcart::PointSet`, etc.
    * Unfortunately, some of the libraries we use (ITK/VTK) decided to not make their collections work with this, so you have to use the old-style loops with those collections.
* `enum` vs `enum class`:
    * Prefer `enum class`. It gives you more compile time safety than a regular `enum`.
* `struct` vs `class`:
    * If you make a class and all the member variables are `public`, use a `struct` instead.
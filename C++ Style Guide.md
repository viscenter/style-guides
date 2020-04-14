# DRI C++ Style Guide

The following set of rules are meant to guide developers who wish to contribute to any of the DRI's C++ projects.
The intent of these rules is to enable easy contribution from a wide base of developers.
Extensive C++ experience should not be a prerequisite for working on our projects.
If any of these rules are ambiguous, we ask that you favor coding practices that will be easily learned by new developers that are unfamiliar with the project.

## Build System
We use the CMake build system for all of our C++ projects.
A future style guide will explore CMake project organization.

## Naming Conventions
The following naming conventions should be followed before code is merged into
the primary branch of a project.

### Files
* Header files should have the extension `.hpp` and source files should have the extension `.cpp`.
* All header and source files should be written in `UpperCamelCase`. Files that declare and define a class should be named after the class.

### Code
* Use `UpperCamelCase` for new type declarations such as `class`, `enum`, and type-aliases. Use `lowerCamelCase` for local variables.
```c++
// Class
class Foo;
// enum
enum class Bar { None, One, Two };
// type-alias
using Integer = int;
// local variable
Integer i = 10;
```

* Use `UpperCamelCase` for all static functions, both member and non-member. Use `lowerCamelCase` for public instance member functions. Public member variables are not allowed. Use `lowerCamelCaseTrailingUnderscore_` for private member variables. Use `lower_case_trailing_underscore_` for private instance member functions:
```c++
// static function
int RandomInteger();
// class rules
class Foo {
public:
    // static member function
    static Foo CreateFoo();
    // public instance member function
    void getBar();
private:
    // private member variable
    int private_bar_;
    // private instance member function
    void init_bar_();
};
```

* Use `SCREAMING_SNAKE_CASE` for `const`, `static const`, or `constexpr` variables and functions.

```c++
const float GLOBAL_FOO = 1.0
static const float LOCAL_FOO = 2.0;
constexpr float FOO_RATIO = GLOBAL_FOO / LOCAL_FOO;

class Foo {
public:
  // Shared across instances
  static float CLASS_VARIABLE{1.0};
  // Shared across instances. Value cannot be changed.
  static const float CLASS_CONSTANT{2.0};
};
```

## Organization
### Header Includes
Includes should be organized into four categories, each separated by a
single blank line:

1. **(.cpp only)** The include that this file implements, wrapped in double quotation marks: `""`
2. Standard library and system includes, wrapped in angle braces: `<>`
3. Third-party library includes, wrapped in angle braces: `<>`
4. Project includes, wrapped in double quotation marks: `""`

Within each of these categories, items should be sorted in descending
lexicographic order.

```c++
// This file's include
#include "vc/core/Foo.hpp"

// System and stdlib includes
#include <iostream>
#include <memory>
#include <vector>

// 3rd party includes
#include <itkPoint.h>
#include <opencv2/core.hpp>
#include <vtkPoint.h>

// Project includes
#include "vc/core/types/Volume.hpp"
#include "vc/meshing/OrderedResampling.hpp"

// Implementation
Foo::Foo() {};
```

### Include It Where You Use It (IIWYUI)
Only include header files in those places where those headers are actually used.
If a header could be included in both the `.hpp` and `.cpp` for a class,
only include it in the `.hpp` file.
Consider the following two files which define the interface and implementation
for class `Foo`:

```c++
// Foo.hpp
#pragma once

#include <vector>

class Foo {
public:
  std::vector<int> bar();
};
```

```c++
// Foo.cpp
#include "Foo.hpp"

#include <random>

std::vector<int> Foo::bar() {
  std::vector<int> results;

  // Setup random number generator
  std::default_random_engine generator;
  std::uniform_int_distribution<int> distribution(1,6);

  // Roll dice 6 times
  for(int i = 0; i < 6; i++) {
    results.emplace_back(distribution(generator));
  }
  return results;
}
```

Because `Foo::bar()` returns a `std::vector`, we make sure to include the standard library `<vector>` in our header file.
However, the implementation of `Foo::bar()` also makes use of classes defined in the
standard library `<random>`.
Since these do not affect the interface for `Foo`, however, we only include `<random>` in `Foo.cpp`.
This pattern of Include It Where You Use It (IIWYUI) reduces the compilation burden on code which includes `Foo.hpp` because the compiler does not have to include `<random>` in order to make use of the class interface.

## Modern C++ Best Practices
In all of our C++ projects, we strive to follow and maintain "Modern C++" coding practices.
This means that, as much as possible, we favor the use of stable, widely adopted functionality from the standard library before we rely upon our own algorithms or those provided by third-party libraries.
As time progresses, we strive to periodically and consistently update our existing code to maintain compatibility with these evolving standards.
At the time of this writing, all C++ projects should target C++11 or C++14.

The following best practices highlight a few of the common expectations we have for our C++ codebases.

### The Standard Template Library (STL)
The STL provides a number of useful data structures and algorithms out-of-the-box.
Contributors should familiarize themselves with the features of STL and make frequent use of them.
The following libraries are of particular note as they contain the most commonly used features in day-to-day programming:

  * [Containers library](https://en.cppreference.com/w/cpp/container): Provides implementations for many common data structures, including linked lists (`std::list`, `std::forward_list`), hashed sets (`std::set`, `std::map`), and stacks (`std::stack`).
  * [Algorithms library](https://en.cppreference.com/w/cpp/algorithm): Provides implementations of algorithms which operate on a collection of items, including search (`std::find`, `std::any_of`), sort (`std::sort`, `std::partial_sort`), and numeric operations (`std::accumulate`, `std::inner_product`).

### Arrays
The best choice for array and list containers is typically `std::vector`.
It provides all of the functionality one would expect from a C-style array, as well as range-based operations provided by the STL.

```c++
#include <algorithm>
#include <iostream>
#include <vector>

std::vector<int> numbers{0, 1, 2, 3};

// Access via [] operator
for(int i = 0; i < numbers.size(); i++) {
  std::cout << numbers[i] << " " << std::endl;
}

// STL transform + lambda function
int f = 2;
std::transform(numbers.begin(), numbers.end(), numbers.begin(), [f](int i) { return i * f; });

// Range-based loop
for(const auto& i : numbers) {
  std::cout << i << numbers << " " << std::endl;
}
```
Output:
```shell
0 1 2 3
0 2 4 6
```

In the circumstance that you need a statically allocated array, the standard libraries also provide `std::array`:

```c++
#include <array>
#include <fstream>

// Static buffer
std::array<char_t, 32> buffer;

// Open file and read portion into buffer
// This is a toy example. std::ifstream already provides its own internal stream buffer.
std::ifstream ifs("foo.txt", std::ifstream::in | std::ifstream::binary);
ifs.read(buffer.data(), buffer.size());
```

### Pointers
The use of `new`, `delete`, and raw pointers is not allowed in our projects.
Instead, use the smart pointers provided by the standard library: `std::shared_ptr`, `std::unique_ptr`, and (less likely) `std::weak_ptr`.
These pointers provide automatic lifetime management of objects.
They can be initialized using the templated `std::make_[shared|unique|weak]` functions:

```c++
#include <memory>

class Foo {
public:
  int x() { return 10; }
};

// Construct a new pointer
std::shared_ptr<Foo> bar = std::make_shared<Foo>();

// Dereference works as expected
std::cout << bar->x() << std::endl;

// Check if pointer doesn't point to anything
std::shared_ptr<Foo> bad;
if(bad == nullptr) {
  std::cout << "Null pointer!" << std::endl;
}
```

One exception to this rule is if you are working with the Qt libraries.
Qt provides its own version of smart pointers/automatic memory management that should not be mixed with the classes provided by the standard library.
Refer to [the Qt documentation](https://wiki.qt.io/Smart_Pointers) for more details.

### For loops
Range-based for loops should be used when the index of a container item is only used to access the item.

```c++
// Range-based for loop
std::vector<int> values{0, 1, 2, 3};
for(int& value : values) {
  value = value * 2;
}
```

In many circumstances, a for loop is used to simply read or modify the items in a container.
The standard for loop requires the programmer to track and maintain an index variable (e.g. `i`), and be careful that this variable does not exceed the bounds of the container:

```c++
// Standard for loop
std::vector<int> values{0, 1, 2, 3};
for(size_t i = 0; i < values.size(); i++) {
  values[i] = values[i] * 2;
}
```

Most of the containers provided by the standard library also enable item access through random access iterators.
Iterators are pointers to locations within the container.
When an iterator is incremented or decremented, it points to the next or previous location in the container.
Iterators can be used when the objects in the container need to be accessed without regard to the object's specific position.
This provides safer access (i.e. implicitly checks bounds) to the items in the container and somewhat improves understanding of code intent:

```c++
// Basic STL for loop
// IntList::end() is one position beyond the last item in the container
using IntList = std::vector<int>;
IntList values{0, 1, 2, 3};
for(IntList::iterator item = values.begin(); item != values.end(); item++) {
  *it = *it * 2;
}
```

The convenience of this approach is that it provides a single, consistent access method for all STL containers, including those containers that don't support integer indices (e.g. `std::unordered_map`).
The negative is that the syntax is bulky and requires a potentially confusing pointer-like access to container items.

Range-based for loops greatly improve upon both of these methods and allow easier control over the type of element access required:

```c++
std::vector<int> values{0, 1, 2, 3};

// Range-based: Access via reference
for(int& value : values) {
  value = value * 2;
}

// Range-based: Access via const reference (object cannot be modified)
for(const int& value : values) {
  std::cout << value << std::endl;
}

// Range-based: Access a copy (object in container is not modified)
for(int value : values) {
  value = value * 2;
}
```

Unfortunately, some of the libraries we use (ITK, VTK, OpenCV) do not work with the range-based syntax.
Sometimes, as with ITK and VTK, there are equivalent iterator access methods that can be used instead.
In these circumstances, these should be preferred to the standard for loop.
If no such method is provided, or if the iterator interface would be prove extremely confusing in practice (e.g. OpenCV), use of the standard for loop is allowed.

### Brace initialization
Where possible, use [brace/list initialization](https://en.cppreference.com/w/cpp/language/list_initialization) when constructing objects:

```c++
int i{0};
std::vector<int> values{0, 1, 2, 3};

class Foo {
public:
  // Default constructor
  Foo() = default;
  // Default value used by default constructor
  int member{1};

  // Initialization value passed to constructor
  explicit Foo(int i) : member{i} {
    // other initialization code here
  };

  // Construct return type from list init
  std::vector<int> bar() {
    return {0, 1, 2, 3};
  }
}

Foo f1;
Foo f2{2}
std::cout << i << " " << f1.member << " " << f2.member << std::endl;
```

Output:
```shell
0 1 2
```

### Use `pragma once` for include guards
Use `#pragma once` in place of the traditional include guard `#ifndef _CLASS_HPP_`.
This performs file-level include guarding:

```c++
#pragma once

class Foo {};

// Nothing required at the end of the file
```

### Typing Rules
#### Fixed width integer types
When using integers where the number of bits is important, use the [fixed width integer](https://en.cppreference.com/w/cpp/types/integer) types provided by C++11 rather than the [fundamental types](https://en.cppreference.com/w/cpp/language/types):

```c++
// Signed integer, implementation-defined size, at least 32-bits
int fundamental;
// Signed integer, exact size, 32-bits
std::int32_t fixed;
```

Additionally, use `std::size_t` when storing or referring to the value of a container's size:

```c++
std::vector<int> values{0, 1, 2, 3};
std::size_t num_objects = values.size();

for(std::size_t i = 0; i < values.size(); i++) {
  ...
}
```

#### Type aliases
Do not use `#typedef` for [type aliases](https://en.cppreference.com/w/cpp/language/type_alias).
Instead, use the `using` keyword to make code more readable.
Type aliases should follow the naming convention used for classes.
This can be particularly helpful when nesting templated types:

```c++
using Pixel8UC1 = std::uint8_t;
using Row8UC1 = std::vector<Pixel8UC1>;
using Image8UC1 = std::vector<Row8UC1>;

// These are the same type
std::vector<std::vector<std::uint8_t>> img1;
Image8UC1 img2;
```

#### `auto` type specifier
The `auto` type specifier should be used on the left-hand side of assignment operations when the type of the r-value is unambiguous to the programmer.
It is also useful when writing concise range-based for loops:

```c++
// double provided, but is r-value intended to be a float?
auto f1 = 1.0;
// Prefer explicit typing here in order to avoid bugs
float f2 = 1.0;

// Function with a long, but well-defined return type
using Pixel = std::uint8_t;
std::vector<std::vector<Pixel>> LoadImage(std::string path) {
  ...
}
// Use auto here to simplify!
auto img = LoadImage(path);

// Range-based for loop
// No modifications to this code if the fundamental type of Pixel changes
for(const auto& row : img) {
  for(const auto& pixel : row) {
    std::cout << pixel << std::endl;
  }
}
```

#### `enum` vs. `enum class`
Prefer `enum class` over `enum`.
It provides more compile time safety than a regular `enum`.
Enumerated types should follow the naming convention used for classes.
If you do not require specific values for your enumerations, let the compiler
select them for you by omitting the assignment in the declaration.
If your enumerations are a list of sequential integers, it is sufficient to
provide only the value for the first item:

```c++
// Value of enumerated type not used
enum class Cardinal { North, South, East, West };

// Value of enumerated type is used
enum class Colors { Red = 0, Green, Blue, Cyan, Magenta, Yellow, White, Black};

// Cast to int for type safety
auto value = static_cast<int>(Colors::Black);
std::cout << "Black:" << value << std::endl;
```

Output:
```shell
Black: 7
```

#### `struct` vs. `class`
If you make a class and all the member variables are `public`, use a `struct` instead.

### Standard C Libraries
When including libraries provided by standard C, use the `<cxxx>` library names [provided by C++](https://en.cppreference.com/w/cpp/header) instead of their `xxx.h` equivalents:

```c++
#include <cmath> // instead of math.h
#include <csignal> // instead of signal.h
#include <ctime> // instead of time.h
```

If you are including one of these headers, first do research to find out if what you are trying to do is better performed with one of the newer C++ libraries.
For example, `std::rand` (provided by `<cstdlib>`), is deprecated in favor of the functionality provided by the `<random>` library.

### Resources
C++ is a very complex language.
There are tons of resources out there for C++ styles.
Some are good, some are not so good.
This is a collection of the ones that the C++ community has found to be pretty good (though some may be opinionated).
They are ordered roughly in the order of importance and usefulness.
* [The C++ Core Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines). These guidelines are assembled and actively worked on by the community. The main drivers are Bjarne Stroustrup (creator of C++) and Herb Sutter, a C++ expert from Microsoft. Though long, they are very in-depth and contain numerous examples of what to do, as well as what to avoid.
* [cppreference.com](https://en.cppreference.com/w/) A useful and well organized breakdown of the C++ standard and standard libraries.
Some say that there are numerous inaccuracies in this website.
We have found that it is accurate enough for most purposes.
* [The Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
Less good than the first link, it still contains some useful suggestions.

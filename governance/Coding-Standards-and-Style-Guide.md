# O3DE C++ Coding Standards <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [Introduction](#introduction)
  - [How to use this guide](#how-to-use-this-guide)
  - [Some choices are arbitrary, but still matter](#some-choices-are-arbitrary-but-still-matter)
  - [Some things are not covered](#some-things-are-not-covered)
- [Compiler compatibility](#compiler-compatibility)
- [Formatting](#formatting)
  - [Use four spaces for indentation](#use-four-spaces-for-indentation)
  - [Apply indentation in a consistent manner](#apply-indentation-in-a-consistent-manner)
  - [Indenting preprocessor statements](#indenting-preprocessor-statements)
  - [Comments after preprocessor statements](#comments-after-preprocessor-statements)
  - [Positioning of curly braces](#positioning-of-curly-braces)
  - [Use of curly braces](#use-of-curly-braces)
  - [Single statement per line](#single-statement-per-line)
  - [Make lines reasonably long](#make-lines-reasonably-long)
  - [Placement of pointer and address operators in variable declarations](#placement-of-pointer-and-address-operators-in-variable-declarations)
  - [Naming](#naming)
  - [In General](#in-general)
  - [Class Names](#class-names)
  - [File names](#file-names)
  - [Type names](#type-names)
  - [Variable names](#variable-names)
  - [Constant names](#constant-names)
  - [Function names](#function-names)
  - [Namespace names](#namespace-names)
  - [Enums](#enums)
  - [Macro names](#macro-names)
  - [Abbreviations](#abbreviations)
  - [Out parameter names](#out-parameter-names)
  - [Exceptions to naming rules](#exceptions-to-naming-rules)
- [Header Files](#header-files)
  - [Include guards](#include-guards)
  - [Include paths](#include-paths)
  - [Use forward declarations to minimize header file dependencies](#use-forward-declarations-to-minimize-header-file-dependencies)
  - [Function inlining](#function-inlining)
  - [Minimize code in headers](#minimize-code-in-headers)
  - [Including headers](#including-headers)
- [Classes](#classes)
  - [Default constructors](#default-constructors)
  - [Member initialization](#member-initialization)
  - [Declaration order](#declaration-order)
  - [Constness](#constness)
  - [Override](#override)
  - [Final](#final)
- [Scoping](#scoping)
  - [Namespaces](#namespaces)
  - [Local variables](#local-variables)
  - [Global variables](#global-variables)
  - [Code](#code)
- [File Structure/Contents](#file-structurecontents)
  - [Copyright Header Requirements](#copyright-header-requirements)
  - [API definitions](#api-definitions)
- [Miscellaneous](#miscellaneous)
  - [Use `AZ` or `AZStd`](#use-az-or-azstd)
  - [Use of `auto`](#use-of-auto)
  - [Use of nullptr](#use-of-nullptr)
  - [C++ Exceptions](#c-exceptions)
  - [RTTI](#rtti)
  - [Casting](#casting)
  - [Use of const](#use-of-const)
  - [Don’t leave disabled code in the source](#dont-leave-disabled-code-in-the-source)
  - [Visual Studio project filter hierarchy should reflect the actual disk folder hierarchy](#visual-studio-project-filter-hierarchy-should-reflect-the-actual-disk-folder-hierarchy)
- [Documentation](#documentation)
  - [Self-documenting code is better than documenting the code yourself](#self-documenting-code-is-better-than-documenting-the-code-yourself)
  - [Use full sentences and avoid abbreviations](#use-full-sentences-and-avoid-abbreviations)
  - [Leave your feelings out of it](#leave-your-feelings-out-of-it)
  - [Use non-gendered pronouns](#use-non-gendered-pronouns)
  - [Explaining "why" is often more valuable than "what"](#explaining-why-is-often-more-valuable-than-what)
  - [Private members need love too](#private-members-need-love-too)
  - [Redundant documentation is redundant](#redundant-documentation-is-redundant)
  - [TODO comments](#todo-comments)

> This style guide does not, and possibly cannot, cover every possible coding situation. If you have something that you think should, or should not, be in the standard please [bring it up with sig-core in the discord channel for discussion](https://discord.gg/fZ7CDwDxrh). It's up to everyone to make sure that this document remains accurate and useful.

### Existing Code

When modifying existing code, or adding new code to an existing module or project, the [rule of thumb](https://en.wikipedia.org/wiki/Rule_of_thumb) is to use only those parts of the coding standard that will not make the new/modified code inconsistent with the rest of the existing file/module/project.

## Introduction

> "The best thing about standards is that there are so many to choose from."

A common style is good for everyone because consistency makes code easier to read, maintain and discuss. Following a standard such as this will result in improved collaboration and higher quality code, reflecting our belief in the core value "Insist on the Highest Standards".

### How to use this guide

The document is divided into sections based on high-level C++ language features, with sub-sections to discuss more specific syntax and style issues. Statements marked  **_Required_** should always be adhered to. Statements marked  **_Recommended_** should be adhered to unless doing so would be detrimental to the code for some technical reason (i.e. beyond personal preference). Statements marked **Reason** exist for some entries to clarify intent and explain contentious or non-obvious decisions.

In common with many standards documents, e.g. ISO C++, all inline examples are provided for illustrative and informational purposes, and are not considered normative. This means that in any situation where an example appears to contradict the written standard or is otherwise ambiguous, the text of the standard always take precedence. Readers should [bring any issues up with the sig-core group in Discord](https://discord.gg/fZ7CDwDxrh).

### Some choices are arbitrary, but still matter

Some of the style choices described in this document are motivated by safety or performance or features, while others are arbitrarily disambiguating choices; sometimes the only benefit to picking one particular style over another is code consistency.

### Some things are not covered

If you encounter something that is not covered you will need to use your best judgment; a safe approach is generally to do as the existing code does. For further clarification you can always [bring it up with with sig-core in the Discord channel](https://discord.gg/fZ7CDwDxrh).

## Compiler compatibility

In an ideal world, specifying a language standard would mean that compliant code would work as expected with any compiler and on any platform that implements that standard. In some cases compilers may be behind the standard and in other cases the implementation may differ from the standard in a way that affects cross-platform portability. This section of the document attempts to deal with the practical implementations of this.

**Required**: Use the C++17 standard where possible.

(This may be impossible for projects or modules with external dependencies that rely on an older standard or toolchain)

**Required**: Use only those features of C++17 that are commonly supported by all of these compiler revisions:

- Microsoft Visual Studio 2019 16.9.2 and above
- MSVC Build Tools v14.29
- Clang 6.0.0 and above

**Required**: Your code must compile without errors and warnings on all of the compilers listed above to be considered compliant with this coding standards document.

For reference purposes C++17 feature support information can be found here:

- MSVC - [https://docs.microsoft.com/en-us/cpp/overview/visual-cpp-language-conformance?view=vs-2017](https://docs.microsoft.com/en-us/cpp/overview/visual-cpp-language-conformance?view=vs-2017)
- Clang - [http://clang.llvm.org/cxx_status.html](http://clang.llvm.org/cxx_status.html)

## Formatting

Rather than specify detailed formatting guidelines in this document we are using a code formatting tool (`clang-format`) that converts code to the desired formatting. See the `.clang-format` file at the root of the repository.

### Use four spaces for indentation

**Required**: Indent with 4 spaces, do not use tabs.

**Reason**: There is no clearly better solution when it comes to tabs vs. spaces, there is only zeal. This topic has generated countless hours of debate – The choice of spaces over tabs is made to ensure consistency, i.e. to address the most important goal of the Coding Standard.

`clang-format`: Will automatically convert tabs to spaces.

### Apply indentation in a consistent manner

**Required**: Files should start without any indentation – only add indentation if the standard indicates it is needed.

- Use a single additional level of indentation for each nested block of code.
- Note: Nested blocks are usually started by entering a new scope, i.e. whenever you would have used a brace, but there are other situations (see examples below).
- Indent all lines of a block by the same amount (see "Make lines reasonably long" for advice on continued lines).

`clang-format`: Will automatically format to these guidelines.

```c++
namespace ExampleProject
{
    // One level of indentation added for namespace block
    void Example()
    {
        // One level of indentation added for the function block
        if (foo())
        {
            // One level of indentation added for the if-block
            bar()

            // Tricky ...
            switch()
            {
            case UNKNOWN:
                // Level added for the statements in the "case"
                baz();
                break;
            }
            // etc...
        }
    }
}

namespace BadExampleProject
{
// Indentation missing after namespace
void BadExample()
{
// Indentation missing...
int a = 10;

    if (foo())
    {
        bar();
            // Too much indentation added
            if (bar())
            {
            }
    }
}
}
```

### Indenting preprocessor statements

**Recommended**: Indent preprocessor statements in a similar way to regular code.

**Reason**: This is purely for ease of reading.

`clang-format`: This is not currently enforced by the `clang-format` settings but is supported - indentation of preprocessor statements is left unchanged.

**Indenting preprocessor statements examples**:

```c++
// Fairly obvious
#if defined(XXX)
    #include <Something.h>
    #define SOMETHING
#else
    #include <SomethingElse.h>
    #define SOMETHING_ELSE
#ifdef WINDOWS
    #include <WindowsSpecific.h>
    #endif
#endif

// Not very nice
#if defined(XXX)
#include <Something.h>
#define SOMETHING
#else
#include <SomethingElse.h>
#define SOMETHING_ELSE
#ifdef WINDOWS
#include <WindowsSpecific.h>
#endif
#endif

// Ok...
void foo()
{
    if (bar())
    {
        #ifdef _DEBUG
        baz();
        #else
        zab();
        #endif
    }
}

// Arguably more pleasing on the eye
void foo()
{
    if (bar())
    {
        // (Use sparingly! Best practice is to avoid conditional compilation in the first place)
        #ifdef _DEBUG
            baz(a,b,c)
        #else
            zab(x,y,z);
        #endif
    }
}
```

### Comments after preprocessor statements

**Recommended**: Use `#endif` or `#else` comments where it will aid in readability, but not in simple cases

**Reason**: This is purely for ease of reading.

**Comments after preprocessor statements examples**:

This comment after the `#endif` is not necessary:

```c++
#ifndef USE_AZ_ASSERT
#include <assert.h>
#endif //USE_AZ_ASSERT
```

These comments help readability:

```c++
} // namespace IO
} // namespace AZ

#endif // #if !defined(AZCORE_EXCLUDE_ZLIB)
#endif // #ifndef AZ_UNITY_BUILD
```

`clang-format`: Will automatically format to these guidelines.

### Positioning of curly braces

**Required**: Open braces on a new line, and flush with the outer block's indentation

`clang-format`: Will automatically format to these guidelines.

**Positioning of curly braces examples**:

```c++
void GoodFunction()
{
    if (condition)
    {
    }
}

class CorrectClass
{
};

void WrongFunction(){
    if ( condition ) {
    }
}

class IncorrectClass {
};
```

### Use of curly braces

**Required**: Always use braces for flow control statements

**Reason**: Control statements like if-else, while, for and do used without braces can lead to serious unintentional coding errors where statements execute at the wrong scope or in the wrong order (e.g. [CVE-2014-1266](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-1266)). Additionally, consider that auto-merges exist, and do not have the context to guess how to merge single statement blocks.

`clang-format`: Will automatically format to these guidelines

**Use of curly braces examples**:

```c++
// Unacceptable: Naked if/else/while/do/for etc...
if(foo)
    bar();

// Acceptable:
if(foo)
{
    bar();
}
```

### Single statement per line

**Required**: Each line should contain no more than a single statement.

**Reason**: It is easy to misread lines with multiple statements compounded together. It also helps with debugging in some tools e.g. breakpoint placement in Visual Studio is per line, not per statement.

`clang-format`: Will automatically format to these guidelines

**Single statement per line examples**:

```c++
// Unacceptable: Single line if statements
if (foo) bar();
if (foo) bar(); else baz();
if (foo) { bar(); baz(); }

// Acceptable: Expand the statement and follow the advice for braces
if (foo)
{
    bar();
}
else
{
    baz();
}

// Unacceptable: Single line case statements
switch(foo)
{
case bar: baz(); break;
}

// Acceptable:
switch(foo)
{
case bar:
    baz();
    break;
}

// Unacceptable: Single line function definition
bool BadMethod() { return false; }

// Acceptable: Method/function definition with the body on its own line (following bracing convention)
bool GoodMethod()
{
    return false;
}

// Unacceptable: Compounding unrelated assignments
x=foo(); y=bar(); z=baz();

// Acceptable:
x=foo();
y=bar();
z=baz();

// Acceptable: Chained assignments
x = y = z = foo();

// Unacceptable: Using the comma operator to get around the single statement rule
a = foo(), bar();

// Acceptable:
a = foo();
bar();
```

### Make lines reasonably long

**Required**: If the line is longer than 140 characters, consider splitting it so that each line is no longer than 140 characters.

**Reason**: It is easy to misread statements when they extend off the page by a significant amount and makes side-by-side comparisons such as in code reviews more difficult to follow.

### Placement of pointer and address operators in variable declarations

**Required**: When declaring variables of pointer or reference type, the `*` and `&` symbol should be placed on the left; i.e. grouped with the type identifier.

**Reason**: There are three schools of thought on this matter; `type& value`, `type &value`, or `type & value`. There is little difference between the styles aesthetically so the standard arbitrarily chooses `type& value` to ensure consistency – code which mixes the 2 styles is considered to look sloppy and can lead to comprehension errors.

`clang-format`: Will automatically format to these guidelines

**Placement of pointer and address operators in variable declarations examples**:

```c++
void* buffer; // Ok
void *buffer; // Not Ok

void Foo(const string& bar); // Ok
void Foo(const string &bar); // Not Ok

void Foo(const string & bar); // Not Ok
const char* GetName(); // Ok
const char *GetName(); // Not Ok
const char * GetName(); // Not Ok

// Pointer-to-pointer declarations follow the same pattern and the base rule:
const char* const* buffer; // Ok
const char* const *buffer; // Not Ok
const char * const * buffer; // Not Ok
```

`clang-format`: Will automatically format to these guidelines

### Naming

- See [CamelCase](http://en.wikipedia.org/wiki/CamelCase) for an explanation of the "Camel Case" used below.
- See [HungarianNotation](http://en.wikipedia.org/wiki/Hungarian_notation) for an explanation of "Hungarian Notation" mentioned below (Note: This is merely here for reference, our style does **not** use Hungarian).

### In General

**Required**: Do not let your names become misleading! Think carefully about what a name represents and what expectations it carries before using it. If the thing you're naming evolves out of its original usage, then strongly consider renaming it.

**Naming examples**:

```c++
Matrix m_position;
```

- Confusing: a position is usually 3 floats in 3D. Why is it a matrix? Is it actually a transform, or is it using an inappropriate storage type?

```c++
String m_age;
```

- Confusing: an age is usually a numeric value, how is this a string?

```c++
Map m_listOfNames;
```

- Confusing: the name implies that it is a list, but it is actually a map.

```c++
class Player: UIString {};
```

- Confusing: player implies at least an actor of some kind, so why is it a string?.

```c++
enum TownNames { TOWNNAME_BOSTON };
```

- Confusing: names are usually strings, but enums are numbers.

**Recommended**: Make your names verbose. That way people will not need to guess, the code will be more readable and you will need less documentation. Typing is not your enemy, copy and paste is. Don't try to shorten words too much either. `i`, `inst` and `instance` are not the same and while `inst` could be common enough for people to know you mean instance, this is not always the case. Remember that programmers from many different domains might come across your code, try not to take anything for granted.

**Example**:

```c++
int m_cnt; // <-- bad. It's not clear whether it's "count" nor what is it that it counts
int m_useCount; // <-- better. It's clearly a count, and it's clearly counting uses
```

### Class Names

**Required**: All class/struct names should be written in UpperCamelCase.

**Recommended**: We don't recommend using `C`,`T`,`I` or any other letter to prefix a class.

**Example**:

```c++
class ExampleClass {...};
```

**Reason**: When we look at the code we will be able to instantly distinguish between class names/types/namespaces and variables.

### File names

**Recommended**: Follow the same rules as the class name. As most often the files carry an actual class name.

### Type names

**Required**: Follow the rules for class naming.

**Recommended**: We do encourage adding a 'Type' at the end.

**Example**:

```c++
using MyClassType = MyClass;
```

### Variable names

**Required**: All variables including local and function parameters should be named in lowerCamelCase, unless they are considered "constants". For constants see the "Constant names" section.

- Hungarian Notation for type information is not to be used (i.e. no prefixes of `i`, `f`, `p`, `sz`, `str`, etc...)
  - Storage class prefixes are not considered Hungarian and should be used as follows:
    - Member variables, for both classes and structs, should begin with an `m_` prefix, e.g. `m_valueOfStuff`
      - Static member variables should begin with the `s_` prefix, e.g. `s_monotonicCounter`
      - Global variables should begin with a `g_` prefix, e.g. `g_dataForEveryone`
      - Global variables are discouraged and forbidden for non-POD types

**Required**: Make sure that you use and name your variables appropriately. Use verbose names that accurately reflect what the stored value represents. Avoid using abbreviations and acronyms.

### Constant names

The use of constants is necessary in many situations. It is almost always better to declare such constants as variables rather than using literals.

**Recommended**: Use variables wherever possible; they are type safe. Use #define only where absolutely necessary.

**Required**: If you use `#define` follow the rules for macro naming

Variables declared `constexpr` or `const`, and **whose value is fixed for the duration of the program** are considered "constants" and should be named in UpperCamelCase (like enum values are).

**Required**: All constants with static storage duration (i.e. class level statics and namespace scope) should be named this way.

**Optional**: Constants of other storage classes, e.g. local variables, can be named this way, otherwise the usual variable naming rules apply.

**Constant names examples**:

```c++
class MyClass : public AZ::EBusTraits
{
public:
    // class level static const - use UpperCamelCase
    static const AZ::EBusAddressPolicy AddressPolicy = AZ::EBusAddressPolicy::Single;

    // class level static constexpr - use UpperCamelCase
    static constexpr int Foo = 2;

    // class level static consts, initialized in .cpp file - use UpperCamelCase, not s_lowerCamelCase
    static const char* const Name;
};

// static const class member initialization
const char* const MyClass::Name = "MyClassName";

namespace Internal
{
    // static consts use UpperCamelCase
    static const float Pi = 3.14159265358979323846f;

    static const char* SimpleJsonTypeIdName = "$typeId";
}

void MyFunction(const char* inputName, float radius)
{
    // These automatic variables that are always the same value can optionally be named in UpperCamelCase
    constexpr float NormalizeEpsilon = 0.002f;
    const int BufferIncrement = 16;

    // Despite being declared const, radiusSquared cannot be named UpperCamelCase, it is different depending on the input arguments
    const float radiusSquared = radius * radius;
}
```

### Function names

**Required**: All functions should be written in UpperCamelCase and be as verbose as possible. Given that we require namespaces, do not prefix function and classes with the system name. Do write `GetValue()`, do not write `AZ_GetValue()`.

NOTE: This includes RPCs or public functors on a structure. These are supposed to look and act like public methods, and they should be named as such.

**Recommended**:

- If the function only modifies a member variable, we do recommend using the `GetX`/`SetX` naming pattern.
  - When one function operates on an object and another function just returns a copy of it, we recommend naming the functions differently. A `Get` prefix is recommended on the function that returns a copy.

**Function names examples**:

```c++
// These accessors are not recommended, because the identical naming can lead to subtle bugs.
void Speed(float speed);
float Speed() const;

// These accessors are recommended, because their names are not ambiguous
void SetSpeed(float speed);
float GetSpeed() const;

// The following function would be expected to mutate the object itself
void Normalize();

// This would be expected to to return a normalized version of the object, without changing the object itself
Vector GetNormalized() const;
```

### Namespace names

**Required**: All namespaces should be written in UpperCamelCase.

**Recommended**: Add the name of the namespace as a comment after the closing brace of a namespace.

**Namespace names examples**:

```c++
namespace O3DE
{
    ...
} // namespace O3DE <-- to help identify end braces, especially when you can't see the namespace opening
```

### Enums

**Required**: Enumerator names and values should both be written in UpperCamelCase.

**Recommended**:

- Prefer scoped enums over unscoped enums where possible
  - For code consistency use only `enum class` for scoped enum declarations, i.e. never `enum struct`
  - The default scoped enum type is `int`, so if using `int` you should omit it from the declaration to reduce redundancy.
  - Using scoped enums for bit field values usually results in a lot of static_casts (there is no implicit conversion to `int` for scoped enums); therefore consider using unscoped enums for bit fields as it will produce less verbose code overall.
  - As an alternative, for scoped enums it is possible to use the `AZ_DEFINE_ENUM_BITWISE_OPERATORS` macro which adds bitwise operator overloads for scoped enum types.
- For unscoped enums, you should name the values of the enum with a prefix derived from the enum name
  - Try to contain the enum declaration in an appropriate outer namespace or struct declaration (see examples)
  - Avoid adding unscoped enums to the global namespace

**Reason**: Unscoped enums can be accessed directly without including the enum name, so the prefixing reduces the chance of name clashes, and helps code assistance tools like Intellisense or VisualAssist display more useful suggestions by keeping related flags grouped. For scoped enums prefixing is not needed as you need to refer to the enum before accessing its members.

**Enum examples**:

```c++
struct MyData
{
    enum Flags
    {
        FlagWeapon   = (1 << 0),
        FlagFast     = (1 << 1),
        FlagFlying   = (1 << 2),
    };

    // Flags can be referred to in code as MyData::FlagWeapon
}

or

// In this variation a namespace is used to contain the enum
namespace WeaponFlags
{
    // An unnamed, unscoped, enum is used for a bitfield
    enum
    {
        TwoHanded = 1,
        HasAmmunition = 2,
        HasRecoil = 4,
    };

    // A type is introduced in the namespace to explicitly control the representation
    // (this may be useful in some serialization and layout contexts)

    typedef int Type;
}

or

enum class AssetStatus
{
    NotLoaded,
    Loading,
    Ready,
    Error,
};

// Used from code AssetStatus::Loading
```

### Macro names

**Required**: All macros should be written in `CAPITAL_LETTERS`. They should be prefixed with the names of the engine and/or system in which they are defined. Macros should NOT contain any occurrence of `__` (double underscore) and they should NOT begin with any `_` (underscore) character(s) as these are reserved exclusively for the compiler's use.

**Recommended**: Use macros as a last resort. Seriously! They're bug prone and difficult to debug.

**Macro name examples**:

```c++
#define AZ_ASSERT  ...
```

**Reason**: While macros can be really useful, they do have serious pitfalls.

The first one is name clashing: Macros are always in the global namespace, so you are basically reserving this name for everyone in the same compilation unit. Macros also match before any function does, and they match signatures more primitively than functions do. For an example of how this can backfire, look at the `windef.h` min and max macro issues: because these are pulled in as a consequence of using the extremely common windows.h, codebases often have to undef them to free up the use of those very common names for types that don't work with those macros, even when the replacements have the same intent. For one, there is an obvious clash between these and `std::min` and `std::max`.

Reserving a common name like "Assert" without a prefix is very bad, as you are likely to also have functions named "Assert". Such issues are hard to debug, as very often there will be no compile error.

The second big pitfall is with macro expansion and its safety. There are many documents online that discuss that issue, so please be familiar with them before deciding to use a macro in O3DE.

### Abbreviations

**Required**: Abbreviations, acronyms and names with alternative casing are treated as if they’re regular words.

**Reason**: Treating abbreviated words, company names and product names in the same uniform way as the other naming conventions means no additional rules have to be added.

**Abbreviation examples**:

| Valid                        | Invalid                      |
| ---------------------------- | ---------------------------- |
| `class FbxRequest`           | `class FBXRequest`           |
| `int emotionFxVersion`       | `int EMotionFXVersion`       |
| `int m_lodLevel`             | `int m_LODLevel`             |
| `enum JsonResult`            | `enum JSONResult`            |
| `namespace AwsNativeSdkInit` | `namespace AWSNativeSDKInit` |
| `#define UI_ELEMENT`         | `#define Ui_ELEMENT`         |

### Out parameter names

**Recommended**: When writing out parameters do not add an explicit `out` pre or post fix to the parameter name.

**Reason**: Any parameter that is non-const may change, so the act of omitting const implies it is an out or in-out parameter. The out prefix is very similar to Hungarian Notation (see [Variable Names](variable-names)) which is prohibited by the coding standards.

**Caveat**: The above advice is recommended in the common case however there are situations where providing more explicit naming would help with a disambiguation (the variable naming section states that a parameter name should contain enough information to understand the purpose of the parameter, please see below for examples).

```c++
bool SampleSplineToLinear(vector<Vector3>& points, float distancePerSegment)
// vs
bool SampleSplineToLinear(vector<Vector3>& inputOutputPoints, float distancePerSegment)
```

In this case the name inputOutputPoints does enhance the readability of the code. The additional information is encoded in the variable name and does not rely on a Hungarian style prefix.

**Related**: If the function call is part of a public API it is possible to document the out parameter using Doxygen. This practice is encouraged.

**Out parameter name examples**:

```c++
// Not idiomatic
void ApplyLateralOffset (const Vector3& playerPosition, const Vector2& offset, Vector3* outPlayerPosition);

// Idiomatic
void ApplyLateralOffset (const Vector3& playerPosition, const Vector2& offset, Vector3* offsetPlayerPosition); // Name communicates usage

bool GetVertex(size_t index, Vector3& vertex); // Trivial, no prefix necessary to clarify intent
// (though in these cases one should likely prefer AZStd::optional<Vector3> and simply return the value)

// With documentation (note the [out] doc directive)
//! Given a Player Position and an Offset, applies a 2D translation.
//! @param playerPosition Position of player to serve as a base when transforming.
//! @param offset 2D offset in the representing XZ translation to apply.
//! @param offsetPlayerPosition [out] Optional location to store the result of the translation if non-null.
void ApplyLateralOffset (const Vector3& playerPosition, const Vector2& offset, Vector3* offsetPlayerPosition);
```

### Exceptions to naming rules

In order to maintain readability, we acknowledge that we must allow for some exceptions to the naming rules.

**When implementing well established standards like the STL, we adopt their conventions**. For example when you use or extend std, follow all the usual STL naming conventions like lower case function names.

**Allowances are made when working directly with middleware.** Although we do recommend that you wrap all middleware with a more O3DE-like interface, it *is* ok to follow the middleware conventions within the code that directly interacts with it. For example, using the Havok `hk` prefix for class names that implement Havok interfaces is ok. As is naming functions inside those classes with lower case initials.

At present, we recognize no other exceptions. If you think one should be added, please raise it for discussion.

## Header Files

### Include guards

**Required**: All header files must include the directive, `#pragma once`.

**Related** : Do not add the old school `#ifndef SYSTEMNAME_FILENAME_H` form, in addition to the #pragma once for new files.  Do not however, remove them from existing files unless you are already making changes near the header.

**Reason**: Include guards avoid problems with having a single file appear multiple times in an include chain. All the compilers we care about currently support the #pragma once directive.

### Include paths

**Required**: All include file paths must use forward slashes e.g. `#include <AzCore/std/containers/unordered_map.h>`

**Reason**: Backslashes for paths are only supported on Windows and will causing compilation errors on other platforms.

### Use forward declarations to minimize header file dependencies

**Required**: Compile times are always a problem, so we require people put in effort to minimize their include chains. The closer we get to the core layer, the more important this is, but no header file should underestimate its contribution.

You should forward declare anything that isn't directly required in a header. That is to say you should avoid all #include directives in header file X, except for those that would prevent a CPP file that only includes header X from compiling.

You should use the PIMPL idiom where possible. ([http://www.boost.org/doc/libs/1_49_0/libs/serialization/doc/pimpl.html](http://www.boost.org/doc/libs/1_49_0/libs/serialization/doc/pimpl.html) )

Useful discussions on forward declaration can be found at [http://stackoverflow.com/questions/4757565/c-forward-declaration](http://stackoverflow.com/questions/4757565/c-forward-declaration) and [http://stackoverflow.com/questions/553682/when-to-use-forward-declaration](http://stackoverflow.com/questions/553682/when-to-use-forward-declaration))

Bear in mind that you can forward declare templated classes too.

### Function inlining

**Recommended**: Despite available directives, function inlining is something on which the compiler has the last word. Even if you use `__forceinline` on MSVC, the compiler can still ignore it and output a warning instead.

Do not use `__forceinline` or any other inline hinting. Use the `inline` keyword only when you absolutely must to hint the linker.

**Reason**: Inlining must be left to the compiler. In all experiments without our codebase, force-inlined code was demonstrably more instructions and equal or worse performance. Force inlining reduces the tools the compiler can use to reason about the code and optimize it.

**Function inlining examples**:

```c++
// In these trivial cases the compiler will output the same code
// so don't worry about inlining such functions.
int GetSpeed() const
{
    return m_speed;
}

inline int GetMass() const
{
    return m_mass;
}

// Please do not do this
__forceinline int GetMass() const
{
    return m_mass;
}
```

### Minimize code in headers

**Recommended**: Along with inlining above, be careful with implicit inlining (functions defined inside class declarations). Also, please restrict implementation of functions in headers to trivial getters and setters and template functions.

**Reason**: We have a large code base. Minimizing compilation overhead and maximizing iteration time are priorities for us. If you have to change a header to iterate on your implementation, your iteration times will suffer. Further, if the compiler has to deal with your code every time it ingests your header, you've added a little bit to everyone's compile time. Multiply that across a large team, and it's death by mosquito.

### Including headers

**Required**: The following syntax should be used when including header files:

```c++
#include <Package/SubdirectoryChain/Header.h>
```

**Reason**: This rule helps disambiguate files from different packages that have the same name. `<Object.h>` might appear relatively often, but `<AZRender/Object.h>` is far less likely to.

Notice the use of angle brackets, and the inclusion of the package name. For this schema to work, the package's include directory will have to have an extra subdirectory with the name of the package in it. The compiler will then be told to include the package include directory as an additional location to search for headers from.

```c++
.../PackageRoot/includes/PackageName/SomeFile.h // <- a file at this location.../PackageRoot/includes // <- is part of a package given to the compiler this way#include <PackageName/SomeFile.h> // <- and included with this statement
```

**Including headers examples**:

```c++
// Correct
#include <Package/string/string.h>
// Incorrect
#include <string.h>
#include "string.h" 
```

## Classes

### Default constructors

**Requirement**: You must define a default constructor if your class defines member variables and has no other constructors. Prefer to use the = default; over {} default constructor. Unless you have a very specifically targeted optimization, always initialize all the variables to a known state (even if it's invalid).

### Member initialization

**Recommended**: Prefer to specify default values for class members at the declaration site in the header (C++11 style), rather than in constructor initializer lists or in the bodies of constructors. This way, the value only has to be specified once, has no requirement for constructor chaining, and is not susceptible to re-ordering causing warnings. If members are initialized in a constructor based on arguments or other members, use initializer lists wherever possible and ensure that members are initialized in declaration order. Warnings are enabled to enforce this.

### Declaration order

**Recommended**: Public declarations come before private declarations.  Methods should be declared before data members. When laying out data members, consider the packing size and order requirements of your structure. Put data members which are used together near each other as much as alignment allows, for cache friendliness. Always prefer fixed size integral types (uint32_t rather than unsigned, for instance). Guarantee consistent structure size and layout, unless you have a very good reason not to.

### Constness

**Recommended**: All methods that do not modify internal state should be const.

**Required**:

- All function parameters passed by pointer or reference should be marked const unless they are output parameters.
  - Do not use const for by value parameters (unless it is part of a template in which the generic treatment of a parameter leads to it being expressed as const)
    - Exception: Feel free to const parameters in the definition of a function (e.g. in the cpp file). This will be more semantically clear to someone reading the implementation later.
  - Do not mark a function as const if it is const-broken, that is if it returns a non-const pointer or reference to another object.

**Constness examples**:

```c++
// Unacceptable: Accessor member function should be const since it doesn't change the state of the object
int GetNumberOfPlayers()
{
    return m_numberOfPlayers;
}

// Acceptable
int GetNumberOfPlayers() const;

// Unacceptable: non-const reference where the parameter is not an output
float GetMagnitude(Vector3& v);

// Acceptable: const reference
float GetMagnitude(const Vector3& v);

// Unacceptable: const on by-value parameters restricts the implementation unnecessarily while doing nothing for the caller
void MyFunction(const int a, const int b);

class SomeClass;
class MyClass
{
    // Unacceptable: This is a const-broken function, so const-ing it isn't actually helping other programmers or the compiler ensure correctness
    SomeClass* GetSomeThing() const { return m_someThing; }

    SomeClass* m_someThing;
};
```

### Override

**Required**: Use the `override` keyword wherever possible

**Recommended**: Omit the keyword *virtual* when using `override` (virtual is implicit whenever `override` is used; follow the mantra "No unnecessary syntax").

**Reason**: The `override` keyword was added in C++11 so that a member function can clarify its intent to override an existing virtual member function on a base class (i.e. as opposed to declaring a new virtual function); it generates a compiler error when it is used on a member function that is not overriding something. This will help catch errors of intent when the code is initially written and also in the future should the base class be modified.

(see also: [http://en.cppreference.com/w/cpp/language/override](http://en.cppreference.com/w/cpp/language/override))

### Final

**Recommended**: Use the `final` keyword where its use can be justified.

- It is justified if overriding a class or member function would e.g. allow an invariant to be violated or it enforces consistency
  - It would not be justified if the only difference it makes is restricting client code

[http://en.cppreference.com/w/cpp/language/final](http://en.cppreference.com/w/cpp/language/final)

## Scoping

### Namespaces

**Required**: All of your code should be in at least a namespace named after the package, following the naming convention specified above.

**Required**: You may not use the `using namespace` directive in a .h file.

**Required**: You may not use the `using namespace` directive in a .cpp file at file scope.

**Reason**:  `using namespace` has the potential to cause issues when Uber files (Unity builds) are enabled as using directives will leak between file boundaries and interfere with name lookup rules.

**Recommended**: Prefer using-declarations to only bring in the types you require (ideally using fully qualified names with leading `::`)

**Optional**: `using namespace` may be used at function scope (be aware this can lead to issues with name lookup if a type in an inner namespace has the same name as one brought in with `using namespace` - use with caution).

**Optional**: Use type aliases in the `private` section of a class to reduce long type names without impacting users. Use of namespace aliases are permitted to allow partially qualified names (please see examples).

**Namespace examples**:

```c++
// .h (aside: C++17 nested namespaces for brevity)
namespace Fizz::Buzz::Snap
{
    struct Device{};
} // Fizz::Buzz::Snap

namespace Pop
{
    class User
    {
        // Type Alias
        // note: in private section - not publicly exporting this

        // but can save typing in the implementation...
        using Device = ::Fizz::Buzz::Snap::Device;

    public:
        // do not need fully qualified name
        void DoSomething(Device device);
    private:
        // do not need fully qualified name
        Device m_device;
    };

    // not needed here either
    void User::DoSomething(Device device) {}

    void PerformTask()
    {
        // using namespace directive at function scope - OK
        // all names imported (can lead to unexpected results,
        // for example if a different type named Device also
        // exists in namespace Pop)

        using namespace ::Fizz::Buzz::Snap;
        Device device;
    }

    void PerformService()
    {
        // using declaration at function scope - GOOD
        // explicit names imported

        using ::Fizz::Buzz::Snap::Device;
        Device device;

    }

    void PerformDuty()
    {
        // namespace alias - OK
        // in certain situations this may be preferable

        namespace Snap = Fizz::Buzz::Snap;
        Snap::Device device;
    }
} // namespace Pop
```

### Local variables

**Required**: Place a function's variable declarations in the narrowest possible scope, and *always* initialize variables in their declaration.

### Global variables

**Required**: Static member or global variables that are concrete class objects are completely forbidden. If you must have a global object it should be a pointer, and it must be constructed and destroyed via appropriate functions.

**Reason**: Static class objects cause hard-to-find bugs due to indeterminate order of construction and destruction. They can also circumvent the system's effort to log, track memory, or generally intercept their actions in useful ways.

### Code

**Required**: All platform specific code needs to be implemented according to the patterns defined by the PAL effort (Platform Abstraction Layer).

**Avoid adding platform specific defines to the codebase.**

## File Structure/Contents

Within Gems: [Gems System Documentation](https://o3de.org/docs/user-guide/gems/)

### Copyright Header Requirements

**Required**: For non-3rd Party source files that are apart of the O3DE project, they are required to carry the copyright notice detailed on the [Copyright Header Requirements](#every-non-3rd-party-source-file) page.
When adding a new source file or modifying any existing O3DE source file without a copyright please add the following copyright notice to the top of the file.

**Every non 3rd Party source file**:

```c++
/*
 * Copyright (c) Contributors to the Open 3D Engine Project.
 * For complete copyright and license terms please see the LICENSE at the root of this distribution.
 *
 * SPDX-License-Identifier: Apache-2.0 OR MIT
 *
 */
```

**Required**: For non-O3DE source code files outside of 3rd Party that we have made modifications to please follow the steps in the Special Cases section of the [Copyright Header Requirements]() document

### API definitions

- API headers (public headers): These should contain bus definitions and structure definitions for communicating with those buses.
- Minimize type dependencies on APIs where possible
  - Try to use pointers (even better, smart pointers)
  - Use `<cstdint>` numeric types (`uint32_t`, `uint64_t`, etc.)
  - Use only `AZStd` types for public containers, strings, and structures.

## Miscellaneous

### Use `AZ` or `AZStd`

**Required**: Always use the `AZ` or `AZStd` version of a container or `std` library class. This allows for maximum interoperability among our code, minimal dependencies, and minimal platform differences. We also have workarounds in AZStd specifically to ensure that our supported compilers and toolchains give a consistent behavior.

### Use of `auto`

This is one of the harder standards topics to address objectively, since what is considered appropriate for this keyword varies considerably across the development community. We will periodically revisit the topic to assess the positive and negative impacts of `auto` on code quality.

**Required**: When using `auto` make sure you are optimizing your code for correctness and maintenance rather than authoring. i.e. all things being equal, if using `auto` would decrease either correctness or maintainability then you should not use it in that scenario.

**Recommended**: If using `auto` would increase either correctness or maintainability then you may use it, but you are not required to do so.

**Reason**: Over the lifecycle of a product your code will be read many more times than it was written, so it is important for us to consider maintenance ahead of authoring concerns.

The following examples cover some common scenarios for illustrative purposes; for other situations some burden is placed on developers and reviewers to consider if the use-case is appropriate under these guidelines.

**`auto` examples**:

```c++
// Using auto for literal assignment is portable and safe – The compiler
// will choose the most appropriate compatible type. Type information is
// easily inferred by the reader at the location of the declaration.

const auto n = 42LL;
const auto s = L"wide"; // i.e. "wchar const * const"

// Using auto to store the result of a typed expression is safe and
// efficient. Type information is easily inferred by the reader.
auto ptr = std::make_shared<Foo>();
auto value = static_cast<int>(number);

// Using auto is the most efficient way to capture a Lambda; since each
// lambda has a compiler generated closure-type, it is impossible to
// specify the type of the lambda manually.

auto op = [](int a, int b){ return a+b; };

// Note: std::function is a library feature not a language feature, and is
// actually a much less efficient way to capture a lambda. This example
// shows an unnecessary use of std::function:

std::function<void(int)> op = [](int a){ std::cout << a; };
std::for_each(m_list.begin(), m_list.end(), op);

// It would be more efficient and less verbose with auto:
auto op = [](int a){ std::cout << a; };
std::for_each(m_list.begin(), m_list.end(), op);

// Using auto for iterator values in algorithms is often clearer because
// it is less verbose.

auto begin = m_list.begin();
auto end = m_list.end();

// Using auto for the results of a generic algorithm or container operation
// is also often much clearer
auto it = std::find(begin, end, value);

// However, in both cases above type information is hidden, so the author
// should consider carefully whether they need to explicitly type some
// later part of the expression for clarity ...

if (it != m_list.end())
{
    // This is a problem; it is not clear to the reader what the assumed
    // type of the iterator is because it was never stated
    it->CallSomeMethod();

    // Clearer version of the same thing
    IActor* actor = *it;
    actor->CallSomeMethod();
}

// ... or consider self-descriptive naming instead; this isn't explicit
// but still leaves enough cues for the reader to understand the code
// without extensive research

auto actorIterator = std::find(begin, end, value);
if (actorIterator != m_list.end())
{
    actorIterator->CallSomeMethod();
}

// Using auto in for-range statements may seem convenient, but it is
// not always as clear as its alternatives overall:
// e.g. dealing with "vector<int> m_list"

// Not a good use of auto; type hiding without any obvious gains
// (This also makes a copy of the list element which may not be your intention)
for (auto element : m_list)

// Clearer version
for (int element : m_list)

// For templates explicitly using T is usually clearer too:
// Note: Remembering to use "const" and "auto&" as appropriate with for-ranges
for (const T& element :  m_list)

// Complex types are more justifiable:
// e.g. dealing with "vector<pair<int, IActor*>> m_pairs"
for (const auto& element : m_pairs)
{
    // Type hidden version
    element.second->CallSomeMethod();

    // Clearer version through explicit local
    IActor* actor = element.second;
    actor->CallSomeMethod();
}
```

**Recommended usage**: In the case of casts, `auto` is preferred on the left side of the expression to remove redundant type information.

**`auto` cast examples**:

```c++
AZ::SerializeContext* serialize = azrtti_cast<AZ::SerializeContext*>(context);
// becomes
auto serialize = azrtti_cast<AZ::SerializeContext*>(context);
```

_Aside: In this case it is not recommended to use `auto` as the type `T` is already present on the right of the expression._ `auto` will match the exact type inside the cast expression (including const qualifiers). Type deduction does not take place as the type is provided explicitly.\_

### Use of nullptr

**Required**: Use `nullptr` in place of the macro `NULL` and the literal `0` for pointer values.

**Reason**: The keyword `nullptr` addresses a long standing deficiency in C++ by allowing a type safe way of unambiguously specifying an empty pointer value. (Other solutions are either ambiguous or not type safe).

### C++ Exceptions

**Required**: Do *NOT* use them. None of the core engine is exception safe and exceptions are disabled for all AZ projects.

> For tools or platform specific code (like WinRT) it is acceptable to use them judiciously.

### RTTI

**Required**: Do *NOT* use it.

**Recommended**: Where you do need runtime type information, use `AZ_RTTI` instead; a faster, lightweight type system with a variety of templated tools. See `<AZCore/Rtti/Rtti.h>`

### Casting

**Required**: Do not use `dynamic_cast`, as we do not use RTTI.

**Note**: It is possible to use `azdynamic_cast` that uses O3DE RTTI (types must opt in to use this using `AZ_RTTI` or equivalent).

**Recommended**: We highly encourage you to use the C++ casts: `static_cast`, `reinterpret_cast`, and `const_cast`, as they are more expressive about the intent of the cast. All casts, including C-style casts, have specific rules though. Make sure you learn them, and use them appropriately.

**Note**: if you are casting integer and floating point types, then it is recommended to use `aznumeric_cast<>`.


### Use of const

**Recommended**:

- Using const when passing parameters by value is not recommended.
- Using const when returning by value is not recommended.
- It is recommended in all other situations where const can be applied (such as reference parameters, local variables that don't change and member functions).

**Reason**: It is better to be restrictive than permissive, and it may help you catch or prevent some types of bugs. Clarity on the intent to modify a variable can also allow the compiler to do more optimizations.

### Don’t leave disabled code in the source

**Recommended**: We have source control if you want to restore some of your old code, no need to clutter up the files.

### Visual Studio project filter hierarchy should reflect the actual disk folder hierarchy

**Required**: This makes figuring out include paths and finding files in the project hierarchy and on disc much easier.

Note: we will switch to tool generated project files if in the future, as we'll need to support XCode, Eclipse, Android Studio, etc. We'll want to have the same view in all of these environments, and so will configure these generators to produce the same kind of disk mirroring project structures.

## Documentation

Great documentation is critical for the long term success of any project. Documentation comes in many forms, from blog posts to tutorials, and from many sources, from engineers to artists. For engineers the most significant area of documentation is the code itself, so this section of the Coding Standard is dedicated to recommendations intended to improve the overall consistency, depth and usefulness of code documentation.

Since code documentation is produced and consumed almost exclusively by engineers, it is important that all engineers consider themselves responsible for contributing to and maintaining it.

We use [Doxygen](http://www.stack.nl/~dimitri/doxygen/index.html) markup when documenting code. Using Doxygen markup has a number of benefits:

- The markup is human readable, so documentation is easy to read directly from the code
- The markup is compact, so it doesn't interfere with comprehension of the content itself
- The markup is flexible; contributors are encouraged to read the [Doxygen documentation guide](http://www.stack.nl/~dimitri/doxygen/manual/docblocks.html) to see all the markup tags available
- Stand-alone documentation is easy to generate using the Doxygen tool, and is considered easy to read and navigate
- The tool itself is free, easy to use, and relatively well known in the industry

**Guidelines**: [API Ref Guidelines]()

### Self-documenting code is better than documenting the code yourself

**Recommended**: Try to be specific. Your first priority should be to give good names to everything in the code

**Documentation Examples:**

**Bad Code – Requires a comment to explain itself**:

```c++
//! Gets the size of the object in bytes.
int  Size() { return m_size; }
```

**Good Code – Uses names that tell you what you need to know**:

```c++
int GetByteSize() { return m_byteSize; }
```

### Use full sentences and avoid abbreviations

**Recommended**: Full sentences with good grammar are preferable to heavily compacted or abbreviated notes. This improves readability and reduces ambiguity.

### Leave your feelings out of it

**Recommended**: Write your comments in an objective style; be free of emotion, conjecture and machinations. It is usually never appropriate to add an exclamation mark (❗), if you're shouting in your comments you're probably emotional 🙂

### Use non-gendered pronouns

In the spirit of inclusivity and objective documentation, avoid gendering the reader or parts of the code.

See [https://chromium.googlesource.com/chromium/src/+/master/styleguide/gender_neutral_code.md](https://chromium.googlesource.com/chromium/src/+/master/styleguide/gender_neutral_code.md) for more information

### Explaining "why" is often more valuable than "what"

**Recommended**: The code intrinsically represents what it being done, but it rarely communicates why it is being done. Adding a short comment to explain the reason for choosing a specific algorithm, data structure or other implementation detail adds something that the code alone can't provide. Of course, sometimes you need to explain "what" too, e.g. if the algorithm is non-obvious or complex.

### Private members need love too

**Recommended**: Public members are obviously very important, since they form the interface that other code will use to access an object, but you should not forget that private and protected members also benefit from being documented. Never assume you will be the only person maintaining the code and leave some clues for your successor.

### Redundant documentation is redundant

**Recommended**: Documentation should always add value. If the documentation adds nothing there is no reason to include it

**Not useful - it just repeats what the code already says**:

```c++
// Iterates through the player list
for (int i=0; i<players.size(); ++i) { ... }
```

**Not useful - non-specific warning may as well not be here**:

```c++
// This will assert if the state is not correct
void OnEventFired()
```

### TODO comments

**Recommended**: Generally TODO is not very useful in code, unless it's an assert. Do not leave TODO-like messages in comments, use Github issues instead, the presence of TODOS could be interpreted as promises to deliver something.  Describe what the code is now, not what it used to be or what it will be.

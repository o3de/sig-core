# C++ Best Practices Guide <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->

- [Best Practices](#best-practices)
- [Numeric Limits](#numeric-limits)
- [Deprecation comments](#deprecation-comments)
  - [ComponentApplicationEventBus Deprecation](#componentapplicationeventbus-deprecation)
- [Memory Management](#memory-management)
  - [Memory Allocations](#memory-allocations)
  - [Issues caused by static variables](#issues-caused-by-static-variables)
    - [Function Static Variables of Container which uses `AZ::SystemAllocator`](#function-static-variables-of-container-which-uses-azsystemallocator)
    - [Global Static Connecting to EBus](#global-static-connecting-to-ebus)
- [Data Structures](#data-structures)
  - [Containers](#containers)
  - [Smart Pointers](#smart-pointers)
    - [Tips when to use specific smart pointer types](#tips-when-to-use-specific-smart-pointer-types)
    - [Tips when to use make smart ptr T vs aznew T](#tips-when-to-use-make-smart-ptr-t-vs-aznew-t)
    - [make_unique example](#make_unique-example)
  - [String Buffer Access](#string-buffer-access)
    - [Printing strings](#printing-strings)
  - [string view vs const string](#string-view-vs-const-string)
    - [string view parameter passing](#string-view-parameter-passing)
    - [string view printing examples](#string-view-printing-examples)
- [Error Handling](#error-handling)
  - [Tracing](#tracing)
- [Testing](#testing)
  - [Unit Tests](#unit-tests)
    - [Avoiding Unit Tests which access disk](#avoiding-unit-tests-which-access-disk)
  - [Testing Error Conditions with AZ Trace Suppression Tests](#testing-error-conditions-with-az-trace-suppression-tests)
- [API Design](#api-design)
  - [Passing Parameters by Value vs Const Ref](#passing-parameters-by-value-vs-const-ref)

## Best Practices

> If you would like to make a change to this document, the [RFC Suggestion Template](https://github.com/o3de/sig-core/issues/new?assignees=&labels=rfc-suggestion&template=rfc-suggestion.md&title=Proposed+RFC+Suggestion+%3Ddescription%3D) can be used to create an RFC for changes to governance documents relating to O3DE.

The [Coding Standards and Style Guide](https://github.com/o3de/sig-core/blob/main/governance/Coding-Standards-and-Style-Guide.md) deals primarily with issues of semantics, language features and general low-level consistency. The [Best Practices](#best-practices) guide builds on this at a higher level, covering common coding problems, solutions, advice and other patterns that we have learned through experience and think are useful for all engineers working on the code to follow.

This document follows similar conventions to the [Coding Standards and Style Guide](https://github.com/o3de/sig-core/blob/main/governance/Coding-Standards-and-Style-Guide.md). There are a number of topics each with a **Recommended** tag. Please refer to the [Coding Standards and Style Guide](https://github.com/o3de/sig-core/blob/main/governance/Coding-Standards-and-Style-Guide.md) before reading this document.

Do note that this document is not an O3DE standard. It contains helpful guidelines and rules to follow to make it easier to write quality code. However, C++ is a very complex language and it's next to impossible to cover every situations in this document, therefore these guidelines should not be blindly applied but should serve as guide to evaluate your particular situation.

## Numeric Limits

**Recommended**: Use `AZStd::numeric_limits` rather than macros such as `FLT_MAX`.

**Reason**: `AZStd::numeric_limits` works correctly in templated code and can also be used in non-templated code. `AZStd::numeric_limits` defines values like `max, min` and `lowest` consistently across types, whereas there is potential for confusion with macros. `AZStd::numeric_limits` requires only a single header to be included (`<AzCore/std/limits.h>`), while the macro versions require different headers depending on the type (`cfloat.h, climits.h` etc.). The macro versions could potentially be undefined or redefined.

**Caveats**: `AZStd::numeric_limits` can't be used in the preprocessor.

## Deprecation comments

**Recommended**: The [Deprecation](https://github.com/o3de/sig-core/blob/main/governance/API-Ref-Guidelines-Update.md#deprecations) section of the [API Ref Guidelines](https://github.com/o3de/sig-core/blob/main/governance/API-Ref-Guidelines-Update.md) should be followed when it comes to deprecation. If the deprecated API has been replaced with a newer API, prefer to forward calls from the deprecated function to the new function.

Examples of API Deprecation are below.

### ComponentApplicationEventBus Deprecation

```c++
//! Deprecation of an EBus interface class


//! Event bus for dispatching component application events to listeners.
//! @deprecated The ComponentApplicationEvents class has been deprecated.
//! Please use the EntitySystemEvents class instead which provides the Entity
//! Ebus notifications directly from the Entity class itself instead of the ComponentApplication

//! The ComponentApplicationEvents class has been deprecated in favor of using
//! O3DE_DEPRECATION_NOTICE(<GHI Number>)
class ComponentApplicationEvents
: public AZ::EBusTraits
{
public:
    static const AZ::EBusHandlerPolicy HandlerPolicy = EBusHandlerPolicy::Multiple;
    static const AZ::EBusAddressPolicy AddressPolicy = AZ::EBusAddressPolicy::ById;

    //! Ensures that only the listeners that care about the entity being added or
    //! removed will be notified.
    using BusIdType = AZ::EntityId;

    //! The ComponentApplicationEvents OnEntityAdded event has been deprecated in favor of
    //! the EntitySystemEvents OnEntityInitialized function
    //! O3DE_DEPRECATION_NOTICE(<GHI Number>)
    //! Notifies listeners that an entity was added to the application.
    //! @param entity The entity that was added to the application.
    //! @deprecated OnEntityAdded has been deprecated in favor of EntitySystemEvents::OnEntityInitialized function
    void OnEntityAdded([[maybe_unused]] AZ::Entity* entity) {}

    
    //! Notifies listeners that an entity was removed from the application.
    //! @param entity The entity that was removed from the application.
    virtual void OnEntityRemoved([[maybe_unused]] const AZ::EntityId& entityId) {}
};

//! The ComponentApplicationEventsBus has been deprecated in favor of using
//! the EntitySystemBus
//! O3DE_DEPRECATION_NOTICE(<GHI Number>)
//! Used when dispatching a component application event.
//! @deprecated The ComponentApplicationEventBus has been deprecated by the EntitySystemBus
using ComponentApplicationEventBus = AZ::EBus<ComponentApplicationEvents>;
```

The most **important** task when deprecating a C++ name/entity is to communicate it to the community. This notifies users of changes coming down the pipe even in the case where the class cannot be deprecated through code for technical reasons. The O3DE Discord [impactful-changes](https://discord.com/channels/805939474655346758/852574203437121586) channel is good place to notify users of changes.

## Memory Management

### Memory Allocations

**Recommended**: Calling `new`, `malloc`, et. al. directly is not recommended. Instead, all allocations are to be piped through AZ memory managers via calls to `aznew`, `azmalloc`, `azfree`, `azcreate`, and `azdestroy`, and by specifying the actual allocators in each class. New Gems and subsystems should either create their own allocator or create a ChildAllocator referencing an existing allocator in order to tag and track resource usage within each system.

**Reason**: We work in a memory managed environment, as in the core AZ systems are providing you with memory instead of the raw system allocator, not as in C#/Java managed memory. We do this for speed, safety, and so that we can track all memory usage.

### Issues caused by static variables

**Recommended**: Avoid global and function local static variables of class type and containers which allocate memory from AZ Allocators in their constructors and deallocate memory in their destructors. Also avoid global static variables that connect/disconnect to EBuses within their constructors/destructors.

**Reason:** As O3DE manages the memory of its classes and container types using the AZ Memory Allocator system, static variables of class type and containers are created before the AZ Memory Allocators have had the opportunity to hook into the module containing the statics. The lifetime of static variables last until whichever module they are declared in are unloaded. For static variables in static linked libraries their duration is the entire duration of the application. This will cause crashes on startup of the Application or loading of a Gem dll if a static variable attempts to allocate from the AZ System Allocator as it has not been created yet. Crashes will also occur on shutdown if the static variable attempts to deallocate from the AZ System Heap as the AZ System Allocator no longer exist at that point.

For example, the below code is actual code in an O3DE AzFramework project that shows one of the issues with using static local variables.

#### Function Static Variables of Container which uses `AZ::SystemAllocator`

```c++
namespace Physics
{
    AZ_INLINE void GetDefaultDebugDrawSettings(DebugDrawSettings& settings)
    {
        static AZStd::vector<Vec3> cryVerts;
        static AZStd::vector<ColorB> cryColors;
        //...
    }
} // namespace Physics
```

The above code has issues in that the `AZStd::vector` class by default uses the `SystemAllocator`, which has a well defined lifetime. Therefore invoking this function even once will cause a deletion from a destroyed heap on application shutdown.

An example of a global static variable which causes an issue on startup is shown below.

#### Global Static Connecting to EBus

```c++
namespace LmbrCentral
{
    //! Wraps a IMaterial pointer in a way that BehaviorContext can use it
    class MaterialHandle
    : public AZ::RenderNotificationsBus::Handler
    {
    public:
        AZ_CLASS_ALLOCATOR(MaterialHandle, AZ::SystemAllocator, 0);
        AZ_TYPE_INFO(MaterialHandle, "{BF659DC6-ACDD-4062-A52E-4EC053286F4F}");

        MaterialHandle()
        {
            AZ::RenderNotificationsBus::Handler::BusConnect();
        }
        //...
    };

    // Declares a MaterialHandle in global space later
    MaterialHandle g_defaultMaterialHandle;
} // namespace LmbrCentral
```

The code above will crash as soon as a module that contains the global variable definition is loaded. This is due to connecting to an EBus in the constructor of a global variable. The EBus system uses an `EnvironmentVariable` which relies on the Module where it resides having initialized an Environment which cannot occur until the module initialization occurs. Because the variable is declared at global scope, it accesses the Environment before it is ready.

## Data Structures

### Containers

**Recommended**: _Do_ use `AZStd`, Do _not_ write your own containers.

We have a full range of containers implemented and if something is missing we will add it for you. You have: `array`, `vector`, `list`, `forward_list`, `deque`, `queue`, `stack`, `map`, `bitset`, `multimap`, `set`, `multiset`, `unordered_map`, `unordered_multimap`, `unordered_multiset`, `ring_buffer`, `fixed_vector`, `fixed_list`, `fixed_forward_list`, `fixed_unordered_map`, `fixed_unordered_set`, and more. For multithreaded environments, you also have lockless containers as well as concurrent `vector`, `map`, `set` and fixed versions thereof. You can find all of these under `AzCore/std/containers`.

**Recommended**: Containers should always be created by value, rather than allocated with `new`. None of the `AZStd` containers allocate memory when empty, so there are very few reasons to ever want to directly `new` them on the heap.

**Recommended**: Containers should usually store their contents by value if they can. Otherwise if the container is the owner of the dynamically allocated contents it is preferred that they are stored in `AZStd` smart pointers.

### Smart Pointers

**Recommended:** Prefer smart pointers to raw pointers when object ownership semantics is needed.

**Reason**: Smart pointers encapsulate the lifetime management of C++ objects to avoid memory leaks when an object scope ends.

#### Tips when to use specific smart pointer types

- `AZStd::unique_ptr<T>` should be used as a when single owner memory semantics for a heap allocated object is required.
- `AZStd::unique_ptr<T[]>` can be used as a replacement for allocating an array of an instance and the overhead of an managing capacity and size isn't needed (otherwise `AZStd::vector<T>` can be used instead.
- `AZStd::shared_ptr<T>` should be used when shared ownership semantics is needed for a heap object. Shared pointers add a reference count structure to allow both strong references and weak references to an object.
- `AZStd::weak_ptr<T>` should be used store a weak reference to an allocated heap object. It will prevent access to the object if it no longer exist.
- `AZStd::intrusive_ptr<T>` is similar to `shared_ptr`, but the reference count structure is associated with the heap object. Therefore no allocations are performed to create `AZStd::intrusive_ptr<T>` instances from an object.
`T*` use for referencing heap objects which are not owned by the field. **Caveat**: Qt ownership semantics use am ObjectTree approach where parent `QObjects` delete children `QObjects`, so raw pointers should be used in this case as well to prevent double deletions due to smart pointers going out scope.

#### Tips when to use make smart ptr T vs aznew T

- `AZStd::make_unique` should be used when a T object needs to be created and either a custom allocator is not needed to allocate a T object or a custom deleter is not needed to delete the `unique_ptr` instance.
- `AZStd::make_unique` has the added benefit of avoiding a memory leak if for some reason an exception is thrown between the time an object is new'ed (`new T`) and stored within the `unique_ptr` (`unique_ptr<T>(new T)`).

#### make_unique example

```c++
static void ReflectSceneModule(ReflectContext* context, AZStd::unique_ptr<DynamicModuleHandle>&& module);

//! Call to ReflectSceneModule using AZStd::unique_ptr<T>(new T)
ReflectSceneModule(new AZ::SerializeContext, AZStd::unique_ptr<DynamicModuleHandle>(new DynamicModuleHandle));

// The function argument resolution order is unspecified in C++
// So in the case of a function argument evaluation order of
// 1. new DynamicModuleHandle, 2. new AZ::SerializeContext, 3. AZStd::unique_ptr<DynamicModule>(<result from step 1>)
// First the DynamicModuleHandle instance is created first, followed by a call to AZ::SerializeContext constructor and that if constructor throws an exception, then the created DynamicModuleHandle object would leak as it has not been associated with the AZStd::unique_ptr yet

//! Call to ReflectSceneModule using AZStd::make_unique<T>()
ReflectSceneModule(new AZ::SerializeContext, AZStd::make_unique<DynamicModuleHandle>());

// Due to there not being an explicit call to new in the creation of the DynamicModuleHandle unique_ptr
// either the construction of the unique_ptr would have completed before the AZ::SerializeContext constructor throws an exception and therefore the AZStd::unique_ptr<DynamicModuleHandle> would cleanup the DynamicModuleHandle instance
// or the the AZ::SerializeContext constructor throws an exception before the unique_ptr was even created
```

`AZStd::make_shared` has the same advantages and drawbacks as `AZStd::make_unique` (memory doesn't leak if an exception occurs, but a custom deleter cannot be specified) with the additional benefit that the control block structure and object instance for the `AZStd::shared_ptr<T>` performs only one heap allocation, where an `AZStd::shared_ptr<T>(new T())` requires two heap allocations. Also the `AZStd::shared_ptr<T>` supports the same creation of its control block structure and object instance with a custom allocator using the `AZStd::allocate_shared` function.

### String Buffer Access

**Recommended**: Prefer `c_str()` over `data()` when accessing a string's internal buffer.

**Reason**: `string::c_str()` is an explicitly safe operation because it guarantees null-termination. Similarly, `string::data()` guarantees null-termination (since C++ 11), _but_ `string_view`, `vector`, and other classes also have a `data()` member function which is _not_ always null-terminated. Using `string::data()` leaves the door open for new bugs, if someone changes the string variable's data type and forgets to fix the places where `data()` was called. Also, using `c_str()` is more readable than `data()` because it clearly communicates the buffer is null-terminated, without the readier having to determine the variable's type.

#### Printing strings

```c++
AZ_Printf("Category", "%s", messageA.data()); // You can't tell whether this operation is safe without knowing the context
AZ_Printf("Category", "%s", messageB.c_str()); // This line of code is clearly safe
```

### string view vs const string

**Recommended**: Use `string_view` instead of a `const string&` for function parameters that accept strings which are not stored immediately within the function.

**Reason**: `string_view` provides access to all the immutable functions of a `std::string` without the need to allocate `std::string`, when a character literal is passed to a function.

#### string view parameter passing

```c++
// Below are several ways to pass an immutable string to a function with each of
// there benefits and detriments being listed above them

// Accepts a null-terminated string
// Benefits: Compact, only uses stack space for the size of a pointer,
// Most C standard API function work with null-terminated string so no additional conversions are needed when using them
// Detriments: Requires the string to be null-terminated,
// Has to examine entire string to calculate string size
void CompareString(const char* cString);

// Accepts a raw string and the amount of characters in that raw string
// Benefits: Does not have to calculate the string size within the function,
// works with non-null terminated strings
// Detriments: Requires the stack space for a pointer and a size_t,
// string data and string size are passed separately from each other even though are related
void CompareString(const char* rawString, size_t size);

// Accepts a const reference to a AZStd::string
// Benefits: Allows passing both AZStd::string objects and c strings using the same interface,
// Does not make a copy if a AZStd::string parameter is passed to the function.
// Only uses stack space for the size of a pointer
// Detriments: Allocates from heap memory a new AZStd::string if a const char* parameter is passed to the function,
// Only works for the AZStd::string type parameters, other string like interfaces requires extraction of a const char* first and then
// construction of a new AZStd::string
void CompareString(const AZStd::string& stdString);

// Accepts a AZStd::string_view by value
// Benefits: Allows passing both AZStd::string objects, c strings and non-null terminated strings with a supplied size using the same interface,
// Never copies allocates memory or makes a copy of the supplied string, therefore it can work with any string type(std::string, AZStd::string) that invokes the { string_view(char*, size_t); } constructor
// Avoids compiler indirection for references and allows better compiler optimizations due to be passed by value
// Can wrap non-null terminated strings
// Provides the same interface as AZStd::string immutable functions(find, substr, compare, etc...)
// Provides a non-allocating substr function unlike AZStd::string
// Detriments: Using the cString constructor{ string_view(const char*); } requires the string length to be calculated,
// Does not own the string and therefore the underlying string can be deleted from under it
// Default constructor creates a string_view whose const char* member is nullptr, therefore care must be taken when passing string_view::data() to a function that works on c-strings
// Easy to mistakenly use if the user assumes that it always wraps a null-terminated string(i.e passing string_view::data() to printf without using the precision specifier
void CompareString(AZStd::string_view);
```

**Recommended**: When using `string_view` within printf-like statements prefer to use the `AZ_STRING_ARG/AZ_STRING_FORMAT` macros to properly specify the size of the string or the precision specifier if the macros are not available.

**Reason**: `strings_view` is implemented as two members. A constant char pointer to the beginning of a string and the size of the string. It is not necessarily null-terminated. It can be constructed with a non-null terminated string if a size parameter is supplied, or a substring of an existing `string_view` is taken. Furthermore, the `string_view` [default constructor](https://en.cppreference.com/w/cpp/string/basic_string_view/basic_string_view) sets the string to `nullptr` and not `""`.

#### string view printing examples

```c++
void PrintString()
{
    // rawString is NOT null_terminated
    char* rawString[11] = { 'H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd' };

    // Constructs string_view by passing in the explicit size of the string
    AZStd::string_view rawView(rawString, 10); // Initialized with const char* = ("Hello World" + non-initialized data after the string) and size = 10
    printf("%s", view.data()); // Undefined Behavior: the string_view is a view into a non-null terminated string.
    printf("%.*s", aznumeric_cast<int>(view.size()), view.data()); // Correct: only outputs characters up to the length of the string
    printf(AZ_STRING_FORMAT, AZ_STRING_ARG(view)); // Correct: Uses MACRO: only outputs characters up to the length of the string

    AZStd::string_view null_terminated_view("Hello World"); // Initialized with const char* = "Hello World" and size = 11
    printf("%s", null_terminated_view.data()); // Technically incorrect, but Works: Requires the user to have knowledge that the string_view is viewing a   null-terminated string
    printf("%.*s", aznumeric_cast<int>(null_terminated_view.size()), null_terminated_view.data()); // Correct: always works
    printf(AZ_STRING_FORMAT, AZ_STRING_ARG(null_terminated_view)); // Correct: Uses MACRO always works

    AZStd::string_view substr_view = null_terminated_view.substr(0, 5); // Initialized with const char* = "Hello World" and size = 5
    printf("%s", substr_view.data()); // Incorrect: outputs the entire null_terminated string of "Hello World"
    printf("%.*s", aznumeric_cast<int>(substr_view.size()), substr_view.data()); // Correct: outputs "Hello"
    printf(AZ_STRING_FORMAT, AZ_STRING_ARG(substr_view)); // Correct: Uses MACRO, outputs "Hello"

    AZStd::string_view empty_view; // Initialized with const char* = nullptr and size = 0
    printf("%s", empty_view.data()); // Undefined Behavior: nullptr dereferences occurs when printf attempts to dereference the first character
    printf("%.*s", aznumeric_cast<int>(empty_view.size()), empty_view.data()); // Correct: Will not attempt to deference the string_view as the precision specified is 0
    printf(AZ_STRING_FORMAT, AZ_STRING_ARG(empty_view)); // Correct: Uses MACRO, outputs nothing at at all as the empty_view sets the precision to 0
}
```

## Error Handling

### Tracing

**Required**: Use `AZ` tracing macros (e.g. `AZ_Printf`, `AZ_TracePrintf`, `AZ_Warning`, `AZ_Error` etc.) for all code tracing needs (see `Framework/AzCore/AzCore/Debug/Trace.h` for more details). For custom type tracing (tools, etc.) or something that needs to be active in release, please create a [Request For Comment (RFC)](https://github.com/o3de/sig-core/issues/new/choose) document detailing the reasons for custom tracking to facilitate feedback from other developers.

**Reason**: Error handling and tracing functions are an important part of the development process used to provide friendly formatted messages to developers and designers about what errors are occurring and why. The tracing functions indicate where in code they are logged from to allow debugging.

## Testing

### Unit Tests

**Recommended**: All core systems should be unit test driven as much as reasonable, without creating complex systems to administer them. The unit test framework is built on GTest and GMock. Unit tests are configured to run through the Automated Review (AR) pipeline to validate Pull Requests. Instructions for building and running unit tests are detailed in the [Getting Started with Test Automation](https://www.o3de.org/docs/user-guide/testing/getting-started/) guide.

**Reason**: Unit testing will catch many bugs, usually at a very early stage. Unit testing also provides validation that existing systems are in working order.

**Recommended**: Unit tests which do not specifically test file based functions should avoid saving and loading to a file. When testing an API that loads its data from a file, add an interface that takes a stream as well. For O3DE this usually involves adding a function which accepts a `AZ::IO::GenericStream` parameter.

**Recommended**: Unit tests which require file IO based functionality should use the GMock framework to create mock calls.

**Reason**: File based unit tests are prone to issues that are normally outside the scope of the functionality being tested, such as the file being unable to be saved/loaded due to permissions, the file no longer existing when the test runs, etc... Furthermore, platforms such as Android and iOS have heavier restrictions on where files can be written and read from. Finally, file based unit test are a source of failures in cases where the file on disk has not been cleaned up between unit tests.

Below is an example of an actual unit test performing disk IO when it shouldn't

#### Avoiding Unit Tests which access disk

```c++
// Below is a ObjectStreamUnit Test which primary purpose is to validate that an ObjectStream
// is able to be saved and loaded without error.
// The interface functions for saving and loading properly accepts a GenericStream interface and not a FileIOStream or StreamerStream parameter
void TestSave(IO::GenericStream* stream, ObjectStream::StreamType format)
{
    ObjectStream* objStream = ObjectStream::Create(stream, *m_context, format);
    SaveObjects(objStream);
    bool done = objStream->Finalize();
    EXPECT_TRUE(done);
}

void TestLoad(IO::GenericStream* stream)
{
    int cbCount = 0;
    bool done = false;
    ObjectStream::ClassReadyCB readyCB(
        AZStd::bind(&SerializeBasicTest::OnLoadedClassReady, this, AZStd::placeholders::_1, AZStd::placeholders::_2, &cbCount));
    ObjectStream::CompletionCB doneCB(
        AZStd::bind(&SerializeBasicTest::OnDone, this, AZStd::placeholders::_1, AZStd::placeholders::_2, &done));
    ObjectStream::LoadBlocking(stream, *m_context, readyCB);
    EXPECT_EQ(27, cbCount);
}

// AVOID: Avoid writing a test such as the one below which uses a streamer stream to save and load an object stream on disk
// Where the ObjectStream is written out is not relevant to this test, only that that ObjectStream has been written out successfully
TEST_F(Serialization, BasicTest)
{
    {
        AZ_TracePrintf("SerializeBasicTest", "\nWriting as JSON...\n");
        IO::StreamerStream stream("serializebasictest.json", IO::OpenMode::ModeWrite);
        TestSave(&stream, ObjectStream::ST_JSON);
    }

    {
        AZ_TracePrintf("SerializeBasicTest", "Loading as JSON...\n");
        IO::StreamerStream stream("serializebasictest.json", IO::OpenMode::ModeRead);
        TestLoad(&stream);
    }
}

// BETTER: Write the test data to an in-memory stream and load that instead
// This makes the test more tolerant to be run on other platforms and avoids any disk based IO issues
TEST_F(Serialization, BasicTest)
{
    // JSON version
    {
        AZStd::vector<uint8_t> byteBuffer;
        IO::ByteContainerStream<decltype(byteBuffer)> stream(&byteBuffer);
        AZ_TracePrintf("SerializeBasicTest", "\nWriting as JSON...\n");
        TestSave(&stream, ObjectStream::ST_JSON);

        // Seek to the beginning of the in memory byte stream in order to load it
        stream.Seek(0, AZ::IO::GenericStream::ST_SEEK_BEGIN);
        AZ_TracePrintf("SerializeBasicTest", "Loading as JSON...\n");
        TestLoad(&stream);
    }
}
```

**Recommended**: When creating or working on defects with complex reproduction steps, attempt to use the UnitTest framework to create a regression test for the defect.

**Reason**: Being able to a reproduce an issue in a unit test helps with isolating the code area responsible for it and simplifies the repro steps to just running the unit test through the TestRunner. Furthermore, it allows other developers to work on that issue at a later date with a lower cognitive burden to determine the issue. Finally, the written unit test acts as a regression test to detect if the issue ever occurs again.

The benefit of creating a unit test to repro an issue is that the issue can be observed just by running a single unit test. This reduces iteration time when working on the issue. Moreover it also allows a different developer to work on this issue with less overhead needed to get up to speed.

### Testing Error Conditions with AZ Trace Suppression Tests

**Recommended**: To write a unit test that verifies whether a statement that logs an `AZ_Error` is an expected error, the `UnitTest::TraceBusRedirector` class and the `AZ_TEST_START_TRACE_SUPPRESSION`/`AZ_TEST_STOP_TRACE_SUPPRESSION` macros can be used together to verify a unit test logs the expected number of errors. This can be used for suppressing `AZ_Error` messages. For writing a unit test  where an `AZ_Assert` is expected, a [Google Death Test](https://github.com/google/googletest/blob/main/docs/advanced.md#death-tests) should be used instead.

Note: If the `AZ_UNIT_TEST_HOOK` macro has been given a custom test environment, a `UnitTest::TraceBusRedirector` will need to be connected by the test class or test environment as shown below. Otherwise, a `UnitTest::TraceBusRedirector` will automatically be installed by the default environment and you should not add your own, otherwise the test will end up double counting all `AZ_Error` messages resulting in an incorrect expectation count causing the test to fail.

```c++
class Vector2ErrorTest
    : public ::testing::Test
    , public UnitTest::TraceBusRedirector
{
protected:
    void SetUp() override
    {
        AZ::Debug::TraceMessageBus::Handler::BusConnect();
    }

    void TearDown() override
    {
        AZ::Debug::TraceMessageBus::Handler::BusDisconnect();
    }
};

TEST_F(Vector2ErrorTest, GetElement_ErrorsWhenIndexIsOutOfBounds)
{
    AZ_TEST_START_TRACE_SUPPRESSION;
    AZ::Vector2 vector2;
    // 3 is an invalid index for a Vector2
    vector2.GetElement(3);
    // Expect 1 error
    AZ_TEST_STOP_TRACE_SUPPRESSION(1);
}
```

## API Design

### Passing Parameters by Value vs Const Ref

There are many factors affecting whether you design your API to take a given parameter by value or by const ref (and similarly whether to return an output as a return value or a ref parameter).

As usual, **consistency** is recommended. For example, most existing O3DE APIs pass AZ::EntityId by value so it recommended that you do the same unless there is some special reason not to.

All other things being equal...

**Recommended**: If it is a built-in type (such as `int`, `double` or `float`) or if the size of the type is less than or equal to 64 bits (8 bytes) AND it has a trivial inline copy constructor then prefer pass by value.

**Reason**: As types get larger or have more complex copy semantics it becomes less efficient to pass them by value. However, this is very compiler and platform dependent. Having a simple guideline to use when there are no other special cases helps keep our APIs consistent.

**Caveats**: There can be good reasons to not follow the general recommendation above in some cases. Just to name a few:

- When passing variables across threads pass by value may be preferred.
- A type may have a non-trivial copy constructor but still be fine to pass by value (such as QString which uses copy on write).

# Summary:
This proposes a solution to managing the lifetime of the stateful allocators
defined in O3DE's AzCore library. This will be accomplished by moving the
allocators out of the static AzCore library, whose symbols are copied into
every user of AzCore. Instead, they will be defined in a shared AzCore
allocators library, so that there is a single instance of each allocator type
available.

# What is the relevance of this feature?
There are a number of use cases for allocators that the current framework does
not support, and many existing workarounds that all have some drawbacks:

* The stateful allocators cannot be used before the AZ::Environment is
  attached. This affects all global static variables, and code that runs in
  order to attach the Environment.
* Workarounds for allocators being unconstructible result in allocations that
  are untracked. For example, some places use `AZ_OS_MALLOC` instead of using
  an allocator, due to the allocator being unconstructible at the needed time.

# Feature design description:
To accomplish the goals of this feature, any given allocator instance is
constructed on demand, at the time it is first used. Each allocator type will
define a static `Get()` method, which will construct an instance of that
allocator type the first time it is called. The allocator's lifetime is then
guaranteed to outlive any variable that uses that allocator. The existing
`AllocatorManager` class becomes an optional part of the system, in that the
manager can be used to report memory leaks and iterate over known registered
allocator instances, but is not the controller for the allocator's
construction/destruction.

To ensure that there is one single instance of any allocator type available,
the allocators and their dependent parts of AzCore will be moved to a shared
library. We'll call that library `O3DEKernel`. `O3DEKernel` include the
following subsystems from AzCore:

* The Allocator types in `AzCore/Memory/*`
* The `AllocatorManager` type
* The RTTI and TypeInfo system in `AzCore/RTTI/*`
  * RTTI depends on UUID, from `AzCore/Math/Uuid.h`
* The Trace system in `AzCore/Debug/*`
* The AZStd library. AZStd depends on Trace, and the Allocators depend on AZStd.
* The Environment system

The use of a shared library eliminates the need for storing the allocator
instances in the `AZ::Environment`. Calls to `AllocatorType::Get()` will
resolve to a single method inside the shared library that defines the
`AllocatorType`. That same library holds the singleton for that type, so using
the Environment to unify the symbol duplication that results from using static
libraries is no longer necessary.

Some allocators depend on other allocator types. For instance, the
SystemAllocator (which uses the Hpha schema) uses the OSAllocator to get its
pages. To ensure that the dependent allocators are destroyed in the reverse
order of their dependencies, all allocators will need to call
`DependentAllocator::Get()` in their constructor.

## Example uses

### Using AZStd containers with no AZ::Environment

```cpp
#include <AzCore/std/vector.h>
int main()
{
    const AZStd::vector<int> ints {1, 2, 3, 4, 5};
    return ints.size();
}
```

On line 4, an instance of a `AZStd::vector<int, AZStd::allocator>` is created,
and is passed an initializer list for the contents of the new vector. The
vector needs to allocate space for those elements, so it uses its allocator's
`allocate()` function.

`AZStd::allocator` uses `AZ::SystemAllocator` for all its allocations. This
allocator is fetched by instantiating the template function
`AZ::AllocatorInstance<AllocatorType>::Get()`. This function will now forward
that call to `AllocatorType::Get()`.

```cpp
template<class AllocatorType>
AllocatorType& AllocatorInstance<AllocatorType>::Get() // This is the current method AZ::AllocatorInstance uses to get the global allocator instances
{
    return AllocatorType::Get();
}
```

`AllocatorType::Get()` will be defined as a DLL-exported function inside O3DEKernel:

```cpp
class O3DEKERNEL_API SystemAllocator
    : public IAllocator
{
public:
    static O3DEKERNEL_API SystemAllocator& Get()
    {
        static SystemAllocator instance{};
        return instance;
    }
};
```

The callstack for this code will look like:

```
myapp!AZStd::vector<int>::vector(std::initializer_list<int> list, const AZStd::allocator& allocator = AZStd::allocator())
\-myapp!AZStd::vector<int>::construct_iter()
  \-myapp!AZStd::allocator::allocate(size_type byteSize, align_type alignment)
    \-myapp!AZ::AllocatorInstance<SystemAllocator>::Get()
      |-O3DEKernel!SystemAllocator::SystemAllocator()
      | |-\O3DEKernel!HphaAllocator<OSAllocator>::HphaAllocator()
      | |  \-O3DEKernel!AZ::AllocatorInstance<OSAllocator>::Get()
      | |   |-O3DEKernel!OSAllocator::OSAllocator()
      | |   #- 1. at this point, the construction of the OSAllocator variable is complete
      | # 2. at this point, the construction of the SystemAllocator variable is complete
      myapp!SystemAllocator::Allocate()
```

Because the construction of the static variable of type OSAllocator (the
dependent allocator) completes (at point 1) before the construction of the
static variable of type SystemAllocator completes (at point 2), and that static
variables are destroyed in reverse order of completion of their constructors,
the SystemAllocator will be destroyed before the OSAllocator.

# What are the advantages of the feature?
The advantages of this feature are that stateful allocators are always
available. This allows for the removal of previous workarounds which would
avoid tracking some allocations, and allows for the use of AZStd-allocator
aware types to be used during static initialization.

# What are the disadvantages of the feature?
This design works best if the `O3DEKernel` library can be the main store for
allocator types. If authors of other systems want to implement their own
allocator types, those allocators will also need to be defined in a shared
library, so that there is a single definition of the allocator's `Get()`
method. Many existing systems today do not use shared libraries at all.

# How will this be implemented or integrated into the O3DE environment?
The new `O3DEKernel` library will be the root of the dependency tree,
restructuring the existing allocator codebase.

# Are there any alternatives to this feature?
The possibilities of using stateful allocators in C++ has been improving
recently, including additions to the STL with the
`std::pmr::polymorphic_allocator` template. But that does not address how
stateful allocators are gathered together in one place, like in O3DE's
`AllocatorManager`, nor how they are to be constructed for global static
variables to use them. There are no existing alternatives.

# How will users learn this feature?
This feature will be quite transparent to most users. Engineers that want to
track memory usage will find that more of their allocations are tracked,
without any intervention on their part.

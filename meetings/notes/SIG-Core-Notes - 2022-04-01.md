# Agenda 3 - Allocator Availability RFC(#31)
@burelc-amzn

## @amzn-phist - How will allocators be made available to control static initialization?

The Allocators will be pushed into a dll that is statically linked to the executable.  
The Environment will be initialized as well when the dll is loaded.

## @amzn-phist - How will Monolithic build static initialization work?

Because all static libraries are linked into executable, the O3DEKernel library will have a single instance linked into the executable.  
@amzn-phist Concerns around monolithic with how using a static variable(AZStd::string, etc...) guarantees initialization of the allocator.  
@burelc-amzn Explained that the allocator is created on demand within the O3DEKernel a static variable requires allocator

## @spham-amzn - Who is responsible for freeing the allocator 

@burelc-amzn On first access of an AZ allocator, it will be created on demand within the O3DEKernel memory space.  
The allocator will be destructed in static de-init of the O3DE Kernel.

## @lumberyard-employee-dm - How is static deinitialization interleaved with the DLL memory space and executable memory space?

@burelc-amzn The statically linked O3DEKernel DLL would static init before the main executable memory space static init.
Therefore the O3DEKernel dll would static de-init after the main executable static de-init.  
This would guarantee that the allocator would outlive any static variables in the main executable.

## Should the name of the Allocator library be called O3DEKernel?

@spham-amzn O3DEKernel implies more than just the Allocator.  
O3DEKernel will be used unless we can come up with a better name for the library

## @burel-amzn Revisions to RFC

Remove Trace system from O3DE Kernel library.  
Use trace function pointer to hook trace functions into the allocation system

# Agenda 2- CODEOWNERS file
@lumberyard-employee-dm

## What areas of the o3de repo should SIG-Core be added to the CODEOWNERS file for Pull Requests
1. Code/Framework/AzCore, Code/Framework/AzFramework, Code/Framework/AzGameFramework
1. Code/Framework/AzToolsFramework? (Shared with SIG-Content?)
1. Gems/LmbrCentral? (Shared with SIG-Content?)
1. Gems/Profiler
1. Gems/Prefab? (Shared with SIG-Content)
1. Gems/ImGui
1. Registry
1. Templates
1. cmake
1. scripts/o3de
1. Code/Legacy
1. Code/LauncherUnified
1. Code/Tools/AssetProcessor? (Should SIG-Content own this completely)
1. Code/CrashHandler
1. Gems/CrashReporting
1. Tools/EventLogTool
1. engine.json

@lumberyard-employee-dm See who owns the following directories/files
- [Tools/SerializeContextAnalyzer/](https://github.com/o3de/o3de/tree/development/Tools/SerializeContextAnalyzer/)
- [SerializeContextAnalysis.bat](https://github.com/o3de/o3de/blob/development/SerializeContextAnalysis.bat)
- [Code/Tools/SerializeContextTools/](https://github.com/o3de/o3de/tree/development/Code/Tools/SerializeContextTools/)

Add o3de repo ticket to remove remove `*.cfg` files from the repo root

# Agenda Items not discussed
The first item on the Agenda was not discussed: https://github.com/o3de/sig-core/issues/30
@kberg-amzn is required for that discussion

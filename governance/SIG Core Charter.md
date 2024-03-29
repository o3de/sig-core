# SIG Core Charter 

This charter adheres to the Roles and Organization Management specified in "[sig-governance.md](https://github.com/o3de/community/tree/main/sigs)".

Team information may be found in the "[README.md](https://github.com/o3de/sig-core/blob/main/README.md)"

# Overview of SIG #

Two concise lines explaining what this SIG does with bullet points of the major responsibilities

- Maintains and updates Core O3DE Systems: Gem (Plugin System), Serialization, Engine Initialization and Configuration, Settings,
- Provides Memory Allocator Framework and Metrics Logging system

# Goals #

- Extend the Gem (Modular Framework) plugin system which allows developers to easily add libraries/applications into O3DE
- Reduce Developer Iteration Time through better organization of code through the Gem and Project Templates
   - Provide framework via the o3de template system as well as guidance on to properly separate public API vs private implementation to help reduce build times
 - Gather performance metrics on Core Applications' startup to come with a strategy to improve runtime performance(Reduce launch time to enter Editor, reduce assets loading time)

# Scope 

* Responsible for the keybind and controller framework system
* Design and implement localization framework for editor and project runtime.
* Publish and maintain localization data format structure

* Maintain behavior context, edit context, serialization contexts, and code reflections frameworks and systems.

* Create framework for exposing profiling and metrics data that can be collected 
* Maintain generic node based scripting framework and data representation model 

* Maintain asset catalog, asset processor, and builder systems and framework 
* Maintain Packaging artifact and catalog system
* Design and Maintain asynchronous loading stream system

* Maintain Prefab system 
* Maintain Core system libraries AZCore, and AZFramework libraries
* Maintain Editor python bindings framework
* Maintain Logging and Trace systems and frameworks

* Publish and maintain list of use case examples for each subsystem of AZCore. 

# Generalized overall scope of work 

## In scope


## Cross-cutting Processes

* Support and collaborate with all SIGs in relation to changed and updates to underlying frameworks 
* Publish procedure for the intake of requests from SIGs in relation to changes and needs to core systems
* Provide consultation, discovery, and guidance for new feature support brought forth by other SIGs
* Publish and maintain best practices, usability examples, and feature documentation.
* Collaborate with SIG-Simulation for updates to core Math library and Tick System


## Out of Scope ##

 * Not responsible for building bespoke solutions to meet individual needs, but may be consulted with from time to time without obligation.


# SIG Links and lists: 

- Joining this SIG
- Slack/Discord
- Mailing list
- Issues/PRs
- Meeting agenda & Notes

# Roles and Organization Management 

SIG-Core adheres to the standards for roles and organization management as specified by [SIG-Governance](https://github.com/o3de/community/tree/main/sigs). This SIG opts in to updates and modifications to [SIG-Governance](https://github.com/o3de/community/tree/main/sigs)

## Individual Contributors 
Must provide a report of performance and blast radius impact of direct and indirectly affected systems

Additional information not found in the [sig-governance](https://github.com/o3de/community/tree/main/sigs) related to contributors.

## Maintainers 
 
Additional information not found in the [sig-governance](https://github.com/o3de/community/tree/main/sigs) related to contributors

## Additional responsibilities of Chairs 

Additional information not found in the [sig-governance](https://github.com/o3de/community/tree/main/sigs) related to SIG Chairs

## Subproject Creation 

Additional information not found in the [sig-governance](https://github.com/o3de/community/tree/main/sigs) related to subproject creation

## Deviations from [sig-governance](https://github.com/o3de/community/tree/main/sigs) 
None

## Explicit Deviations from the [sig-governance](https://github.com/o3de/community/tree/main/sigs) 

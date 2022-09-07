Facilitator: Pratik Patel

Meeting Notes: AMZN-Liv

# Agenda
https://github.com/o3de/sig-core/issues/1
## Introductions
*Presenters: Any/all*

* AMZN - Phil, koppersr, pratik, scottR, Esteban, tommy, moudgils, kberg, Liv, lumberyard-employee-dm, rgba16f, RoyalOBrien, Stephen Tramer, daimini 
* Gavin Freyberg, RoddieKieley, sergey, Dmytro Kovalchuk, dg211

## SIG Charter

*Presenter: Pratik Patel*

* The charter is posted [here](https://github.com/o3de/sig-core/blob/main/governance/SIG%20Core%20Charter.md)

**General questions**
* Who owns the compiler support (cross-cutting functionality)? 
    * General sentiment that it's proposed to move this responsibility to SIG-Platform

* Which SIG is scripting under? 
    * Currently under SIG-core, also should include non node-based programming as well; LUA is under SIG-content. There needs to be more clarity around scripting. 
        * Mention SIG-content also overlaps with SIG-content. Scripting/LUA core should be under Core   

* Should SIG-platform be part of SIG-core?
    * SIG Core should consult with SIG platform for cross-cutting API definition 
    * The main difference is organizational
    * Ideally, representatives from both SIGs will contribute to these fundamentals 

* Should SIG-Core own keybindings?
    * AZ::Core and AZ::Framework 
    * Core owns the base input API
        * Input should be pulled out of framework, should be SIG-presentation? 
    * Content owns the implementation      

 * What does Core represent?
    * One comment - API designs
    * Another comment - the collection of math libraries, data structures, essential stuff we don't want to copy across modules 
    * Additional comment - "It seems like core owns the shared code," but someone has to own that there are reference implementations that are easily available for everyone. 

* Who owns the core game loop? 
    * Tick process is AZ::Core; that's SIG-Core, might want to call that out in the charter scope 
    * Someone else notes that prefabs / entity-component system is also included in core game loop    

**Clarifications and additional notes**
 * Asynchronous load system - is this the foundational layer? Individual platforms may have different and bespoke solutions. 
    * O3DE has two layers of streaming, file and asset streaming. Planning to add a third layer (scheduling) to include base assets at specific times. File level system has a native implementation for Windows, with other platforms to follow. Other platforms have generic file streaming solutions. 
    * Is it this SIG's repsonsibility to work out the architecture and separation vs. shared components? Or implementation of the loaders as well?
        * File level is agnostic
        * Propose SIG-platform cover platform-specific implementations explicitly, clarify this in the charter  

* There needs to be a reference platform for O3DE; mentions that it should be POSIX in response to a suggestion that it's Windows 10
    * Clarifies that there needs to be references for target platforms and host platforms
    * Another reference suggestion is that a better reference is platform-agnostic 
    * Implementation is different (SIG-PLATFORM)
    * Integration with CORE and PLATFORM needs to be tight 

* PhysX mentioned as an example
    * Different game teams might want to swap out physics backend; the components aren't going to work with feature sets only with the generic API. A generic API would target the lowest common denominator. 
    * Would there be an AZ::Physics?
        * No, suggests Karl. Talks about how networking is all in a gem implementation. Suggests that Core is as narrow as possible to not limit how teams adopt the engine. Build system facilitates this. Two conversations there, then: who owns those gems? 

* General suggestion: Core shouldn't _only_ be Core and Framework; but may also own gems.
    * This would allow other SIGs to take ownership of pieces as it made sense 

## Meeting Cadence    
* Suggest having two meetings to address for various time zones, or have meeting between 7am - 11am Pacific 
* Might be worth having a sooner meeting than would be otherwise, since there is a lot to work out
    * Royal proposes a Doodle schedule for ~2 weeks out, post in SIG-CORE
    * Pratik will post schedule proposal for next meeting 

## Actions
* Propose SIG-platform own compiler via RFCs
* Clarify distinctly how scripting / behavior context is owned between content and core
    * Propose that the language and API implementation is Core, bindings and everything that sits on top of that are on content 
* Go through each line in the SIG charter to clarify ownership areas
* Identify what is currently in Core that should be broken out; should be a priority to help define what Core owns.

## Ongoing discussions
* What is the target for SIG-Core? Builds off of the conversation around PhysX. Do modules take responsibility for also authoring the APIs? What is the expectation for the core engine? There are pros and cons about the approaches between targeting a lowest common demoninator vs. the modular approach. Need to have further discussions. 

* What do we want to do in the first few months, vs. what we want to do long-term down the road? Before we can talk about the specifics around API design, the SIG-Core team will need to weigh in on architecture / reference. 


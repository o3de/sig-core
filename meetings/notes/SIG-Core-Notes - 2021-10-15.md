
## Agenda 1: splitting sig core into more sigs
* Ben agrees, would like to see sig-animation
* Dave thought animation with graphics/audio
* Dave in favour of separate sig for physics

        - sig-physics
        - sig-animation
        - sig-ai
            - sig-simulation (AI (future), Physics, Simulation)
        - sig-systems (core)
        - sig-scripting (both content and core)
            - Script Canvas, Lua, Python
        
* Question: (nemerle) Re: splitting sig/core: do we have a map of functionalities/processes we can assign/route to various sigs ?

* Scripting might need it's own sig ( if we have a roadmap for generic scripting support - new languages etc. )

* From @bosnichd: on the topic of splitting up sig-core, we talked a while ago about whether input should move to sig-platform, and I think the general consensus at the time was that it should, but I don't think it ever happened. The fact that all the discussions about adding mouse/keyboard/gamepad support for Linux are happening in sig-platform is good evidence that it should indeed probably be moved there

### Action items
* RFC to ratify a change to the sig (@amzn-pratik)
* Think of name to distinguish from Simulation Vertical that exists to minimize confusion




## Agenda 2: Integration of GSL into AzCore
    https://github.com/microsoft/GSL
    - span, not_null, better type safety

    - need to discuss with TSC
    - Compile times being too great?
    - RFC to put proposal across
    - concern about compiler support

## Agenda 3: Nomination of burelc_amzn to be a maintainer
    - maintainer promotion (approved)

## Agenda 4: Remove PhysX sample Gem
  
    - Should only check sample gems if they have really solid/good examples
        - where to put the files (LFS budget)
        - Adds to SDK budget 

# Other items

* Mentioned CryTimer change (amzn-sean) working on this
* Make sure everyone is aware of the changes

## C+20 support
* Do all platform tool chains support C+20 at the same time?
* AZStd would benefit from the new ranges library. From @nemerle: as for c++20 ranges being a new big feature, I'd agree it's a big one, but from what I've seen, the build-time impact is huge 🙂
* 3rd party packages and adding C+20 support may be challenging
* VS2022 support: wait until it's released before adding it to AR



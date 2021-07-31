# Core notes

## Mission statement

* SIG is a hub for other SIGs. Should handle coordination between them
* This SIG has a lot of cross-cutting dependencies but shouldn't be a dumping ground
* Heavily related to SIG-PLATFORM so perhaps CORE designs and PLATFORM implements?

@obwando to explore voting mechanisms that can be used with GitHub



## Coding Standards

* Fix formatting
* Need separate standards for each language used (C++, Python)
* Modifications should come through RFC's
* Do we need a fallback if something isn't covered by the standard? Perhaps start with telling people * to follow the format of the code that's alreay there.
* How strict should reviewers be for PR's? Should be some but not draconian?
* Use Clang tidy to detect and fix standards violations. Could (should?) be added to PR validation. Need to chat with Pipelines team about the feasibility of this.




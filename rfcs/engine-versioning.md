# Summary:
Currently there are large increments of time between each version of the engine correlating with an official release in main. While this does provide some course versioning, many developers are in branches incrementally working through each improvement as it is added to the engine. Without more granular versioning the user and other automated systems cannot differentiate between incremental development versions. This solves this issue by providing an incremental internal versioning number that provides specificity to the version than the official release version. From here the user along with automated systems can use this info to detect potentiality new conflicts and incompatibility introduced during development.

# What is the relevance of this feature?
This will provide a consistent and granular versioning system for impactful incremental changes to the engine. It can be used to detect the need for upgrades to compatible gems, assets and other data schemas after new change to the engine is pulled or installed. All of the changes that need to be made for a project to support the latest version of the engine can be determined based on the version difference from the project's last upgrade. This system can also be used to predict whether a gem is compatible with the engine, as gems can provide a range of compatible engines versions.

# Feature design description:
The current internal version has been kept at 0.0.0.0 since the launch of O3DE but with this change it will start being incremented whenever a potentially breaking change is pushed. Before being pushed it will need approval from sig-core.

The version will follow this pattern 0.main.main_patch.dev. The first 0 will not change until O3DE is officially out of beta so that will stay the same. The first main will be incremented whenever a stabilization branch is created for a new advertised release version such as 22.05.0. main_patch will increment for each patch to the current release version when a stabilization branch is created for that such as 22.05.1. And finally, dev will be incremented whenever a potentially breaking change such as a data schema change, or external API change is pushed to the development branch.

Note the advertised/named version uses a such as 22.05.0 uses a different versioning schema similar to Ubuntu versioning. The internal version will not use this schema as it makes it difficult to determine that relation between versions and a specify ranges as versions do not increment on fixed intervals.

The versions will be able to be compared to determine the difference so using that all of the upgraders necessary can be determined when upgrading projects. Specifying a range of supported versions will use the same formatting that python uses such as ^1.2.3.4 or 2.4.0 or >=3.4.

References: https://en.wikipedia.org/wiki/Software_versioning https://python-poetry.org/docs/dependency-specification/ https://ubuntu.com/about/release-cycle#:~:text=Version%20numbers%20are%20YY.MM,every%20two%20years%20in%20April.

# Technical design description:
A new field will be added to engine.json at the root the o3de directory called "EngineVersion". The field "O3DEVersion" already exists but is used for the advertised/named version number. This field will be used to track the current version of engine. To ensure this field is only updated when it should it will be it has been added to the .codeowners under sig-core to request a review from the group.

Python scripts for the o3de objects will be updated to allow setting the supported engine versions. Python scripts will also be added for retrieving the current engine version along with supported engine versions from different objects. Capacity for comparing engine versions and versions ranges must also be added to the API.

The Project Manager will also need to display the engine version by updating the handling for object data types. From there the new information for internal engine version will need to be displayed in the Project Manager UI along with the supported versions for projects and gems.

# What are the advantages of the feature?
Provides more granular versioning beyond individual releases
Allows gem creators to specify specific ranges of engine versions that gems are compatible with
Allows projects to be incrementally upgraded in development branches instead of per engine release version
What are the disadvantages of the feature?
Requires developers to understand when they are making potentially breaking change
Requires review from sig-release each time version is incremented
How will this be implemented or integrated into the O3DE environment?
This does not require any new libraries. It will expand on both C++ source code and o3de python CLI code.

# Are there any alternatives to this feature?
Two other similar approaches were considered but both ultimately selected against due to their reliance on git which not all customers will use as source control.

GitHub Tags
Utilize the GitHub tag system to mark commits that increment version.

Pros:

Does not require editing a file
Cons:

Requires access to git api from scripts checking this
Per repo, so the tags must be configured when merging from a fork and will be different
May require some form of approval to create new tags for a version
This versioning only works for customers using git for source control
Version Per Commit
Each commit is considered a separate incremental version.

Pros:

Automatic versioning
Does not require editing a file
Cons:

Requires access to git api from scripts checking this
Commits can be squashed or merged from other branches changing hashes so they are not stable until in development
This versioning only works for customers using git for source control

# How will users learn this feature?
The version of the engine along with the current version of projects as well as the supported versions for a gem will all be displayed within the Project Manager for users to see.

Upgrading projects will be added as a feature utilizing this system and automatically detecting potential upgrades and presenting them to the user.


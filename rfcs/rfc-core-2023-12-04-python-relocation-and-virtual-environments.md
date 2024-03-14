Current status
==============
Approved

Credits: This document was created by [@spham-amzn](https://github.com/spham-amzn)

### Summary:

The download and installation of Python should be moved out of the engine and into a package outside of the engine root, and subsequent uses of Python should be done from a [virtual environments](https://docs.python.org/3.10/library/venv.html?highlight=virtual%20environment) instead of directly in the downloaded package.

### What is the relevance of this feature?

Moving Python out of the engine path and into the `LY_3RDPARTY_PATH` with the other 3rd Party packages will provide the following benefits:

* Makes the location of the Python package consistent with the rest of the downloaded 3rd Party packages.
* Moving it out of the engine root makes the engine folder consistent with the github source. Python no longer needs to be explicitly added to the .gitignore file.
* Installers that rely on the engine being immutable (i.e. [SNAP](https://snapcraft.io/)) no longer need to download and package Python along with the rest of the engine binaries into the installer, reducing the size of the installer.

Python virtual environments provides a mechanism to isolate different packages and libraries for a specific application from other applications. Since virtual environments creates a small layer on top of the main Python package, the overhead is minimal. Switching O3DE to use Python virtual environments will add the following benefits:

* The Python library can be shared across multiple copies of O3DE that is installed locally. The same Python version and revision will only be downloaded and unpackaged once. 
* The validation hash against the package will still be valid across the lifetime of the package. All additional modules that are installed via pip will be done in the virtual environment, the Python package will not be altered.

### Feature design description:

This feature is a change to how O3DE uses Python, and the updates will have minimal impacts on development workflows that do not involve Python directly. 

* The current process to bootstrap Python will be updated to download and unpack Python in the common O3DE 3rd Party folder.
* For each unique engine on the local system, a Python virtual environment will automatically be created for it.
* The o3de-specific Python script ([Python.cmd](https://github.com/o3de/o3de/blob/607c548791f2170b4d23a8cac191be1b834ca15b/Python/Python.cmd)/[Python.sh](https://github.com/o3de/o3de/blob/607c548791f2170b4d23a8cac191be1b834ca15b/Python/Python.sh)) will be updated to refer to Python in the virtual environment.
* All targets that use the Python bindings 3rd Party library (ProjectManager, EditorPythonBindings) will be updated to use the virtual environment.

### Technical design description:

**Python 3rd Party Package**
A new revision of the Python 3rd Party package is not necessary. The Python virtual environment module is part of the standard Python library. The pip modules are already part of the package as well. The handling of the 3rd Party Package will change in the following ways:

* Currently the Python 3rd Party package is downloaded and extracted into `<Engine Root>/Python/runtime/$PYTHON_PACKAGE_NAME` where `<Engine Root>` is the location of the engine (either from github or an installer package). The location will change to `$LY_3RDPARTY_PATH` (inside of the `$HOME/.o3de`/`%USERPROFILE%\.o3de` folder)
* The package currently discards the original downloaded compressed package after it has been expanded into its final location. This will change such that this package will be treated the same as all of the other 3rd Party Packages: The downloaded compressed package will be retained in the `downloaded_packages` folder, and the package will be validated against its hash during every cmake generation run.

**Bootstrap Process**

The initial bootstrap process for Python will be updated to include the creation of the Python virtual environment, and the subsequent O3DE specific installation of modules will use the `venv` instead of direct Python calls in the 3rd Party Package. The target location of the virtual environment will be unique to the engine path from where the bootstrap process is occuring. O3DE will use the full absolute path of the current engine to generate a reasonable unique identifier. This path will be deterministic based on the engine path.

Below is the current bootstrap flow for O3DE:

1. Download and unpack the Python 3rd Party Package into `$O3DE/Python/runtime` if needed.
2. Only perform the [package validation](https://github.com/o3de/o3de/blob/2314dd188458dc35322daef4b87d67d820ea6517/cmake/LYPython.cmake#L236-L238) on the first time download, not on subsequent `cmake` project generation calls. Prevent normal 3rd party package hash validation. This is due to the fact that the subsequent calls to `pip` into the 3rd Party package will alter the package contents, and thus the package hash will be different.
3. Perform `pip install` using the O3DE Python script under `$O3DE/Python` to perform the installation of the following:
    * General packages defined in $O3DE/Python/requirements.txt
    * Packages from `$O3DE/Tools/LyTestTools`
    * Packages from $O3DE/Tools/RemoteConsole/ly_remote_console
    * Packages from $O3DE/AutomatedTesting/Gem/PythonTests/EditorPythonTestTools

The updated bootstrap flow for O3DE will be:

1. Download and unpack the Python 3rd Party Package into `$LY_3RDPARTY_PATH` if needed.
2. Perform the standard package validation against the package hash.
3. Calculate the full path (`$PYTHON_VENV_PATH`) to the Python virtual environment based on the absolute path of the engine. The full path will be:
   `$HOME/.o3de/Python/$ENGINE_ID/` where `$ENGINE_ID` will be the first 8 hexadecimal digits of the SHA-1 hash of the absolute path of the current engine.
4. Check if the `$PYTHON_VENV_PATH` path exists. If it exists, check the following scenarios:
    * Check if the expected Python binaries (by platform) exists.
    * Check if the expected `pyvenv.cfg` exists.
    * Check if a 3rd Party hash marker file `.hash` exists.
    * If the `.hash` exists, check if it matches the package hash of the 3rd Party Python package the virtual environment is meant for.
   If one or more of the above check fails, then check against a cmake variable `O3DE_ERROR_ON_PYTHON_HASH_MISMATCH` to determine if we want to clear out the venv and regenerate it, or just report a fatal error with instructions on how to re-generate the venv.

5. If `$PYTHON_VENV_PATH` does not exist, or it was cleared out by the above validation checks, then perform the creation of the virtual environment based on the following command (`$PYTHON_EXECUTABLE` will be the Python executable inside the Python 3rd Party package, and its sub folder is platform specific. )
    `$PYTHON_EXECUTABLE -m venv $PYTHON_VENV_PATH`
   Once the environment is initialized to `$PYTHON_VENV_PATH`, write the package hash for the Python 3rd Party package to the folder to `.hash`.
6. Perform `pip install` using the O3DE Python script under `$O3DE/Python` to perform the installation of the following:
    * General packages defined in $O3DE/Python/requirements.txt
    * Packages from `$O3DE/Tools/LyTestTools`
    * Packages from $O3DE/Tools/RemoteConsole/ly_remote_console
    * Packages from $O3DE/AutomatedTesting/Gem/PythonTests/EditorPythonTestTools
   The O3DE Python will be updated to use the Python virtual environment for the engine instead (see 'Python script updates')

**Python PAL-ification**

The current Python bootstrap script does not employ the Platform Abstraction Layer (PAL) pattern for the Python packages since it is used in both generation and script modes. In script mode, the main [LYPython.cmake](https://github.com/o3de/o3de/blob/development/cmake/LYPython.cmake) does not have access to many of the PAL variables that is available during the generation mode, so it currently does a manual check against the current platform (and architecture) to determine the package name, hash, etc. 

In order to refactor the script to follow the PAL pattern, the PAL variables that are needed will instead be initialized by the current `get_python.*` scripts instead. [get_Python.bat](https://github.com/o3de/o3de/blob/development/Python/get_Python.bat) is guaranteed to run on Windows only, so the windows specific PAL variables can be hardcoded in that script. [get_Python.sh](https://github.com/o3de/o3de/blob/development/Python/get_Python.sh) is valid for both Linux and Mac, so platform detection (as well as architecture detection for Linux) will be handled there. Since this is a BASH script, it can detect the platform by using the `$OSTYPE` environment variable.

These `get_Python.*` scripts subsequently calls the [get_python.cmake](https://github.com/o3de/o3de/blob/10e664f571e0f8f70eec18fae8921ea073e3a882/python/get_python.cmake), which provides the necessary `PAL*` related variables needed and passes it to [LYPython.cmake](https://github.com/o3de/o3de/blob/development/cmake/LYPython.cmake) to run through the same bootstrap process as a cmake project generation workflow.

The platform specific information for the Python package will be moved to PAL'ified files `cmake/3rdParty/Platform/{$PLATFORM_NAME}/Python_{$PLATFORM_LOWER}.cmake`.

**Python Script Updates**

Since PAL-ification is handled in the either the `get_Python.*` scripts or part of the cmake generation workflow, the location of the Python virtual environment for the engine will be set to a known location based on the current engine rather than trying to detect the platform and the location of the actual Python 3rd Party package. The current [Python.cmd](https://github.com/o3de/o3de/blob/607c548791f2170b4d23a8cac191be1b834ca15b/Python/Python.cmd)/[Python.sh](https://github.com/o3de/o3de/blob/607c548791f2170b4d23a8cac191be1b834ca15b/Python/Python.sh) will updated to generate the deterministic path to the Python virtual environment `$PYTHON_VENV_PATH` by performing the same logic employed by the bootstrap workflow.

Instead of running `Python` from the 3rd Party package, it will instead run through the execution flow for virtual environments:

1. Run the `activate` script within the `venv` to setup the proper environment
2. Run the `Python` executable within the `venv`
3. Run the `deactivate` script (Windows) to restore the environment

The `activate`/`deactivate` is necessary to set up and tear down the virtual environment. (Only activate is needed on BASH since it is running in its own shell)

**Embedded Python Updates**

The targets that depend on the [Pybind 3rd Party Package](https://github.com/o3de/3p-package-source/tree/main/package-system/pybind11) ([Project  Manager](https://github.com/o3de/o3de/tree/development/Code/Tools/ProjectManager) and the [Editor Python Bindings Gem](https://github.com/o3de/o3de/tree/development/Gems/EditorPythonBindings)) will also need to update its environment to use the virtual environment. The `pyvenv.cfg` file is only used when running the Python interpreter that is inside the virtual environment. With embedded Python, however, we will need to read in this file and set the `PYTHON_HOME` to the 3rd Party Python library. In addition to initializing the Python interpreter from pybind, we will need to scan the virtual environment's site-packages to look for `*.egg-link` files. These files tell the Python interpreter where to look for additional modules in other folders. [Pybind11](https://github.com/pybind/pybind11) has trouble interpreting these files, so we will need to work around this issue by scanning and manually adding the paths that are contained in these `egg-link` files into the `$PYTHON_PATH`` environment.

### What are the advantages of the feature?

* Removes the one-off location of the 3rd Party packaging for Python. Python is now treated the same as the other 3rd Party Packages.
* Prevents changes / updates to the extracted Python 3rd Party package. This allows for package integrity hash checks beyond just the initial download.
* Moving the package outside of the engine root helps the engine folder maintain consistency. The Python runtime can be removed from the source `.gitignore` file.
* Linux [SNAP](https://snapcraft.io/) packages rely on the installed engine to run from an immutable path/system. This made it impossible to have a SNAP installed version of O3DE download Python as needed. To get around this issue, the SNAP packaging process downloads and installs Python and all of its pip modules into the local `Python/runtime` folder, and then packages it in the SNAP container. The disadvantages of this are:
    * Larger snap packages to download.
    * Python resides in an immutable storage. It is not possible to add (pip-install) and new modules. 
* Cleaning the Python environment can be done by simply wiping out the generated virtual environment instead of removing the engine local runtime and re-downloading the entire package.
* Multiple instances of the engine on the same machine can share the same Python 3rd Package (granted they are on the same version and revision). Their engine-specific packages will instead be stored in their own virtual environment.
* Protects against any system `PYTHONPATH` injection when running Python. For instance, the [ROS ecosystem](https://docs.ros.org/en/humble/index.html) installs its own Python and injects ROS-specific Python packages into `PYTHONPATH`. 

### What are the disadvantages of the feature?

* The setup and teardown for the virtual environment for every Python call may incur a minor performance hit (as opposed to running Python directly)
* There is no mechanism to clean up generated virtual environments other than manually deleting them. 
* Introduces risks of dangling virtual environments if the Python 3rd Party package is removed but the virtual environment is not.

### How will this be implemented or integrated into the O3DE environment?

The implementation is described in the technical design description above. For source versions of the engine, the updated scripts will perform the bootstrap process as needed to setup the Python environment properly. The legacy Python runtime will still exist in the engine path and may be removed manually. 

### Are there any alternatives to this feature?

* Python 3rd Party can be moved externally without using virtual environments
    We can skip using virtual environments and just directly follow the same scenario where the package content is updated directly. This will re-introduce the work around where package validation is only done once when the package is initially downloaded. This also means that the 
    location of the 3rd party binary will no longer be deterministic for the engine and the path needs to be hardcoded or generated somewhere in 
    the engine in order to access it. 
* Use alternatives to Python Virtual Environments:
    * [Conda](https://docs.conda.io/en/latest/) provides package environment management for any language.
    * [Pipenv](https://pypi.org/project/pipenv/) provides management for pip, venv for Python beyond the default support with venv.

    Python Environments was chosen because it is provided by default by Python is does not require additional packages/dependencies.

### How will users learn this feature?

The bootstrap process will occur automatically, and all Python calls in O3DE currently are wrapped with O3DE python script files already. Users will learn of this update through the release notes and impactful change messaging from O3DE.

### Are there any open questions?

* **What happens to the virtual environment when there is an update to the 3rd Party Python package? (ie security update)**
    The workflow attempts to remedy this scenario by keeping the package hash within the generated virtual environment folder. If there a detected change in the hash, the bootstrap process will wipe out the virtual environment contents and force a generation of a new virtual environment based on the updated 3rd Party Python.
  
* **Why is the virtual environment stored in the `.o3de` folder?**
    We need a location that is outside of the engine path, and the `.o3de` folder already manages all the 3rd Party packages. Alternatively the location could be stored in the `$HOME/O3DE` folder instead, but since it has a dependency on a specific 3rd Party Python package, it made more sense to place it there.

* **Will this solve individual projects from injecting python libraries globally through pip-install?**
    The virtual environment scope is still at the engine level, not the project level, so this will not solve that issue. It does protect the 3rd Party Python site packages however since the project-injected python libraries are installed into the virtual environment, and not the 3rd Party Python packages.


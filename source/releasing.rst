Releasing
=========

This chapter explains how to package the BAMM executable program
for Mac OS X and Windows, so that users can download it from the website.
It is assumed that Linux users will want to compile BAMM themselves,
so Linux users are not provided with an executable program.

We generate releases from the *master* branch.
Therefore, before starting the release process,
make sure that all changes to *master* have been committed
and pushed to the GitHub repository.


Update changes.rst
------------------

Important changes from one release to another
are documented in *doc/source/changes.rst*,
which is converted to an HTML file when generating the documentation.
Update this file with the changes made since the last release.
To help you see these changes, use the *git log* command as follows::

    git log --oneline v2.1.0..HEAD

where ``v2.1.0`` is the version of the last release.
This command will display all the commit messages
since version 2.1.0 up to the present (i.e., ``HEAD``).

You must choose a version number of the current release
in order to provide a title for the new change entry.
Please format your own entry
using the same conventions used in previous change entries.
Generate the HTML documentation and make sure the new change entry looks OK.
If everything looks good, commit the new changes to the repository.


Update version number
---------------------

Each BAMM release should have its own version number.
This version number is displayed every time BAMM is run
or when the ``--version`` option is used.
Before releasing BAMM, provide the new version number
in the ``CMakeLists.txt`` file, located in the BAMM root directory:

#. Open ``CmakeLists.txt``.

#. Starting on line 4, there are several ``SET`` commands
   that specify the current version of BAMM:
   ``BAMM_VERSION_MAJOR``, ``BAMM_VERSION_MINOR``, and ``BAMM_VERSION_PATCH``.

#. Update the these version numbers appropriately.

#. There is also a ``BAMM_VERSION_DATE`` where you specify
   the date for this version release.

It is recommended that the *major* version number is changed
for large changes to BAMM, such as those that make it backwards incompatible.
The *minor* version number should be changed for new features or bug fixes
that do not compromise the compatibility with previous versions of BAMM.
The *patch* version number should be changed for small updates or bug fixes
that do not break the compatibility with previous BAMM versions.

Compile BAMM and then run ``./bamm --version``
to make sure the new version is correct.
Commit the change made in ``CMakeLists.txt``, and tag this commit
(the following example assumes that the new version is 2.2.0)::

    git tag -m "Release version 2.2.0" v2.2.0

Remember to *push* these changes to GitHub before continuing.
Git does not automatically push tags to the remote repository,
so you must run the following command to push them::

    git push --tags


Mac OS X
--------

To create an executable file for Mac OS X,
BAMM must be compiled on a 64-bit computer with Mac OS X 10.7.5.
This will allow users with OS X 10.7.5 or greater run the executable.

#. On Mac OS X 10.7.5, open the Terminal application.

#. If you do not have BAMM downloaded from GitHub, do so now.
   Otherwise, update the local repository using ``git pull``.

#. Checkout the tagged version (e.g., *v2.2.0*) you would like to release::

       git checkout v2.2.0

#. If you have a *build* directory in BAMM's root directory, remove it::

       rm -rf build

#. Create and go into a new *build* directory::

       mkdir build
       cd build

#. For releases, BAMM must be built using Xcode.
   Create an Xcode project automatically using CMake::

       cmake -GXcode ..

#. Open the project in Xcode from the command-line::

       open BAMM.xcodeproj

#. Click on *BAMM* in the left-hand side panel of Xcode.
   Options will appear in the center panel.

#. To the left of the center panel, there is a list under the *TARGET* heading
   (*ALL_BUILD*, *ZERO_CHECK*, *bamm*, *install*, and *package*).
   Click on *bamm*.

#. In the panel with options, scroll down to a section called
   *Apple LLVM compiler 4.2 - Language*.
   Find the option *C++ Standard Library* and change its value
   from *Compiler Default* to *libc++*.

#. In the menu, select *Project*, then click on *Build*.

#. If the build succeeded, a directory called *Debug* will be created
   within the *build* directory, containing the executable *bamm* file.

#. Go into the *Debug* directory and compress the executable for distribution::

       cd Debug
       tar -czf bamm-2.2.0-MacOSX.tar.gz bamm

#. Upload this compressed file somewhere online that can be accessed by anyone.


Windows
-------

To create an executable for Windows, BAMM must be compiled on Windows 7
using Visual Studio (e.g., Visual Studio Express 13).
Rather than having a separate computer running Windows,
you may have a virtual box running Windows.

#. In Windows, open the command-line terminal
   (Click on Start, then search for "cmd").

#. If you do not have BAMM downloaded from GitHub, do so now
   (the following instructions assume that BAMM
   is in a directory called ``bamm``).
   Otherwise, update the local repository using ``git pull``.

#. If you have a previous ``build`` directory, remove it::

       rmdir build /s /q

#. Checkout the tagged version (e.g., v2.2.0) you would like to release::

       git checkout v2.2.0

#. Create a new ``build`` directory and go into it::

       mkdir build
       cd build

#. Run ``cmake`` with the Visual Studio generator::

       cmake -G"Visual Studio 12" ..

#. Run Visual Studio Express 13 and open the BAMM project
   located in the ``build`` directory (called *BAMM*).

#. On the toolbar, change *Debug* to *Release*.

#. On the right-hand panel, right-click on *bamm* and click on *Build*.
   If the build succeeded, a directory called *Release* will be created
   within the *build* directory, containing the executable *bamm.exe* file.

#. On the Desktop (or somewhere convenient),
   create a new folder named *bamm-<version>-Windows*,
   where <version> is the release version of BAMM.

#. Copy the *bamm.exe* file from the *build\Release* directory
   into the new folder *bamm-<version>-Windows*.
   For example, if BAMM is located in *C:\\Users\\Auto\\bamm*
   and you are in the *build* directory, you may copy *bamm.exe* as follows::

       copy Release\bamm.exe C:\Users\Auto\Desktop\bamm-2.2.0-Windows

#. Copy the required *DLL* files from Visual Studio
   to the *bamm-<version>-Windows* folder (these files are located in
   *C:\\Program Files (x86)\\Microsoft Visual Studio 12.0\\VC\\redist\\x86\\Microsoft.VC120.CRT*).

#. Right-click on the *bamm-<version>-Windows* folder,
   select *Send to*, then *Compressed (zipped) folder*.
   This will create a new file called
   *bamm-<version>-Windows.zip*.

#. Upload this compressed file somewhere online that can be accessed by anyone.

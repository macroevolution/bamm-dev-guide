Releasing
=========

This chapter explains how to package the BAMM executable program
for Mac OS X and Windows, so that users can download it from the website.
It is assumed that Linux users will want to compile BAMM themselves,
so Linux users are not provided with an executable program.


Version number
--------------

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

Commit the change made in ``CMakeLists.txt``, and tag this commit
(the following example assumes that the new version is 2.2.1)::

    git tag -m "Release version 2.2.1" v2.2.1


Mac OS X
--------

Blah.


Windows
-------

To create an executable for Windows, BAMM must be compiled on Windows
(Windows 7 and above) using Visual Studio (e.g., Visual Studio Express 13).
Rather than having a separate computer running Windows,
you may have a virtual box running Windows.

#. Open the command-line terminal
   (Click on Start, then search for "cmd").

#. If you do not have BAMM downloaded from github, do so now
   (the following instructions assume that BAMM
   is in a directory called ``bamm``).

#. If you have a previous ``build`` directory, remove it::

       rmdir build /s /q

#. Make sure your repository is updated (run ``git pull``),
   then go to the correct commit entry for this release.
   For example, if you are releasing version 2.1.0 and you have tagged it,
   checkout that tag::

       git checkout v2.1.0

#. Create a new ``build`` directory and go into it::

       mkdir build
       cd build

#. Run ``cmake`` with the Visual Studio generator::

       cmake -G"Visual Studio 12" ..

#. Run Visual Studio Express 13.

#. Open the BAMM project located in the ``build`` directory.

#. On the toolbar, change "Debug" to "Release".

#. On the right-hand panel, right-click on "bamm" and click on "Build".

#. On the Desktop (or somewhere convenient),
   create a new folder named "bamm-<version>",
   where <version> is the release version of BAMM.

#. Copy the ``bamm.exe`` file from the ``build\Release`` directory
   into the new folder bamm-<version>.

#. *TODO: Say which DLL files are needed and where to get them*

#. Compress this folder into a ZIP file.

#. Upload this ZIP file somewhere that can be accessed by anyone.

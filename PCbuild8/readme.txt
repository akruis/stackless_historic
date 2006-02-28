Building Python using VC++ 8.0
------------------------------
This directory is used to build Python for Win32 and win64 platforms.
It requires Microsoft Visual C++ 8.x.

All you need to do is open the workspace "pcbuild.dsw" in MSVC++, select
the Debug or Release setting (using Build -> Set Active Configuration...),
and build the projects.

At the moment, the major purpose of tjis folder is to support 64 bit
builds. The output of a build will appear in a subfolder x64.
You can use it to build the standard win32 version as well.
But note that support for both versions of this compiler is limited,
and there are some tests not passing, due to incompatible library changes.

For further comments on windows builds, see the PCbuild folder.

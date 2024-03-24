# Crosscompiled-Windows-installer

Create a Windows installer for wxMaxima only (not combined with Maxima, Gnuplot, ...).

This repository is used to crosscompile a wxMaxima installer for Windows on Linux.
Similar to the procedure for Maxima, but only a wxMaxima-only installer
(not combined with Maxima, Gnuplot, ...)

The following options are possible:

`-DBUILD_64BIT=NO` (by default a 64 bit installer will be build)

`-DBUILD_WXMAXIMA_GIT=YES` (by default the installer will download a wxMaxima release.
(Version number / MD5 download checksum are defined in the main CMakeLists.txt.
The wxWidgets version is can be changed in wxwidgets/CMakeLists.txt

`-DWITH_WXMAXIMA_SOURCE=<path-to-source>`
Set the path to the (local) wxMaxima sourcecode tree, which will be used instead of
a release or the current Git version.


To build a installer, run:
```
mkdir build
cd build
cmake .. # maybe with some of the options above
make
make package
```

You will need the `g++-mingw-w64-x86-64` (crosscompiler).


To create a Release-Build (higher optimization, no debug info (smaller executable)), use
```
cmake -DCMAKE_BUILD_TYPE=Release <path-to-source>
```
instead.

The MinGW compiler comes in two flavors for threading (win32 and posix threads).
wxMaxima requires posix threads, so you must reconfigure mingw and select the posix
version, on Debian/Ubuntu Linux using:

`update-alternatives --set x86_64-w64-mingw32-g++ /usr/bin/x86_64-w64-mingw32-g++-posix`

Wolfgang Dautermann


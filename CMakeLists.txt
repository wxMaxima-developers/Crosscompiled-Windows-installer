# CMake file for crosscompiling wxMaxima for Windows
# Copyright (C) by Wolfgang Dautermann
# License GPLv2+: GNU GPL version 2 or later <http://gnu.org/licenses/gpl.html>
# This is free software: you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.


# Expects a crosscompiled wxWidgets library in $HOME/wxWidgets-3.1.0
set(CMAKE_SYSTEM_NAME Windows)

set(WXMAXIMAVERSION "21.01.0")

set(WXMAXIMA_MD5 "2c9b97c786806172aa6ed000784e995e")

option(BUILD_64BIT "Build a 64 bit installer." YES)

if(BUILD_64BIT)
    set(HOST x86_64-w64-mingw32)
    set(INSTALLERSUFFIX "win64")
else()
    set(HOST i686-w64-mingw32)
    set(INSTALLERSUFFIX "win32")
endif()

set(CMAKE_C_COMPILER   ${HOST}-gcc)
set(CMAKE_CXX_COMPILER ${HOST}-g++)
set(CMAKE_RC_COMPILER  ${HOST}-windres)
project(wxmaxima)
cmake_minimum_required(VERSION 3.7)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wextra")
set(wxWidgets_ROOT_DIR "${CMAKE_BINARY_DIR}/wxwidgets/wxwidgets-prefix/src/wxwidgets-build")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/ ${CMAKE_BINARY_DIR}/)


# Where are libgcc and libstdc++?
if(BUILD_64BIT)
    execute_process(COMMAND "${CMAKE_CXX_COMPILER}" -print-file-name=libgcc_s_seh-1.dll OUTPUT_VARIABLE MINGW_LIBGCC    OUTPUT_STRIP_TRAILING_WHITESPACE)
else()
    execute_process(COMMAND "${CMAKE_CXX_COMPILER}" -print-file-name=libgcc_s_sjlj-1.dll OUTPUT_VARIABLE MINGW_LIBGCC    OUTPUT_STRIP_TRAILING_WHITESPACE)
endif()

execute_process(COMMAND "${CMAKE_CXX_COMPILER}" -print-file-name=libstdc++-6.dll     OUTPUT_VARIABLE MINGW_LIBSTDCPP OUTPUT_STRIP_TRAILING_WHITESPACE)

message(STATUS "Found libstdc++ at ${MINGW_LIBSTDCPP}")

message(STATUS "Found libgcc at ${MINGW_LIBGCC}")

include(ExternalProject)


add_subdirectory("wxwidgets")
add_subdirectory("wxmaxima")


# Include DLLs from the current Mingw environment.
install(FILES "${MINGW_LIBSTDCPP}" "${MINGW_LIBGCC}" DESTINATION bin)

set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
set(CPACK_GENERATOR "NSIS;ZIP")
set(CPACK_PACKAGE_VERSION "${WXMAXIMAVERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "wxMaxima")
set(CPACK_PACKAGE_VENDOR "wxMaxima Team")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/wxmaxima/README.md")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/wxmaxima/GPL.txt")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "wxMaxima-${CPACK_PACKAGE_VERSION}")
set(CPACK_PACKAGE_FILE_NAME "wxMaxima-${CPACK_PACKAGE_VERSION}-${INSTALLERSUFFIX}")
set(CPACK_PACKAGE_ICON "${CMAKE_SOURCE_DIR}/wxmaxima.bmp")
#set(CPACK_NSIS_INSTALL_ROOT "C:")
set(CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL ON)
set(CPACK_NSIS_URL_INFO_ABOUT "https://wxmaxima-developers.github.io/wxmaxima/")
set(CPACK_NSIS_MENU_LINKS "bin/wxmaxima.exe" "wxMaxima" "https://wxmaxima-developers.github.io/wxmaxima/" "About wxMaxima")
set(CPACK_NSIS_COMPRESSOR "/SOLID lzma")

# #File associations:
# set(CPACK_NSIS_DEFINES "!include ${CMAKE_SOURCE_DIR}\\\\FileAssociation.nsh")
# set(CPACK_NSIS_EXTRA_INSTALL_COMMANDS "
# \\\${registerExtension} \\\"\\\$INSTDIR\\\\wxMaxima\\\\wxmaxima.exe\\\" \\\".wxm\\\" \\\"wxMaxima Document\\\"
# \\\${registerExtension} \\\"\\\$INSTDIR\\\\wxMaxima\\\\wxmaxima.exe\\\" \\\".wxmx\\\" \\\"wxMaxima XML Document\\\"
# \\\${registerExtension} \\\"\\\$INSTDIR\\\\wxMaxima\\\\wxmaxima.exe\\\" \\\".mac\\\" \\\"Maxima Batch file\\\"
# WriteRegStr HKCR \\\".wxmx\\\\ShellNew\\\" \\\"NullFile\\\" \\\"\\\"
# ")
# set(CPACK_NSIS_EXTRA_UNINSTALL_COMMANDS "
# \\\${unregisterExtension} \\\".wxm\\\" \\\"wxMaxima Document\\\"
# \\\${unregisterExtension} \\\".wxmx\\\" \\\"wxMaxima XML Document\\\"
# \\\${unregisterExtension} \\\".mac\\\" \\\"Maxima Batch File\\\"
# DeleteRegKey HKCR \\\".wxmx\\\\ShellNew\\\"
# ")


include(CPack)

# CMake file for crosscompiling wxMaxima for Windows
# Copyright (C) by Wolfgang Dautermann
# License GPLv2+: GNU GPL version 2 or later <http://gnu.org/licenses/gpl.html>
# This is free software: you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.

set(CMAKE_SYSTEM_NAME Windows)

# An installer version, which is appended after the version to the installer name, e.g.
# if that is set to "a", the name might be: wxMaxima-21.04.0a-win64.exe
# if for one wxMaxima version an improved installer is released.
set(INSTALLER_VERSION "")


option(BUILD_64BIT "Build a 64 bit installer." YES)
option(BUILD_WXMAXIMA_GIT "Build current wxMaxima git version." NO)

if(BUILD_WXMAXIMA_GIT)
    set(WXMAXIMAVERSION "current")
    set(WXMAXIMA_URL "https://github.com/wxMaxima-developers/wxmaxima/archive/main.zip")
    set(WXMAXIMA_MD5 "")
else()
    set(WXMAXIMAVERSION "21.05.2")
    set(WXMAXIMA_URL "https://github.com/wxmaxima-developers/wxmaxima/archive/Version-${WXMAXIMAVERSION}.tar.gz")
    set(WXMAXIMA_MD5 "9904b96fd9acb0950c65d06dec7dd82c")
endif()

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
cmake_minimum_required(VERSION 3.17)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pedantic -Wextra")
set(wxWidgets_ROOT_DIR "${CMAKE_BINARY_DIR}/wxwidgets/wxwidgets-prefix/src/wxwidgets-build")

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/ ${CMAKE_BINARY_DIR}/)


find_file(NSISTEMPLATE_IN  NAMES "NSIS.template.in" PATHS "${CMAKE_ROOT}/Modules/" "${CMAKE_ROOT}/Modules/Internal/CPack")
file(READ "${NSISTEMPLATE_IN}" MYNSISTEMPLATE)
string(REPLACE "  RequestExecutionLevel admin" "  RequestExecutionLevel user" MYNSISTEMPLATE "${MYNSISTEMPLATE}")
file(WRITE ${CMAKE_BINARY_DIR}/NSIS.template.in "${MYNSISTEMPLATE}")
message(STATUS "Copied and patched ${CMAKE_ROOT}/Modules/NSIS.template.in to ${CMAKE_BINARY_DIR}/NSIS.template.in")

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
install(FILES "wxmaximaicon.ico" DESTINATION "share/wxMaxima")

# Add some metainformation to the installer, see
# https://nsis.sourceforge.io/Reference/VIProductVersion
# https://nsis.sourceforge.io/Reference/VIAddVersionKey
set(CPACK_NSIS_DEFINES "
  VIProductVersion \\\"${WXMAXIMAVERSION}.0\\\"
  VIAddVersionKey ProductName \\\"wxMaxima\\\"
  VIAddVersionKey CompanyName \\\"wxMaxima Team\\\"
  VIAddVersionKey LegalCopyright \\\"(c) wxMaxima Team - GPL 2.0\\\"
  VIAddVersionKey ProductVersion \\\"${WXMAXIMAVERSION}\\\"
  VIAddVersionKey FileVersion \\\"${WXMAXIMAVERSION}\\\"
  ")


set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
set(CPACK_GENERATOR "NSIS;ZIP")
set(CPACK_PACKAGE_VERSION "${WXMAXIMAVERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "wxMaxima")
set(CPACK_PACKAGE_VENDOR "wxMaxima Team")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/wxmaxima/README.md")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/wxmaxima/GPL.txt")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "wxMaxima-${CPACK_PACKAGE_VERSION}")
set(CPACK_PACKAGE_FILE_NAME "wxMaxima-${CPACK_PACKAGE_VERSION}${INSTALLER_VERSION}-${INSTALLERSUFFIX}")
set(CPACK_NSIS_MUI_HEADERIMAGE "${CMAKE_SOURCE_DIR}/wxmaxima.bmp")
set(CPACK_NSIS_INSTALL_ROOT "C:")
set(CPACK_NSIS_ENABLE_UNINSTALL_BEFORE_INSTALL ON)
set(CPACK_NSIS_URL_INFO_ABOUT "https://wxmaxima-developers.github.io/wxmaxima/")
set(CPACK_NSIS_MENU_LINKS "bin/wxmaxima.exe" "wxMaxima" "https://wxmaxima-developers.github.io/wxmaxima/" "About wxMaxima")
set(CPACK_NSIS_COMPRESSOR "/SOLID lzma")
set(CPACK_NSIS_MUI_WELCOMEFINISHPAGE_BITMAP "${CMAKE_SOURCE_DIR}/wxmaxima-installerimage.bmp")
set(CPACK_NSIS_INSTALLED_ICON_NAME "share/wxMaxima\\\\wxmaximaicon.ico")


include(CPack)

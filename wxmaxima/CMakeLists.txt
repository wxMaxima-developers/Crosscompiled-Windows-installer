# CMake file for crosscompiling wxMaxima for Windows
# Copyright (C) by Wolfgang Dautermann
# License GPLv2+: GNU GPL version 2 or later <https://www.gnu.org/licenses/old-licenses/gpl-2.0.html>
# This is free software: you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.

# If you want to use a updated version of a program,
# update the version number and the checksum.
# If no further patches are needed, you should get a
# updated setup-file automatically.


externalproject_add(wxmaxima
    URL "${WXMAXIMA_URL}"
    DOWNLOAD_DIR ${CMAKE_SOURCE_DIR}/downloads
    DOWNLOAD_NAME wxmaxima-${WXMAXIMAVERSION}.tar.gz
    URL_MD5 ${WXMAXIMA_MD5}
    DEPENDS wxwidgets
    # search wx-config here - do not use the host wx-config
    # had to set CMAKE_LIBRARY_PATH=/usr/${HOST}/lib - libraries for mingw are installed there, but CMake (>3.8)
    # tries to check all the libraries reported by wx-config, but does not check this directory...
    CMAKE_CACHE_ARGS "-DCMAKE_PROGRAM_PATH:PATH=${CMAKE_BINARY_DIR}/wxwidgets/wxwidgets-prefix/src/wxwidgets-build"
    CMAKE_ARGS -DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME} -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER} -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER} -DCMAKE_RC_COMPILER=${CMAKE_RC_COMPILER} -DCMAKE_LIBRARY_PATH=/usr/${HOST}/lib
    BUILD_COMMAND $(MAKE)
    INSTALL_COMMAND $(MAKE) install "DESTDIR=${CMAKE_BINARY_DIR}/installroot"
)

# install wxMaxima files
install(DIRECTORY ${CMAKE_BINARY_DIR}/installroot/usr/local/ DESTINATION .)

# wxMaxima uses C++ threads, include the required mingw thread library.
execute_process(COMMAND "${CMAKE_CXX_COMPILER}" -print-file-name=libwinpthread-1.dll OUTPUT_VARIABLE MINGW_LIBWINPTHREAD OUTPUT_STRIP_TRAILING_WHITESPACE)
message(STATUS "Found libwinpthread at ${MINGW_LIBWINPTHREAD}")

# Include DLLs from the current Mingw environment.
install(FILES "${MINGW_LIBWINPTHREAD}" DESTINATION bin/)


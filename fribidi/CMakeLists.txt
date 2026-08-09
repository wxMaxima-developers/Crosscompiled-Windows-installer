# CMake file for crosscompiling wxMaxima for Windows
# Copyright (C) by Wolfgang Dautermann
# License GPLv2+: GNU GPL version 2 or later <https://www.gnu.org/licenses/old-licenses/gpl-2.0.html>
# This is free software: you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.

# If you want to use a updated version of a program,
# update the version number and the checksum.
# If no further patches are needed, you should get a
# updated setup-file automatically.

set(FRIBIDI_VERSION "1.0.16")
set(FRIBIDI_MD5 "333ad150991097a627755b752b87f9ff")
set(FRIBIDI_URL "https://github.com/fribidi/fribidi/releases/download/v${FRIBIDI_VERSION}/fribidi-${FRIBIDI_VERSION}.tar.xz")

externalproject_add(fribidi
    URL "${FRIBIDI_URL}"
    DOWNLOAD_DIR ${CMAKE_SOURCE_DIR}/downloads
    DOWNLOAD_NAME fribidi-${FRIBIDI_VERSION}.tar.xz
    URL_MD5 ${FRIBIDI_MD5}
    BUILD_IN_SOURCE 1
    CONFIGURE_COMMAND ./autogen.sh
              COMMAND ./configure --host=${HOST} --prefix=${CMAKE_BINARY_DIR}/fribidi-installroot
  BUILD_COMMAND $(MAKE)
  INSTALL_COMMAND $(MAKE) install
)




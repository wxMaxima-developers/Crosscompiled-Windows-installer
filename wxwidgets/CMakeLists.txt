# CMake file for crosscompiling wxMaxima for Windows
# Copyright (C) by Wolfgang Dautermann
# License GPLv2+: GNU GPL version 2 or later <https://www.gnu.org/licenses/old-licenses/gpl-2.0.html>
# This is free software: you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.

# If you want to use a updated version of a program,
# update the version number and the checksum.
# If no further patches are needed, you should get a
# updated setup-file automatically.

set(WXWIDGETSVERSION "3.2.8")
set(WXWIDGETS_MD5 "d809582fa4508eb350ef68a81f2fbd47")
set(WXWIDGETS_URL "https://github.com/wxWidgets/wxWidgets/releases/download/v${WXWIDGETSVERSION}/wxWidgets-${WXWIDGETSVERSION}.tar.bz2")


# Build wxWidgets static library (required for wxMaxima)
# We link statically, so the installation of the library is not
# necessary, we only need the locale files. But fortunately
# there is a make target, which does install the locale files only.
externalproject_add(wxwidgets
  URL "${WXWIDGETS_URL}"
  DOWNLOAD_DIR ${CMAKE_SOURCE_DIR}/downloads
  URL_MD5 ${WXWIDGETS_MD5}
  CONFIGURE_COMMAND ${CMAKE_BINARY_DIR}/wxwidgets/wxwidgets-prefix/src/wxwidgets/configure --disable-shared --disable-compat28 --with-zlib=builtin --with-libjpeg=builtin --without-libtiff --with-libpng=builtin --with-expat=builtin --build=${BUILDHOST} --host=${HOST} --enable-accessibility --enable-webview
  BUILD_COMMAND $(MAKE)
  INSTALL_COMMAND $(MAKE) locale_install "DESTDIR=${CMAKE_BINARY_DIR}/wxwidgets-installroot"
)

# Install wxWidgets locale files only.
install(DIRECTORY "${CMAKE_BINARY_DIR}/wxwidgets-installroot/usr/local/share/" DESTINATION share/)

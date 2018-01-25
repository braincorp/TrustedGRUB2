project(grub-tpm)
cmake_minimum_required (VERSION 3.5)

include(ExternalProject)

ExternalProject_Add(
  grub-tpm
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/
  CONFIGURE_COMMAND ./autogen.sh COMMAND ./configure --prefix=/usr/local --enable-device-mapper
  BUILD_COMMAND $(MAKE) COMMAND echo 'done'
  INSTALL_COMMAND $(MAKE) DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/grub-tpm install
  BUILD_IN_SOURCE 1
)

add_custom_target(
  grub-tpm.clean
  COMMAND rm -rf ${CMAKE_CURRENT_BINARY_DIR}
  COMMAND $(MAKE) -C ${CMAKE_CURRENT_SOURCE_DIR} distclean
)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/grub-tpm/ DESTINATION / USE_SOURCE_PERMISSIONS COMPONENT boot)

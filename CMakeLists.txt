project(grub-tpm)
cmake_minimum_required (VERSION 3.5)

# TPM debugging is really useful, so we have a flag for it
# When this is enabled, the system takes forever to boot, but you can
# see each PCR change and resulting hash as the system is checked
option(GRUBTPM_DEBUG off)

if(GRUBTPM_DEBUG) 
  set(GRUBTPM_OPTIONS "CPPFLAGS=-DTGRUB_DEBUG")
  message("building GRUB2 with debug...")
else()
  set(GRUBTPM_OPTIONS "")
endif()

# first generate ./configure for autotools
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/configure
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/autogen.sh 
)

# generate our output
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/.stamp
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/configure
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
  COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/configure --prefix=/usr/local --enable-device-mapper
  COMMAND $(MAKE) ${GRUBTPM_OPTIONS} 
  COMMAND mkdir -p ${CMAKE_CURRENT_BINARY_DIR}/pkg
  COMMAND $(MAKE) DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/pkg install
  COMMAND touch ${CMAKE_CURRENT_BINARY_DIR}/.stamp
)

# create a nice target
add_custom_target(
  grub-tpm ALL
  DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/.stamp
)

add_custom_target(
  grub-tpm.clean
  COMMAND rm -rf ${CMAKE_CURRENT_BINARY_DIR}
)

install(
  COMPONENT boot
  DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/pkg/
  DESTINATION /
  USE_SOURCE_PERMISSIONS)
add_package_content(
  boot
  DIRECTORIES ${CMAKE_CURRENT_BINARY_DIR}/pkg/
  DESTINATION /)
add_package_build_dependencies(boot grub-tpm)

cmake_minimum_required(VERSION 3.1)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/cmake/toolchain.cmake"
    CACHE FILEPATH
    "Toolchain to use for building this package and dependencies")

include(HunterGate)
HunterGate(
  URL "https://github.com/ruslo/hunter/archive/v0.19.181.tar.gz"
  SHA1 "d1746c22942f7310fc6a45fb6cdd5b8afcd6742c"
)

project(lmdbxx VERSION 0.9.14.0)

hunter_add_package(lmdb)
find_package(liblmdb CONFIG REQUIRED)

add_library(lmdbxx INTERFACE)
target_link_libraries(lmdbxx INTERFACE liblmdb::lmdb)

# Installation (https://github.com/forexample/package-example)

# Layout. This works for all platforms:
#   * <prefix>/lib/cmake/<PROJECT-NAME>
#   * <prefix>/lib/
#   * <prefix>/include/
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Configure '<PROJECT-NAME>ConfigVersion.cmake'
# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
)

# Configure '<PROJECT-NAME>Config.cmake'
# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
  "cmake/Config.cmake.in"
  "${project_config}"
  INSTALL_DESTINATION "${config_install_dir}"
)

# Targets:
#   * header location after install: <prefix>/include/lmdbxx/lmdb++.h
#   * headers can be included by C++ code `#include <lmdbxx/lmdb++.h>`
install(
    TARGETS lmdbxx
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

# Headers:
#   * lmdb++.h -> <prefix>/include/lmdbxx/lmdb++.h
install(
  FILES "lmdb++.h"
  DESTINATION "${include_install_dir}/lmdbxx"
)

# Config
#   * <prefix>/lib/cmake/Foo/FooConfig.cmake
#   * <prefix>/lib/cmake/Foo/FooConfigVersion.cmake
install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
)

# Config
#   * <prefix>/lib/cmake/Foo/FooTargets.cmake
install(
  EXPORT "${TARGETS_EXPORT_NAME}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
)

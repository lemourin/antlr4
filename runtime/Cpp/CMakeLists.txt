# -*- mode:cmake -*-
cmake_minimum_required (VERSION 2.8)
# 2.8 needed because of ExternalProject

# Detect build type, fallback to release and throw a warning if use didn't specify any
if(NOT CMAKE_BUILD_TYPE)
  message(WARNING "Build type not set, falling back to Release mode.
 To specify build type use:
 -DCMAKE_BUILD_TYPE=<mode> where <mode> is Debug or Release.")
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING
       "Choose the type of build, options are: Debug Release."
       FORCE)
endif(NOT CMAKE_BUILD_TYPE)

if(NOT WITH_DEMO)
  message(STATUS "Building without demo. To enable demo build use: -DWITH_DEMO=True")
  set(WITH_DEMO False CACHE STRING
    "Chose to build with or without demo executable"
    FORCE)
endif(NOT WITH_DEMO)

project(LIBANTLR4)

if(CMAKE_VERSION VERSION_EQUAL "3.0.0" OR
   CMAKE_VERSION VERSION_GREATER "3.0.0")
  CMAKE_POLICY(SET CMP0026 NEW)
  CMAKE_POLICY(SET CMP0054 OLD)
  CMAKE_POLICY(SET CMP0045 OLD)
  CMAKE_POLICY(SET CMP0042 OLD)
endif()

if(CMAKE_VERSION VERSION_EQUAL "3.3.0" OR
   CMAKE_VERSION VERSION_GREATER "3.3.0")
  CMAKE_POLICY(SET CMP0059 OLD)
  CMAKE_POLICY(SET CMP0054 OLD)
endif()

if(APPLE)
  find_library(COREFOUNDATION_LIBRARY CoreFoundation)
endif()

file(STRINGS "VERSION" ANTLR_VERSION)

if(WITH_DEMO)
  # Java is not necessary if building without demos.
  find_package(Java COMPONENTS Runtime REQUIRED)

  if(NOT ANTLR_JAR_LOCATION)
    message(FATAL_ERROR "Missing antlr4.jar location. You can specify it's path using: -DANTLR_JAR_LOCATION=<path>")
  else()
    get_filename_component(ANTLR_NAME ${ANTLR_JAR_LOCATION} NAME_WE)
    if(NOT EXISTS "${ANTLR_JAR_LOCATION}")
      message(FATAL_ERROR "Unable to find ${ANTLR_NAME} in ${ANTLR_JAR_LOCATION}")
    else()
      message(STATUS "Found ${ANTLR_NAME}: ${ANTLR_JAR_LOCATION}")
    endif()
  endif()
endif(WITH_DEMO)

add_subdirectory(runtime)
if(WITH_DEMO)
 add_subdirectory(demo)
endif(WITH_DEMO)

# Generate CMake Package Files only if install is active
if (ANTLR4_INSTALL)

  include(GNUInstallDirs)
  include(CMakePackageConfigHelpers)

  if(NOT ANTLR4_CMAKE_DIR)
    set(ANTLR4_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake CACHE STRING
      "Installation directory for cmake files." FORCE )
  endif(NOT ANTLR4_CMAKE_DIR)

  set(version_runtime_config ${PROJECT_BINARY_DIR}/antlr4-runtime-config-version.cmake)
  set(version_generator_config ${PROJECT_BINARY_DIR}/antlr4-generator-config-version.cmake)
  set(project_runtime_config ${PROJECT_BINARY_DIR}/antlr4-runtime-config.cmake)
  set(project_generator_config ${PROJECT_BINARY_DIR}/antlr4-generator-config.cmake)
  set(targets_export_name antlr4-targets)

  set(ANTLR4_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE STRING
      "Installation directory for libraries, relative to ${CMAKE_INSTALL_PREFIX}.")
  
  set(ANTLR4_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR}/antlr4-runtime CACHE STRING
      "Installation directory for include files, relative to ${CMAKE_INSTALL_PREFIX}.")

  configure_package_config_file(
    cmake/antlr4-runtime.cmake.in
    ${project_runtime_config}
    INSTALL_DESTINATION ${ANTLR4_CMAKE_DIR}/antlr4-runtime
    PATH_VARS 
    ANTLR4_INCLUDE_DIR
    ANTLR4_LIB_DIR )
  
  configure_package_config_file(
    cmake/antlr4-generator.cmake.in
    ${project_generator_config}
    INSTALL_DESTINATION ${ANTLR4_CMAKE_DIR}/antlr4-generator
    PATH_VARS 
    ANTLR4_INCLUDE_DIR
    ANTLR4_LIB_DIR )
  
  write_basic_package_version_file(
    ${version_runtime_config}
    VERSION ${ANTLR_VERSION}
    COMPATIBILITY SameMajorVersion )

  write_basic_package_version_file(
    ${version_generator_config}
    VERSION ${ANTLR_VERSION}
    COMPATIBILITY SameMajorVersion )

  install(EXPORT ${targets_export_name}
          DESTINATION ${ANTLR4_CMAKE_DIR}/antlr4-runtime )

  install(FILES ${project_runtime_config}
                ${version_runtime_config}
          DESTINATION ${ANTLR4_CMAKE_DIR}/antlr4-runtime )

  install(FILES ${project_generator_config}
                ${version_generator_config}
          DESTINATION ${ANTLR4_CMAKE_DIR}/antlr4-generator )

endif(ANTLR4_INSTALL)

if(EXISTS LICENSE.txt)
install(FILES LICENSE.txt
        DESTINATION "share/doc/libantlr4")
elseif(EXISTS ../../LICENSE.txt) 
install(FILES ../../LICENSE.txt
    DESTINATION "share/doc/libantlr4")
endif()

install(FILES README.md VERSION 
    DESTINATION "share/doc/libantlr4")

set(CPACK_PACKAGE_CONTACT "antlr-discussion@googlegroups.com")
set(CPACK_PACKAGE_VERSION ${ANTLR_VERSION})
include(CPack)

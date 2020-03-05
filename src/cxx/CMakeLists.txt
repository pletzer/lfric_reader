cmake_minimum_required(VERSION 3.1)
project(lfric_reader VERSION 1.0 LANGUAGES CXX)

# Source code requires at least C++ 11
set (CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED YES)

# Default to optimised build
if (NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE STREQUAL "")
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)
endif()

# Adds option -DBUILD_TESTING
include(CTest)

# These five lines are required in order to set up installation directories
# (which also control build directory locations) and enable shared builds
# (CMake's default is for a static build).
include(GNUInstallDirs)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(BUILD_SHARED_LIBS ON)

# Locate ParaView build and import CMake config, macros etc.
FIND_PACKAGE(ParaView REQUIRED)
# The `PARAVIEW_USE_FILE` is no longer used starting with 5.7.
#INCLUDE(${PARAVIEW_USE_FILE})

# Turn on DEBUG macro if CMAKE_BUILD_TYPE=Debug has been set
# This will enable debug output at runtime
add_compile_options("$<$<CONFIG:DEBUG>:-DDEBUG>")

# Scan the plugin file in order to set up internal data structures for building
# plugins.
paraview_plugin_scan(
  # The `paraview.plugin` file describing the plugin.
  PLUGIN_FILES "${CMAKE_CURRENT_SOURCE_DIR}/Plugin/paraview.plugin"
  # A result variable for the (enabled) plugins found during the scan.
  PROVIDES_PLUGINS plugins
  # Enable plugins during this scan by default.
  ENABLE_BY_DEFAULT ON)

# Build plugins discovered during the scan
paraview_plugin_build(PLUGINS ${plugins})

# Tell CMake to build a shared library plugin for ParaView
# Readers are only needed on the server side
# Unknown CMake command "ADD_PARAVIEW_PLUGIN"
# ADD_PARAVIEW_PLUGIN(
#      netCDFLFRicReader "1.0"
#      SERVER_MANAGER_XML netCDFLFRicReader.xml
#      SERVER_MANAGER_SOURCES vtkNetCDFLFRicReader.cxx
#      SOURCES netCDFLFRicFile.cxx netCDFLFRicReaderUtils.cxx
#      REQUIRED_ON_SERVER)

if(BUILD_TESTING)
  add_subdirectory(test)
endif()

# Copyright 2011,2012-2016 Free Software Foundation, Inc.
#
# This file is part of GNU Radio
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.


########################################################################
#
# Project setup
#
########################################################################

cmake_minimum_required(VERSION 3.0)
########################################################################
#
# set search path for cmake modules to OOT, Gnuradio
#
########################################################################
set( CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules")
include(FindGnuradioModules)

if(${GNURADIO_MODULE_DIRECTORY} STREQUAL "GNURADIO_MODULE_DIRECTORY-NOTFOUND")
    message(FATAL_ERROR "Cannot find gnuradio cmake Modules directory")
else()
    message(STATUS "Modules found in ${GNURADIO_MODULE_DIRECTORY}")
endif() 
list(APPEND CMAKE_MODULE_PATH "${GNURADIO_MODULE_DIRECTORY}" )
########################################################################
#
# First look in the install directory for package config of gnuradio
#
########################################################################
set( ENV{PKG_CONFIG_PATH} "${CMAKE_INSTALL_PREFIX}/lib/pkgconfig:$ENV{PKG_CONFIG_PATH}")

project(gr-display CXX C)
enable_testing()

#select the release build type by default to get optimization flags
if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Release")
   message(STATUS "Build type not specified: defaulting to release.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)

########################################################################
# Compiler specific setup
########################################################################
if(CMAKE_COMPILER_IS_GNUCXX AND NOT WIN32)
    #http://gcc.gnu.org/wiki/Visibility
    add_definitions(-fvisibility=hidden)
endif()

########################################################################
# Find boost
########################################################################
if(UNIX AND EXISTS "/usr/lib64")
    list(APPEND BOOST_LIBRARYDIR "/usr/lib64") #fedora 64-bit fix
endif(UNIX AND EXISTS "/usr/lib64")
set(Boost_ADDITIONAL_VERSIONS
    "1.35.0" "1.35" "1.36.0" "1.36" "1.37.0" "1.37" "1.38.0" "1.38" "1.39.0" "1.39"
    "1.40.0" "1.40" "1.41.0" "1.41" "1.42.0" "1.42" "1.43.0" "1.43" "1.44.0" "1.44"
    "1.45.0" "1.45" "1.46.0" "1.46" "1.47.0" "1.47" "1.48.0" "1.48" "1.49.0" "1.49"
    "1.50.0" "1.50" "1.51.0" "1.51" "1.52.0" "1.52" "1.53.0" "1.53" "1.54.0" "1.54"
    "1.55.0" "1.55" "1.56.0" "1.56" "1.57.0" "1.57" "1.58.0" "1.58" "1.59.0" "1.59"
    "1.60.0" "1.60" "1.61.0" "1.61" "1.62.0" "1.62" "1.63.0" "1.63" "1.64.0" "1.64"
    "1.65.0" "1.65" "1.66.0" "1.66" "1.67.0" "1.67" "1.68.0" "1.68" "1.69.0" "1.69"
)
find_package(Boost "1.35" COMPONENTS filesystem system)

if(NOT Boost_FOUND)
    message(FATAL_ERROR "Boost required to compile display")
endif()

########################################################################
# Find gnuradio build dependencies
########################################################################
set(GR_REQUIRED_COMPONENTS RUNTIME)

########################################################################
# Make sure to find the appropriate gnuradio installition if more than
# one installation of gnuradio exists
########################################################################
find_package(Gnuradio "3.8" REQUIRED PATHS ${CMAKE_INSTALL_PREFIX} ${GNURADIO_MODULE_DIRECTORY})

if(NOT GNURADIO_RUNTIME_FOUND)
    message(FATAL_ERROR "GnuRadio Runtime required to compile display")
endif()
find_package(Qt5Widgets REQUIRED )
set(QT_FOUND ${Qt5Widgets_FOUND})
set(QT_LIBRARIES ${Qt5Widgets_LIBRARIES})
set(QT_INCLUDE_DIRS ${Qt5Widgets_INCLUDE_DIRS})

find_package(PythonLibs 3)
find_package(SWIG)

if(NOT SWIG_FOUND)
    message(FATAL_ERROR "Swig required to build gr-display")
endif()

find_package(Doxygen)


include(GrPython)
GR_PYTHON_CHECK_MODULE("PyQt5" PyQt5 True PYQT5_FOUND)

########################################################################
#
# setup version info
#
########################################################################

set(VERSION_INFO_MAJOR_VERSION 3)
set(VERSION_INFO_API_COMPAT    8)
set(VERSION_INFO_MINOR_VERSION 0)
set(VERSION_INFO_MAINT_VERSION 0)
include(GrVersion) #setup version info

########################################################################
# Install directories
########################################################################
include(GrPlatform) #define LIB_SUFFIX
set(GR_RUNTIME_DIR      bin)
set(GR_LIBRARY_DIR      lib${LIB_SUFFIX})
set(GR_INCLUDE_DIR      include/display)
set(GR_DATA_DIR         share)
set(GR_PKG_DATA_DIR     ${CMAKE_INSTALL_PREFIX}/${GR_DATA_DIR}/gnuradio)
set(GR_DOC_DIR          ${GR_DATA_DIR}/doc)
set(GR_PKG_DOC_DIR      ${GR_DOC_DIR}/${CMAKE_PROJECT_NAME})
set(GR_CONF_DIR         etc)
set(GR_PKG_CONF_DIR     ${GR_CONF_DIR}/${CMAKE_PROJECT_NAME}/conf.d)
set(GR_LIBEXEC_DIR      libexec)
set(GR_PKG_LIBEXEC_DIR  ${GR_LIBEXEC_DIR}/${CMAKE_PROJECT_NAME})
set(GRC_BLOCKS_DIR      ${GR_PKG_DATA_DIR}/grc/blocks)

########################################################################
# On Apple only, set install name and use rpath correctly, if not already set
########################################################################
if(APPLE)
   if(NOT CMAKE_INSTALL_NAME_DIR)
       set(CMAKE_INSTALL_NAME_DIR
           ${CMAKE_INSTALL_PREFIX}/${GR_LIBRARY_DIR} CACHE
           PATH "Library Install Name Destination Directory" FORCE)
   endif(NOT CMAKE_INSTALL_NAME_DIR)
   if(NOT CMAKE_INSTALL_RPATH)
       set(CMAKE_INSTALL_RPATH
           ${CMAKE_INSTALL_PREFIX}/${GR_LIBRARY_DIR} CACHE
           PATH "Library Install RPath" FORCE)
   endif(NOT CMAKE_INSTALL_RPATH)
   if(NOT CMAKE_BUILD_WITH_INSTALL_RPATH)
       set(CMAKE_BUILD_WITH_INSTALL_RPATH ON CACHE
           BOOL "Do Build Using Library Install RPath" FORCE)
   endif(NOT CMAKE_BUILD_WITH_INSTALL_RPATH)
endif(APPLE)
########################################################################
# Setup the include and linker paths
########################################################################
include_directories(
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_SOURCE_DIR}/lib
    ${CMAKE_CURRENT_BINARY_DIR}/lib
    ${Boost_INCLUDE_DIRS}
    ${GNURADIO_RUNTIME_INCLUDE_DIRS}
    ${PYTHON_INCLUDE_DIRS}
)

link_directories(
    ${Boost_LIBRARY_DIRS}
    ${GNURADIO_RUNTIME_LIBRARY_DIRS}
)

# Set component parameters
set(GR_DISPLAY_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include CACHE INTERNAL "" FORCE)
set(GR_DISPLAY_SWIG_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/swig CACHE INTERNAL "" FORCE)

########################################################################
# Create uninstall target
########################################################################
configure_file(
    ${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
@ONLY)

add_custom_target(uninstall
    ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
)

########################################################################
# Add subdirectories
########################################################################
add_subdirectory(include/display)
add_subdirectory(lib)
add_subdirectory(swig)
add_subdirectory(python)
add_subdirectory(grc)
add_subdirectory(apps)
add_subdirectory(docs)
########################################################################
# Create Pkg Config File
########################################################################
# fake gnuradio cpack behavior as long as we don't use it directly
set(CPACK_PACKAGE_NAME "gnuradio-display")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "GNU Radio block to display text and png images")
set(CPACK_PACKAGE_VERSION ${VERSION})

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/gnuradio-display.pc.in
    ${CMAKE_CURRENT_BINARY_DIR}/gnuradio-display.pc
@ONLY)

install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/gnuradio-display.pc
    DESTINATION ${GR_LIBRARY_DIR}/pkgconfig
    COMPONENT "display"
)
########################################################################
# Print Summary
########################################################################
MESSAGE(STATUS "================ Build Summary =========================")
MESSAGE(STATUS "Building gr-display    : ${LIBVER} for ${CMAKE_SYSTEM_NAME} ")
MESSAGE(STATUS "Building for gnuradio  : ${Gnuradio_VERSION}")
MESSAGE(STATUS "Using CMAKE Module path: ${CMAKE_MODULE_PATH}" )
MESSAGE(STATUS "GNURADIO INCLUDES IN   : ${GNURADIO_RUNTIME_INCLUDE_DIRS}")
MESSAGE(STATUS "Using install prefix   : ${CMAKE_INSTALL_PREFIX}")
MESSAGE(STATUS "Installing grc files to : ${GRC_BLOCKS_DIR}")
MESSAGE(STATUS "========================================================")


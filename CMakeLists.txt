# SPDX-License-Identifier: Apache-2.0
# Copyright (C) 2021 oxmc

cmake_minimum_required(VERSION 2.8.12)
if (APPLE)
set(CMAKE_OSX_DEPLOYMENT_TARGET "10.13" CACHE STRING "" FORCE)
endif()

project(test LANGUAGES CXX C)
set(IMAGER_VERSION_MAJOR 1)
set(IMAGER_VERSION_MINOR 6)
set(IMAGER_VERSION_STR "1.0")
set(IMAGER_VERSION_CSV "1,0,0,0")
add_definitions(-DIMAGER_VERSION_STR="${IMAGER_VERSION_STR}")
add_definitions(-DIMAGER_VERSION_CSV=${IMAGER_VERSION_CSV})

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

include_directories(BEFORE .)

# Test if we need libatomic
include(CheckCXXSourceCompiles)
check_cxx_source_compiles("
    #include <atomic>
    #include <stdint.h>
    int main() {
        std::atomic<int64_t> x;
        x = 1;
        return (int) x;
    }"
    atomicbuiltin)

if (NOT atomicbuiltin)
        find_library(ATOMIC_LIBRARY NAMES atomic libatomic.so.1)
        if (NOT ATOMIC_LIBRARY)
                message( FATAL_ERROR "Missing libatomic while architecture does need it" )
        endif()
endif()

set(SOURCES "main.cpp" "Main.qml")

find_package(Qt5 COMPONENTS Core Quick LinguistTools Svg OPTIONAL_COMPONENTS Widgets)
if (Qt5Widgets_FOUND)
    set(EXTRALIBS ${EXTRALIBS} Qt5::Widgets)
endif()

set(SOURCES ${SOURCES} ${CMAKE_CURRENT_BINARY_DIR} ${QM_FILES})

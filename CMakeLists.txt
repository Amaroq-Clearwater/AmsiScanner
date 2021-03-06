# CMakeLists.txt --- CMake project settings
#    ex) cmake -G "Visual Studio 9 2008" .
#    ex) cmake -DCMAKE_BUILD_TYPE=Release -G "MSYS Makefiles" .
##############################################################################

# CMake minimum version
cmake_minimum_required(VERSION 2.4)

# project name and language
project(amsiscan C CXX)

# check build type
if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected, default to Debug")
    set(CMAKE_BUILD_TYPE "Debug")
endif()

# make uppercase string of build type
string(TOUPPER "${CMAKE_BUILD_TYPE}" uppercase_CMAKE_BUILD_TYPE)

# check build type again
if (NOT uppercase_CMAKE_BUILD_TYPE MATCHES "^(DEBUG|RELEASE|RELWITHDEBINFO|MINSIZEREL)$")
    message(FATAL_ERROR "Invalid value for CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
endif()

## set output directory (build/)
#set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/build)
#set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
#set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})

if (WIN32)
    # enable Win32 resource
    enable_language(RC)

    if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        # using Clang
        set(CMAKE_C_FLAGS "-static")
        set(CMAKE_CXX_FLAGS "-static")
    elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        # using GCC
        set(CMAKE_C_FLAGS "-static -static-libgcc -static-libstdc++ -s")
        set(CMAKE_CXX_FLAGS "-static -static-libgcc -static-libstdc++ -s")
    elseif (MSVC)
        # replace "/MD" with "/MT" (building without runtime DLLs)
        set(CompilerFlags
            CMAKE_C_FLAGS
            CMAKE_C_FLAGS_DEBUG
            CMAKE_C_FLAGS_RELEASE
            CMAKE_C_FLAGS_RELWITHDEBINFO
            CMAKE_CXX_FLAGS
            CMAKE_CXX_FLAGS_DEBUG
            CMAKE_CXX_FLAGS_RELEASE
            CMAKE_CXX_FLAGS_RELWITHDEBINFO)
        foreach(CompilerFlags ${CompilerFlags})
            string(REPLACE "/MD" "/MT" ${CompilerFlags} "${${CompilerFlags}}")
        endforeach()
    endif()
endif()

##############################################################################

# HAVE_AMSI_H
include(CheckIncludeFile)
check_include_file(amsi.h HAVE_AMSI_H)
if (HAVE_AMSI_H)
    add_definitions(-DHAVE_AMSI_H)
endif()

# PAMSIXX_STATIC
option(PAMSIXX_STATIC "PAMSI++ static" OFF)
if (HAVE_AMSI_H AND PAMSIXX_STATIC)
    add_definitions(-DPAMSIXX_STATIC)
else()
    set(PAMSIXX_STATIC OFF)
endif()
message(STATUS "PAMSIXX_STATIC: ${PAMSIXX_STATIC}")

# libads.a
add_library(ads STATIC ads.cpp)

# amsiscan.exe
add_executable(amsiscan amsiscan.cpp AmsiScanner.cpp)
if (HAVE_AMSI_H AND PAMSIXX_STATIC)
    target_link_libraries(amsiscan PRIVATE amsi)
endif()
target_link_libraries(amsiscan PRIVATE ads)

# ads_test.exe
add_executable(ads_test ads_test.cpp)
target_link_libraries(ads_test PRIVATE ads)

##############################################################################

cmake_minimum_required(VERSION 2.8.12.2)
project(netcoredbg)

set(CLR_DIR "${CMAKE_SOURCE_DIR}/../coreclr" CACHE FILEPATH "Path to coreclr directory")
set(CLR_BIN_DIR "" CACHE FILEPATH "Path to coreclr bin directory")
set(BUILD_MANAGED ON CACHE BOOL "Build managed part")
set(DBGSHIM_DIR "" CACHE FILEPATH "Path to dbgshim library directory")

if (WIN32)
    set(CMAKE_CXX_STANDARD 11)
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wno-null-conversion")
endif()

function(clr_unknown_arch)
    message(FATAL_ERROR "Only AMD64, ARM64, ARM, ARMEL, I386 and WASM are supported")
endfunction()

if (WIN32)
    # For windows it is expected that CLR_CMAKE_HOST_ARCH parameter is passed
    # to CMAKE to determine build arch.
    # If it is not passed, detect arch here.
    if ("${CLR_CMAKE_HOST_ARCH}" STREQUAL "")
        if ("${CMAKE_SIZEOF_VOID_P}" STREQUAL "4")
            set(CLR_CMAKE_HOST_ARCH x86)
        else()
            set(CLR_CMAKE_HOST_ARCH x64)
        endif()
    endif()
endif()

# Follow order as in root CoreCLR CMakeLists.txt
include(detectplatform.cmake) # from root
include(platformdefinitions.cmake) # from root
include(compileoptions.cmake) # disabled setting arm compiler flags in Tizen build
include(clrdefinitions.cmake)

# Basic checks for coreclr paths
if (NOT IS_DIRECTORY "${CLR_DIR}/src/pal")
    message(FATAL_ERROR "Invalid CoreCLR source directory")
endif()
if (NOT IS_DIRECTORY "${CLR_BIN_DIR}/shared/Microsoft.NETCore.App")
    message(FATAL_ERROR "Invalid .NET SDK directory")
endif()

add_subdirectory(src/debug/netcoredbg)

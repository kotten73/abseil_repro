cmake_minimum_required(VERSION 3.25)

# Check if the VCPKG_ROOT is defined.
if(NOT DEFINED ENV{VCPKG_ROOT})
    message(FATAL_ERROR "Environment variable VCPKG_ROOT is not set. Cloning/updating VCPKG locally.")
endif()

file(TO_CMAKE_PATH "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" VCPKG_CMAKE_PATH)
if(NOT EXISTS "${VCPKG_CMAKE_PATH}")
    message(FATAL_ERROR "Could not find file ${VCPKG_CMAKE_PATH}. Make sure you have set environment variable VCPKG_ROOT correctly.")
endif()

set(CMAKE_TOOLCHAIN_FILE "${VCPKG_CMAKE_PATH}")

set(VCPKG_TARGET_TRIPLET "x64-windows-static")

project(abseil_repro LANGUAGES CXX)

find_package(absl CONFIG REQUIRED)

add_executable(abseil_repro
    "src/main.cpp")

target_link_libraries(abseil_repro absl::any)
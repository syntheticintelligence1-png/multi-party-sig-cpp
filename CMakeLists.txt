cmake_minimum_required(VERSION 3.10)

if (DEFINED PLATFORM AND NOT PLATFORM STREQUAL "SGX")
    message(FATAL_ERROR "Unsupported PLATFORM: ${PLATFORM}. Only 'SGX' is supported, or leave it unset for the default platform.")
endif()

if (PLATFORM STREQUAL "SGX")
    project("MultiPartySigSgx" VERSION 1.1.0)
else()
    project("MultiPartySig" VERSION 1.1.0)
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE "Release")
#set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -Wall -Wextra -Wno-unused-parameter -Wno-unused-variable -fmacro-prefix-map=${CMAKE_SOURCE_DIR}=/safeheron")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -Wall -Wextra -Wno-unused-parameter -Wno-unused-variable -fmacro-prefix-map=${CMAKE_SOURCE_DIR}=/safeheron")

option(ENABLE_SNAP_SCOPE "Enable Snap Scope" OFF)
if (${ENABLE_SNAP_SCOPE})
    message(STATUS "Notice: ******************* Enable Snap Scope ******************* ")
endif()

option(TEST_STARK_CURVE "Use Stark Curve" OFF)
if (TEST_STARK_CURVE)
    add_definitions(-DTEST_STARK_CURVE)
endif()

include_directories(src)

cmake_policy(SET CMP0076 NEW)
cmake_policy(SET CMP0079 NEW)

if (PLATFORM STREQUAL "SGX")
    list(APPEND CMAKE_PREFIX_PATH "/opt/safeheron/ssgx")
    find_package(ssgx REQUIRED)
    find_package(SafeheronCryptoSuitesSgx REQUIRED)
    ssgx_add_trusted_library(${CMAKE_PROJECT_NAME}
        TRUSTED_LIBS SafeheronCryptoSuitesSgx
    )
else()
    add_library(${CMAKE_PROJECT_NAME} SHARED "")
endif()

add_subdirectory(src)

if (NOT PLATFORM STREQUAL "SGX")
    option(ENABLE_TESTS "Enable tests" OFF)
    if (${ENABLE_TESTS})
        enable_testing()
        add_subdirectory(test)
    endif()
endif()

include("cmake/setup_installation.cmake")

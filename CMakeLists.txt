# SPDX-License-Identifier: GPL-3.0-or-later
# Boldan Nanotech at any hour (README.md), a mod for OpenRAC's native port: the mod's library (C,
# on the SDK's openrac/openrac_mod.h) for Going Commando, and its test, the library's code against a
# host table and Boldan's mobys laid out by the test, on each disc.
#
#   python3 tools/fetch_sdk.py                     # the SDK sdk.lock pins, into .sdk/
#   cmake -S . -B build -G Ninja
#   cmake --build build
#   ctest --test-dir build
#   python3 .sdk/openrac-mod-sdk-*/tools/package_mod.py build/package/* -o dist
#
# Push a tag v<version> and the workflow (.github/workflows/build.yml) publishes the release the
# launcher installs from.
cmake_minimum_required(VERSION 3.20)
project(boldan_nanotech LANGUAGES C CXX)

# The SDK: the release sdk.lock pins (tools/fetch_sdk.py puts it in .sdk/); OPENRAC_SDK=<dir> (an
# unpacked SDK, or sdk/ in a checkout of OpenRAC) or OpenRACMod_DIR=<dir>/cmake for another.
if(NOT OpenRACMod_DIR)
  if(DEFINED OPENRAC_SDK)
    set(OpenRACMod_DIR "${OPENRAC_SDK}/cmake")
  elseif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/sdk.lock")
    file(READ "${CMAKE_CURRENT_SOURCE_DIR}/sdk.lock" lock)
    string(JSON sdk_version GET "${lock}" version)
    set(OpenRACMod_DIR "${CMAKE_CURRENT_SOURCE_DIR}/.sdk/openrac-mod-sdk-${sdk_version}/cmake")
    if(NOT EXISTS "${OpenRACMod_DIR}/OpenRACModConfig.cmake")
      message(FATAL_ERROR "No OpenRAC mod SDK ${sdk_version}: run python3 tools/fetch_sdk.py first "
                          "(or pass -DOPENRAC_SDK=<an unpacked SDK>)")
    endif()
  endif()
endif()
find_package(OpenRACMod 1.20 REQUIRED)

include(CTest)

openrac_add_mod(boldan_nanotech MANIFEST openrac-mod.json SOURCES src/mod.c)
set_target_properties(boldan_nanotech PROPERTIES C_STANDARD 11 C_STANDARD_REQUIRED ON)
if(NOT MSVC)
  target_compile_options(boldan_nanotech PRIVATE -Wall -Wextra)
endif()

if(BUILD_TESTING)
  add_executable(boldan_nanotech_test tests/boldan_test.c)
  set_target_properties(boldan_nanotech_test PROPERTIES C_STANDARD 11 C_STANDARD_REQUIRED ON)
  target_link_libraries(boldan_nanotech_test PRIVATE OpenRAC::mod)
  add_test(NAME boldan_nanotech COMMAND boldan_nanotech_test)
endif()

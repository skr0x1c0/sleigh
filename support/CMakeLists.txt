#
# Copyright (c) 2021-present, Trail of Bits, Inc.
# All rights reserved.
#
# This source code is licensed in accordance with the terms specified in
# the LICENSE file found in the root directory of this source tree.
#

# Generate the build and install directories to use in the support helpers
configure_file(
  "${CMAKE_CURRENT_LIST_DIR}/SpecFilePaths.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/SpecFilePaths.h"
  @ONLY
)

# Based on: https://github.com/andrew-hardin/cmake-git-version-tracking/blob/98441f9dc056b8d3d4b4287b09e6a99775d70305/better-example/CMakeLists.txt
# By Andrew Hardin
# Released under the MIT License.
# https://raw.githubusercontent.com/andrew-hardin/cmake-git-version-tracking/98441f9dc056b8d3d4b4287b09e6a99775d70305/LICENSE
#
# Define the two required variables before including
# the source code for watching a git repository.
set(PRE_CONFIGURE_FILE "${CMAKE_CURRENT_LIST_DIR}/Version.cpp.in")
set(POST_CONFIGURE_FILE "${CMAKE_CURRENT_BINARY_DIR}/Version.cpp")
set(GIT_STATE_FILE "${CMAKE_CURRENT_BINARY_DIR}/git-state-sleigh")
set(GIT_WORKING_DIR "${PROJECT_SOURCE_DIR}")
set(GIT_FAIL_IF_NONZERO_EXIT OFF)
include("${PROJECT_SOURCE_DIR}/cmake/git_watcher.cmake")

# Generate GHIDRA version helpers
configure_file(
  "${CMAKE_CURRENT_LIST_DIR}/GhidraVersion.cpp.in"
  "${CMAKE_CURRENT_BINARY_DIR}/GhidraVersion.cpp"
  @ONLY
)

add_library(support
  Support.cpp
  "${POST_CONFIGURE_FILE}"
  "${CMAKE_CURRENT_BINARY_DIR}/GhidraVersion.cpp"
)
# Need to explicitly add this dependency because BYPRODUCTS doesn't work under
# CMake 3.16 with GNU Makefiles
add_dependencies(support check_git)

add_library(sleigh::support ALIAS support)

target_compile_features(support PUBLIC
  cxx_std_17
)

set_target_properties(support PROPERTIES
  INTERFACE_POSITION_INDEPENDENT_CODE ON
)

target_include_directories(support PRIVATE
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
  "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/support>"
)

# Copyright (c) 2020 pongasoft
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not
# use this file except in compliance with the License. You may obtain a copy of
# the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations under
# the License.
#
# @author Yan Pujante

cmake_minimum_required(VERSION 3.13)

# Setting up project (note that the version is defined in lua.info)
project(BlankPlugin LANGUAGES CXX)

# Using C++17
set(CMAKE_CXX_STANDARD 17)

# Using RE SDK version 4.2.0
set(RE_SDK_VERSION 4.2.0)

# Location of RE SDK: can be set when invoking cmake => cmake -D "RE_SDK_ROOT:PATH=/path/to/re_sdk"
# or via -p option in configure.py script or in cmake-gui
if(APPLE)
  set(RE_SDK_ROOT "/Users/Shared/ReasonStudios/JukeboxSDK_${RE_SDK_VERSION}/SDK" CACHE PATH "Location of RE SDK")
else()
  set(RE_SDK_ROOT "C:/Users/Public/Documents/ReasonStudios/JukeboxSDK_${RE_SDK_VERSION}/SDK" CACHE PATH "Location of RE SDK")
endif()

# Location of RE2DRender (can be set similarly to RE_SDK_ROOT)
set(RE_2D_RENDER_ROOT "${RE_SDK_ROOT}/../RE2DRender" CACHE PATH "Location of RE2DRender")

set(RE_CMAKE_ROOT_DIR "")
#set(RE_CMAKE_ROOT_DIR "../../pongasoft/re-cmake")

# If RE_CMAKE_ROOT_DIR is undefined/empty, it will fetch/download it from github
# If RE_CMAKE_ROOT_DIR is defined, it should point to the (local) folder where re-cmake is located
include(cmake/re-cmake.cmake)

# Including logging cmake file (loguru) for Recon build
include(cmake/logging.cmake)

# Defining where the sources of the plugin live
set(RE_CPP_SRC_DIR ${CMAKE_CURRENT_LIST_DIR}/src/cpp)

# Defining all the source files for the project
# Although you could use a file(GLOB ...) pattern, it is NOT recommended as the CMake generation would happen on
# every single build!
set(re_sources_cpp
    ${RE_CPP_SRC_DIR}/Device.h
    ${RE_CPP_SRC_DIR}/Device.cpp
    ${RE_CPP_SRC_DIR}/JukeboxExports.cpp
    )

# Defining where the local GUI2D files live
set(RE_2D_SRC_DIR ${CMAKE_CURRENT_LIST_DIR}/GUI2D)
# Defining where the built-in images live (no need to duplicate them in this project)
set(RE_SDK_BUILTIN_IMG_DIR "${RE_2D_RENDER_ROOT}/Images")

# Defining all the 2D source files
# Although you could use a file(GLOB ...) pattern, it is NOT recommended as the CMake generation would happen on
# every single build!
set(re_sources_2d

    # lua files describing the GUI
    ${RE_2D_SRC_DIR}/device_2D.lua
    ${RE_2D_SRC_DIR}/hdgui_2D.lua

    # Images for the device
    ${RE_2D_SRC_DIR}/Panel_Back.png
    ${RE_2D_SRC_DIR}/Panel_Folded_Back.png
    ${RE_2D_SRC_DIR}/Panel_Folded_Front.png
    ${RE_2D_SRC_DIR}/Panel_Front.png

    # Images provided by the SDK
    ${RE_SDK_BUILTIN_IMG_DIR}/Placeholder.png
    ${RE_SDK_BUILTIN_IMG_DIR}/SharedAudioJack.png
    ${RE_SDK_BUILTIN_IMG_DIR}/TapeHorz.png

    )

# Adding the RE plugin
add_re_plugin(
    RE_SDK_VERSION ${RE_SDK_VERSION}
    RE_SDK_ROOT ${RE_SDK_ROOT}
    RE_2D_RENDER_ROOT ${RE_2D_RENDER_ROOT}
    BUILD_SOURCES ${re_sources_cpp} # compiled for both local and jbox builds
    NATIVE_BUILD_SOURCES ${logging_sources} # compiled only for local builds
    RENDER_2D_SOURCES ${re_sources_2d}
    INCLUDE_DIRECTORIES ${LOGGING_CPP_SRC_DIR}
    COMPILE_OPTIONS -Wall
    ENABLE_DEBUG_LOGGING # turn on JBOX_TRACE and loguru
)

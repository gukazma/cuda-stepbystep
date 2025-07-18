cmake_minimum_required(VERSION 3.16)
if(CMAKE_HOST_SYSTEM_NAME MATCHES "Linux")
	message("Current platform: Linux ")
  set(VCPKG_TARGET_TRIPLET x64-linux-dynamic CACHE STRING "")
elseif(CMAKE_HOST_SYSTEM_NAME MATCHES "Windows")
	message("Current platform: Windows ")
  set(VCPKG_TARGET_TRIPLET x64-windows CACHE STRING "")
else()
	message("Current platform: unkonw ")
endif()
# 设置基础环境
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# 设置默认的编译配置为 Release，conan.cmake 需要确定你当前正在使用哪种编译配置以决定如何下载和编译三方库
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build." FORCE)
    message(STATUS "No CMAKE_BUILD_TYPE selected, defaulting to ${CMAKE_BUILD_TYPE}")
endif ()

get_filename_component(ROOT_NAME ${CMAKE_SOURCE_DIR} NAME)
message("Configure project: ${ROOT_NAME}")
project(${ROOT_NAME} VERSION 0.1 LANGUAGES CXX CUDA)

option(ENABLE_DOCS "Enable doxygen docs" ON)
option(ENABLE_QTAPP "Enable config Qt app" OFF)
option(PUBLISH_APP "PUBLISH APP" OFF)
option(ENABLE_CUDA_DEBUG "ENABLE CUDA DEBUG" ON)

# just for qt
# set(CMAKE_AUTOUIC ON)
# set(CMAKE_AUTOMOC ON)
# set(CMAKE_AUTORCC ON)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
find_package(CUDAToolkit REQUIRED)
find_package(CUDA REQUIRED COMPONENTS cudart)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# set(CMAKE_CUDA_ARCHITECTURES 50 52 60 61 70 72 75 80 86 87 89 90 100 101 120)
set(CMAKE_CUDA_ARCHITECTURES 70 72 75 80 86 87)
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -Wno-deprecated-gpu-targets")
if(ENABLE_CUDA_DEBUG)
    set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -G -g")        # enable cuda-gdb (may significantly affect performance on some targets)
else()
    set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -lineinfo") # add line information to all builds for debug tools (exclusive to -G option)
endif()
set(CMAKE_POLICY_DEFAULT_CMP0091 NEW)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_DEBUG_POSTFIX "d")
set(CMAKE_RELEASE_POSTFIX "") 
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_INSTALL_PREFIX}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_INSTALL_PREFIX}/bin)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${CMAKE_INSTALL_PREFIX}/bin)  



include(CMakeModules/doxygen.cmake)
include(CMakeModules/dependency-graph.cmake)
include(CMakeModules/formatting.cmake)
# ---- Include guards ----
if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(
    FATAL_ERROR
      "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

add_subdirectory(Modules)
add_subdirectory(Apps)
enable_testing()
add_subdirectory(Tests)

if(ENABLE_DOCS)
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/../Docs)
  build_docs(PROCESS_DOXYFILE TRUE DOXYFILE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/Docs/Doxyfile.in)
endif()
set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT DemoApp)

if(CLANG_FORMAT_EXE)
  message(STATUS "Found clang format ............ Configuring ..............")
  file(GLOB_RECURSE FFILES ${CMAKE_SOURCE_DIR}/Modules/*.[hc] ${CMAKE_SOURCE_DIR}/Modules/*.[hc]pp ${CMAKE_SOURCE_DIR}/Apps/*.[hc] ${CMAKE_SOURCE_DIR}/Apps/*.[hc]pp)
  clang_format(${PROJECT_NAME}Format ${FFILES})
  set_target_properties(${PROJECT_NAME}Format PROPERTIES FOLDER Tools)
endif()

if(DOT_EXE)
  message(STATUS "Found Graphviz ............ Configuring ..............")
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/../Docs/DependencyGraph)
  gen_dep_graph(
    png 
    OUTPUT_DIR ${CMAKE_CURRENT_BINARY_DIR}/../Docs/DependencyGraph
    TARGET_NAME ${PROJECT_NAME}DependencyGraph
  )
  set_target_properties(${PROJECT_NAME}DependencyGraph PROPERTIES FOLDER Tools)
endif()
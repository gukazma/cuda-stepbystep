set(TARGET_NAME "TEST")

find_package(GTest CONFIG REQUIRED)

file(GLOB_RECURSE H_FILES "*.h")
file(GLOB_RECURSE CPP_FILES "*.cpp")
file(GLOB_RECURSE CU_FILES "*.cu")

source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}" FILES ${H_FILES} ${CPP_FILES} ${CU_FILES})

add_executable(${TARGET_NAME} ${H_FILES} ${CPP_FILES} ${CU_FILES})

set_target_properties(${TARGET_NAME} PROPERTIES CXX_STANDARD 17 OUTPUT_NAME ${TARGET_NAME})
set_target_properties(${TARGET_NAME} PROPERTIES FOLDER Test)
set_target_properties(${TARGET_NAME} PROPERTIES VS_DEBUGGER_COMMAND_ARGUMENTS "${DEBUG_ARGUES}")
set_target_properties(${TARGET_NAME}  PROPERTIES CUDA_SEPARABLE_COMPILATION ON)

target_link_libraries(${TARGET_NAME} 
      PRIVATE
            GTest::gtest GTest::gtest_main
)

target_include_directories(
  ${TARGET_NAME} 
  PUBLIC 
  ${CMAKE_CURRENT_LIST_DIR}/Common
)

target_compile_options(${TARGET_NAME}  PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:--extended-lambda>)
target_compile_features(${TARGET_NAME}  PRIVATE cxx_std_17 cuda_std_17)

add_test(
    NAME ${TARGET_NAME}
    COMMAND ${TARGET_NAME}
)
cmake_minimum_required( VERSION 2.8 )

set(PROJECT_NAME ft_prdt)
project(${PROJECT_NAME})


set(GTEST_ROOT ${CMAKE_SOURCE_DIR}/../thirdparty/gtest)
set(PRDT_ROOT ${CMAKE_SOURCE_DIR}/../source)
set(PRDT_PUB_HEADER ${CMAKE_SOURCE_DIR}/../include)

message(STATUS "——————————————————————————————————————")
message(STATUS "PROJECT_NAME : " ${PROJECT_NAME})
message(STATUS "DIR : " ${CMAKE_BINARY_DIR})
message(STATUS "——————————————————————————————————————")

find_package(OpenCV REQUIRED)

add_subdirectory(${PRDT_ROOT} ${CMAKE_BINARY_DIR}/PRDT)
add_subdirectory(${GTEST_ROOT} ${CMAKE_BINARY_DIR}/gtest)

include_directories(${PRDT_PUB_HEADER})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/source/include)
include_directories(${OpenCV_INCLUDE_DIRS})
include_directories(${GTEST_ROOT}/include)

macro(group_src_by_dir src_files)
    foreach(file_path ${${src_files}})
        string(REGEX REPLACE ${CMAKE_CURRENT_SOURCE_DIR}/\(.*\) \\1 relative_path ${file_path})
        string(REGEX REPLACE "(.+)\\..*" \\1 relative_group ${relative_path})
        list(FIND c_groups ${relative_group} index)
        if(NOT index LESS 0)
            string(REPLACE "/" "\\" relative_group ${relative_group})
            source_group(${relative_group} FILES ${file_path})
        else()
            string(REGEX MATCH "(.+)/.*" has_group ${relative_path})
            if(has_group)
                string(REGEX REPLACE "(.+)/.*" \\1 group ${relative_path})
                string(REPLACE "/" "\\" group ${group})
                source_group(${group} FILES ${file_path})
            else()
                source_group("" FILES ${file_path})
            endif()
        endif()
    endforeach(file_path)
endmacro(group_src_by_dir)

file(GLOB_RECURSE HEADER_FILES
    ${PRDT_PUB_HEADER}/*.h
    *.h
    *.hpp
)

file(GLOB_RECURSE SRC_FILES 
    *.c
    *.cpp
    *.cc
)

group_src_by_dir(HEADER_FILES)
group_src_by_dir(SRC_FILES)

if(MSVC)
    add_compile_options("/MT")
elseif (UNIX)
endif()

add_executable(${PROJECT_NAME} ${HEADER_FILES} ${SRC_FILES})

target_link_libraries(${PROJECT_NAME} prdt gtest)

enable_testing()
add_test(NAME ${PROJECT_NAME}
                CONFIGURATIONS  Release
                COMMAND ${PROJECT_NAME})

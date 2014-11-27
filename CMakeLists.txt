project(PLY CXX)

cmake_minimum_required(VERSION 2.8.0)

# Location where cmake first looks for modules.
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/modules)

################################################################
# Configure and find libraries
################################################################
include(find_compiler)
include(FindBoost)

set(PLY_PACKAGE_AUTHOR "Ares Lagae")
set(PLY_PACKAGE_BUGREPORT "https://github.com/4og/libply/issues")
set(PLY_PACKAGE_NAME "ply")
set(PLY_PACKAGE_VERSION "0.1")

configure_file (
  "${PROJECT_SOURCE_DIR}/config.hpp.in"
  "${PROJECT_BINARY_DIR}/ply/config.hpp"
  )

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif()

set(TOOLS ply2obj ply2raw ply2ply plyheader)

if(UNIX)
	set(CMAKE_CXX_FLAGS_RELEASE "-s -O4 --std=c++11")
	set(CMAKE_CXX_FLAGS_DEBUG "-g -Wall --std=c++11")
elseif(MSVC)
	set(CMAKE_CXX_FLAGS_RELEASE "-D NDEBUG -D _USE_MATH_DEFINES -D _SECURE_SCL=0 -D _SCL_SECURE_NO_WARNINGS -D _CRT_SECURE_NO_DEPRECATE /MD /MP")
	set(CMAKE_CXX_FLAGS_DEBUG "-D_DEBUG -D _USE_MATH_DEFINES /MDd /Zi /MP")
endif(UNIX)

file(GLOB PLY_H RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ply/*.hpp)
file(GLOB PLY_SRC RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ply/*.cpp)

link_directories(${BOOST_LIBRARY_DIRS})

include_directories("${PROJECT_BINARY_DIR}")
include_directories("${PROJECT_SOURCE_DIR}")
include_directories(SYSTEM ${BOOST_INCLUDE_DIRS})

add_library(ply_static STATIC ${PLY_H} ${PLY_SRC})
set_target_properties(ply_static PROPERTIES OUTPUT_NAME ply)

set(TARGETS ${TOOLS} ply_static)

if(UNIX)
  add_library(ply_shared SHARED ${PLY_H} ${PLY_SRC})
  set_target_properties(ply_shared PROPERTIES OUTPUT_NAME ply)
  list(APPEND TARGETS ply_shared)
  target_link_libraries(ply_static general ${BOOST_LIBRARIES})
endif()

foreach(TOOL_NAME ${TOOLS})
  add_executable(${TOOL_NAME} tools/${TOOL_NAME}.cpp)
  target_link_libraries(${TOOL_NAME} general ply_static)
  add_dependencies(${TOOL_NAME} ply_static)
endforeach()

install(FILES ply.hpp DESTINATION include)
install(FILES ${PLY_H} "${PROJECT_BINARY_DIR}/ply/config.hpp"        
        DESTINATION include/ply)

install(TARGETS ${TARGETS}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)


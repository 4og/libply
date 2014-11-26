PROJECT(PLY CXX)
ENABLE_LANGUAGE(C)

cmake_minimum_required(VERSION 2.8.0)

# Location where cmake first looks for modules.
list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/modules)

################################################################
# Configure and find libraries
################################################################
include(find_compiler)
include(FindBoost)

set(LIB_PATHS
    ${BOOST_LIBRARY_DIRS}
)

set(INCLUDE_PATHS
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/ply
    ${CMAKE_CURRENT_SOURCE_DIR}/build
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif()


set(CMAKE_CXX_FLAGS "-D PACKAGE_AUTHOR -D PACKAGE_VERSION -D PACKAGE_NAME -D PACKAGE_BUGREPORT")

IF (UNIX)
	set(CMAKE_CXX_FLAGS_RELEASE "-s -O4 --std=c++11")
	set(CMAKE_CXX_FLAGS_DEBUG "-g -Wall --std=c++11")
ELSEIF(MSVC)
	set(CMAKE_CXX_FLAGS_RELEASE "-D NDEBUG -D _USE_MATH_DEFINES -D _SECURE_SCL=0 -D _SCL_SECURE_NO_WARNINGS -D _CRT_SECURE_NO_DEPRECATE /MD /MP")
	set(CMAKE_CXX_FLAGS_DEBUG "-D_DEBUG -D _USE_MATH_DEFINES /MDd /Zi /MP")
ENDIF(UNIX)

FILE(GLOB         PLY_SRC           RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ply/*.cpp ply/*.hpp)

LINK_DIRECTORIES(${BOOST_LIBRARY_DIRS})

INCLUDE_DIRECTORIES(${INCLUDE_PATHS} ${BOOST_INCLUDE_DIRS})

ADD_LIBRARY( libply STATIC ${PLY_SRC} )

IF (MSVC)
ELSEIF (UNIX)
  TARGET_LINK_LIBRARIES( libply general ${BOOST_LIBRARIES})
ENDIF (MSVC)

ADD_EXECUTABLE (ply2obj tools/ply2obj.cpp)
TARGET_LINK_LIBRARIES( ply2obj general libply)
ADD_DEPENDENCIES(ply2obj libply)

ADD_EXECUTABLE (ply2raw tools/ply2raw.cpp)
TARGET_LINK_LIBRARIES( ply2raw general libply)
ADD_DEPENDENCIES(ply2raw libply)

ADD_EXECUTABLE (ply2ply tools/ply2ply.cpp)
TARGET_LINK_LIBRARIES( ply2ply general libply)
ADD_DEPENDENCIES(ply2ply libply)

ADD_EXECUTABLE (plyheader tools/plyheader.cpp)
TARGET_LINK_LIBRARIES( plyheader general libply)
ADD_DEPENDENCIES(plyheader libply)

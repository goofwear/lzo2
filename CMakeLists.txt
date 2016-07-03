#
# CMakeLists.txt --- a simple "CMake" file for building LZO
#
# This file is part of the LZO data compression library.
#
# Copyright (C) 1996-2015 Markus Franz Xaver Johannes Oberhumer
# All Rights Reserved.
#

#
# simple usage:
#     mkdir -p build && cd build && cmake .. && make
#
# another usage example:
#     mkdir -p build/release-i686
#     cd       build/release-i686
#     cmake ../.. -DENABLE_STATIC=0 -DENABLE_SHARED=1 \
#         -DCMAKE_C_COMPILER=gcc -DCMAKE_C_FLAGS="-m32 -march=i686" \
#         -DCMAKE_INSTALL_PREFIX=/opt/local/prefix-i686
#     make VERBOSE=1
#     make install
#
# see http://www.cmake.org/ for more info
#

#
# init
#

cmake_minimum_required(VERSION 2.6 FATAL_ERROR)

# Disallow in-source builds. Note that you will still have to manually
# clean up a few files if you accidentally try an in-source build.
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES  ON)
if(",${CMAKE_SOURCE_DIR}," STREQUAL ",${CMAKE_BINARY_DIR},")
    message(FATAL_ERROR "ERROR: In-source builds are not allowed.")
endif()

project(lzo C)

#
# configuration options
#
option(BUILD_TEST "Build test" OFF)
option(BUILD_EXAMPLES "Build examples" OFF)
#
# targets
#

file(GLOB lzo_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c")
list(SORT lzo_SOURCES)

if(MSVC)
    set(CMAKE_DEBUG_POSTFIX "d")
endif()

# LZO library
add_library(lzo2 ${lzo_SOURCES})
if(BUILD_SHARED_LIBS)
    set_target_properties(lzo2 PROPERTIES SOVERSION 2 VERSION 2.0.0)
endif()


# main test driver

if(BUILD_TESTS)
    add_executable(lzotest  lzotest/lzotest.c)
    target_link_libraries(lzotest lzo2)
endif()
# examples
if(BUILD_EXAMPLES)
    add_executable(dict     examples/dict.c)
    target_link_libraries(dict lzo2)
    add_executable(lzopack  examples/lzopack.c)
    target_link_libraries(lzopack lzo2)
    add_executable(overlap  examples/overlap.c)
    target_link_libraries(overlap lzo2)
    add_executable(precomp  examples/precomp.c)
    target_link_libraries(precomp lzo2)
    add_executable(precomp2 examples/precomp2.c)
    target_link_libraries(precomp2 lzo2)
    add_executable(simple   examples/simple.c)
    target_link_libraries(simple lzo2)
endif()
# some boring internal test programs
if(0)
    lzo_add_executable(align    tests/align.c)
    lzo_add_executable(chksum   tests/chksum.c)
    lzo_add_executable(promote  tests/promote.c)
    lzo_add_executable(sizes    tests/sizes.c)
endif()

# miniLZO
if(BUILD_TESTS)
    add_executable(testmini minilzo/testmini.c minilzo/minilzo.c)
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/lzo) # needed for "lzoconf.h"
endif()

#
# compilation flags
#

target_include_directories(lzo2
    PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR};include>
    INTERFACE
    $<INSTALL_INTERFACE:include/lzo>
)

include(TestBigEndian)

# mfx_LZO_CHECK_ENDIAN
TEST_BIG_ENDIAN(big_endian)
if ("${big_endian}" MATCHES "^1$")
    add_definitions(-DLZO_ABI_BIG_ENDIAN=1)
elseif ("${big_endian}" MATCHES "^0$")
    add_definitions(-DLZO_ABI_LITTLE_ENDIAN=1)
else()
    message(FATAL_ERROR "ERROR: TEST_BIG_ENDIAN failed with result '${big_endian}'.")
endif()

# LZO_HAVE_CONFIG_H
add_definitions(-DLZO_CFG_NO_CONFIG_HEADER=1)

#
# "make install"
#

# these subdirs are relative to CMAKE_INSTALL_PREFIX
if(NOT DEFINED install_doc_subdir)
    set(install_doc_subdir "share/doc/lzo")
endif()
if(NOT DEFINED install_examples_subdir)
    set(install_examples_subdir "libexec/lzo/examples")
endif()

set(doc_DATA AUTHORS COPYING NEWS THANKS doc/LZO.FAQ doc/LZO.TXT doc/LZOAPI.TXT)
set(pkginclude_HEADERS
    include/lzo/lzo1.h include/lzo/lzo1a.h include/lzo/lzo1b.h
    include/lzo/lzo1c.h include/lzo/lzo1f.h include/lzo/lzo1x.h
    include/lzo/lzo1y.h include/lzo/lzo1z.h include/lzo/lzo2a.h
    include/lzo/lzo_asm.h include/lzo/lzoconf.h include/lzo/lzodefs.h
    include/lzo/lzoutil.h
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/lzo2-config-version.cmake
  VERSION 2.0.9
  COMPATIBILITY AnyNewerVersion
)

install(FILES ${doc_DATA} DESTINATION "${install_doc_subdir}")
install(FILES ${pkginclude_HEADERS} DESTINATION include/lzo)
install(TARGETS lzo2 EXPORT lzo2
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib)


install(EXPORT lzo2
  FILE
    lzo2.cmake
  NAMESPACE
    lzo2::
  DESTINATION
    lib/cmake/lzo2
)
install(
  FILES
    cmake/lzo2-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/lzo2-config-version.cmake
  DESTINATION
    lib/cmake/lzo2
)
# Copyright (C) 2007-2014 Glenn Randers-Pehrson
# Copyright (C) 2015 Ruslan Baratov

# This code is released under the libpng license.
# For conditions of distribution and use, see the disclaimer
# and license in png.h

cmake_minimum_required(VERSION 3.0)
set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS true)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.8.1.tar.gz"
    SHA1 "a9bd96ab113f3af870b13cd05cf4e75b8848cf4a"
)

project(PNG VERSION 1.6.16 LANGUAGES C)

enable_testing()

set(PNGLIB_MAJOR ${PROJECT_VERSION_MAJOR})
set(PNGLIB_MINOR ${PROJECT_VERSION_MINOR})
set(PNGLIB_RELEASE ${PROJECT_VERSION_PATCH})
set(PNGLIB_NAME libpng${PNGLIB_MAJOR}${PNGLIB_MINOR})
set(PNGLIB_VERSION ${PNGLIB_MAJOR}.${PNGLIB_MINOR}.${PNGLIB_RELEASE})

# needed packages
hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED)

# COMMAND LINE OPTIONS
option(PNG_TESTS  "Build libpng tests" YES)

# Many more configuration options could be added here
option(PNGARG            "Disable ANSI-C prototypes" NO)

# to distinguish between debug and release lib
if(NOT DEFINED CMAKE_DEBUG_POSTFIX)
  set(CMAKE_DEBUG_POSTFIX "d")
endif()

# Use the prebuilt pnglibconf.h file from the scripts folder
# TODO: fix this by building with awk; without this no cmake build can be
# configured directly (to do so indirectly use your local awk to build a
# pnglibconf.h in the build directory.)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/scripts/pnglibconf.h.prebuilt
               ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# OUR SOURCES
set(libpng_public_hdrs
  png.h
  pngconf.h
  ${CMAKE_CURRENT_BINARY_DIR}/pnglibconf.h
)
set(libpng_sources
  ${libpng_public_hdrs}
  pngdebug.h
  pnginfo.h
  pngpriv.h
  pngstruct.h
  png.c
  pngerror.c
  pngget.c
  pngmem.c
  pngpread.c
  pngread.c
  pngrio.c
  pngrtran.c
  pngrutil.c
  pngset.c
  pngtrans.c
  pngwio.c
  pngwrite.c
  pngwtran.c
  pngwutil.c
  arm/arm_init.c
  arm/filter_neon.S
  arm/filter_neon_intrinsics.c
)
set(pngtest_sources
  pngtest.c
)
set(pngvalid_sources
  contrib/libtests/pngvalid.c
)
set(pngstest_sources
  contrib/libtests/pngstest.c
)
# SOME NEEDED DEFINITIONS

if(MSVC)
  add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
endif(MSVC)

# NOW BUILD OUR TARGET
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_library(png ${libpng_sources})

# Some platforms (e.g. Linux) need separate math library
include(CheckLibraryExists)
check_library_exists(m pow "" LIB_M_REQUIRED)
set(M_LIBRARY "")
if(LIB_M_REQUIRED)
  set(M_LIBRARY m)
endif()

target_link_libraries(png PRIVATE ZLIB::zlib ${M_LIBRARY})

target_compile_definitions(png PUBLIC "$<$<CONFIG:Debug>:PNG_DEBUG>")

if(BUILD_SHARED_LIBS AND WIN32)
  target_compile_definitions(png PUBLIC "PNG_BUILD_DLL")
endif()

if(PNG_TESTS)
  # does not work with msvc due to png_lib_ver issue
  add_executable(pngtest ${pngtest_sources})
  target_link_libraries(pngtest png)
  add_test(pngtest ./pngtest ${CMAKE_CURRENT_SOURCE_DIR}/pngtest.png)
  #
  add_executable(pngvalid ${pngvalid_sources})
  target_link_libraries(pngvalid png)
  add_test(pngvalid ./pngvalid)
  add_executable(pngstest ${pngstest_sources})
  target_link_libraries(pngstest png)
  add_test(pngstest ./pngstest
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn0g01.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn0g02.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn0g04.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn0g08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn0g16.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn2c08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn2c16.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn3p01.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn3p02.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn3p04.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn3p08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn4a08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn4a16.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn6a08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/basn6a16.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbbn0g01.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbbn0g02.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbbn0g04.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbbn2c16.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbbn3p08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbgn2c16.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbgn3p08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbrn2c08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbwn0g16.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbwn3p08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftbyn3p08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftp0n0g08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftp0n2c08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftp0n3p08.png
    ${CMAKE_CURRENT_SOURCE_DIR}/contrib/pngsuite/ftp1n3p08.png
  )
endif()

####
# Installation (https://github.com/forexample/package-example)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: variable 'targets_export_name' used
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)

install(
    TARGETS png
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    FILES ${libpng_public_hdrs}
    DESTINATION "${include_install_dir}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

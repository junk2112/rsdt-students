cmake_minimum_required(VERSION 2.4.4)

set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)

project(zlib)

include(CheckTypeSize)
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckCSourceCompiles)
enable_testing()

check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(stdint.h    HAVE_STDINT_H)
check_include_file(stddef.h    HAVE_STDDEF_H)

#
# Check to see if we have large file support
#
set(CMAKE_REQUIRED_DEFINITIONS -D_LARGEFILE64_SOURCE=1)
# We add these other definitions here because CheckTypeSize.cmake
# in CMake 2.4.x does not automatically do so and we want
# compatibility with CMake 2.4.x.
if(HAVE_SYS_TYPES_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_SYS_TYPES_H)
endif()
if(HAVE_STDINT_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_STDINT_H)
endif()
if(HAVE_STDDEF_H)
    list(APPEND CMAKE_REQUIRED_DEFINITIONS -DHAVE_STDDEF_H)
endif()
check_type_size(off64_t OFF64_T)
if(HAVE_OFF64_T)
   add_definitions(-D_LARGEFILE64_SOURCE=1)
endif()
set(CMAKE_REQUIRED_DEFINITIONS) # clear variable

#
# Check for fseeko
#
check_function_exists(fseeko HAVE_FSEEKO)
if(NOT HAVE_FSEEKO)
    add_definitions(-DNO_FSEEKO)
endif()

#
# Check for unistd.h
#
check_include_file(unistd.h Z_HAVE_UNISTD_H)

if(MSVC)
    add_definitions(-D_CRT_SECURE_NO_DEPRECATE)
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
endif()

if(UNIX)
    if(CMAKE_COMPILER_IS_GNUCXX OR CV_ICC)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
    endif()
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/zconf.h.cmakein
               ${CMAKE_CURRENT_SOURCE_DIR}/src/zconf.h @ONLY)

#============================================================================
# zlib
#============================================================================

set(ZLIB_PUBLIC_HDRS
    src/zconf.h
    src/zlib.h
)
set(ZLIB_PRIVATE_HDRS
    src/crc32.h
    src/deflate.h
    src/gzguts.h
    src/inffast.h
    src/inffixed.h
    src/inflate.h
    src/inftrees.h
    src/trees.h
    src/zutil.h
)
set(ZLIB_SRCS
    src/adler32.c
    src/compress.c
    src/crc32.c
    src/deflate.c
    src/gzclose.c
    src/gzlib.c
    src/gzread.c
    src/gzwrite.c
    src/inflate.c
    src/infback.c
    src/inftrees.c
    src/inffast.c
    src/trees.c
    src/uncompr.c
    src/zutil.c
)

add_library(zlib STATIC ${ZLIB_SRCS} ${ZLIB_PUBLIC_HDRS} ${ZLIB_PRIVATE_HDRS})

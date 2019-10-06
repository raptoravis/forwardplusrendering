cmake_minimum_required(VERSION 2.6)
project(ZLIB)


##########################################################################################
# XMake-related settings

# Set the output paths
if (NOT DEFINED XMAKE_BINARY_DIR)
    set(XMAKE_BINARY_DIR "${ZLIB_BINARY_DIR}")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${XMAKE_BINARY_DIR}/bin")
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${XMAKE_BINARY_DIR}/lib")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${XMAKE_BINARY_DIR}/bin")
endif()

# Set the dependencies path
if (NOT DEFINED XMAKE_DEPENDENCIES_DIR)
    set(XMAKE_DEPENDENCIES_DIR "${ZLIB_SOURCE_DIR}/dependencies")
endif()


##########################################################################################
# XMake importation

if (NOT EXISTS "${XMAKE_DEPENDENCIES_DIR}/XMake/XMake.cmake")
    message(FATAL_ERROR
"Missing dependency: XMake
This distribution of ZLib requires XMake as a dependency.

It is provided as GIT submodule of this repository.

Did you forgot to execute the following commands?
   git submodule init
   git submodule update")
endif()

include("${XMAKE_DEPENDENCIES_DIR}/XMake/XMake.cmake")


##########################################################################################
# Declaration of the library

# List the header files
set(HEADERS crc32.h
            deflate.h
            inffast.h
            inffixed.h
            inflate.h
            inftrees.h
            trees.h
            zconf.h
            zlib.h
            zutil.h
)

# List the source files
set(SRCS adler32.c
         compress.c
         crc32.c
         deflate.c
         gzio.c
         infback.c
         inffast.c
         inflate.c
         inftrees.c
         trees.c
         uncompr.c
         zutil.c
)

# List the include paths
set(INCLUDE_PATHS "${ZLIB_SOURCE_DIR}")

include_directories(${INCLUDE_PATHS})

# Declaration of the library
xmake_create_static_library(ZLIB zlib_my ${HEADERS} ${SRCS})

# Exportations
xmake_export_include_paths(ZLIB ${INCLUDE_PATHS})

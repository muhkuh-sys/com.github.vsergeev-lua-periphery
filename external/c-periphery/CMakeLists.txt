cmake_minimum_required(VERSION 2.8)

PROJECT("c-periphery")

INCLUDE(ExternalProject)


SET(VERSION_EXTERNAL "2.2.2")


#----------------------------------------------------------------------------
#
# Build the project for the target architecture.
#

SET(PRJ_CMAKE_ARGS "")
LIST(APPEND PRJ_CMAKE_ARGS "-DBUILD_TESTS=OFF")
LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=''")

ExternalProject_Add(TARGET_cperiphery
                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/cperiphery-${VERSION_EXTERNAL}
                    URL ${CMAKE_CURRENT_SOURCE_DIR}/c-periphery-${VERSION_EXTERNAL}.tar.gz
                    URL_HASH SHA1=8b39ab80b1657fe07d3655f31f6784ef7b72242d
                    CMAKE_ARGS ${PRJ_CMAKE_ARGS}
                    INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install DESTDIR=${CMAKE_CURRENT_BINARY_DIR}
)

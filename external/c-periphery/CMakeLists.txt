cmake_minimum_required(VERSION 2.8)

PROJECT("c-periphery")

INCLUDE(ExternalProject)


SET(VERSION_EXTERNAL "2.0.1")


#----------------------------------------------------------------------------
#
# Build the project for the target architecture.
#

SET(PRJ_CMAKE_ARGS "")
LIST(APPEND PRJ_CMAKE_ARGS "-DVERSION_EXTERNAL=${VERSION_EXTERNAL}")
LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=''")

ExternalProject_Add(TARGET_cperiphery
                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/cperiphery-${VERSION_EXTERNAL}
                    URL ${CMAKE_CURRENT_SOURCE_DIR}/c-periphery-${VERSION_EXTERNAL}.tar.gz
                    URL_HASH SHA1=3bc26ff5017c251090fa5c444ade7ea009f0dc5b
                    PATCH_COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/../cmake/tools/apply_patches.py --working-folder ${CMAKE_CURRENT_BINARY_DIR}/cperiphery-${VERSION_EXTERNAL}/src/TARGET_cperiphery --copy-folder ${CMAKE_CURRENT_SOURCE_DIR}/copy
                    CMAKE_ARGS ${PRJ_CMAKE_ARGS}
                    INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install DESTDIR=${CMAKE_CURRENT_BINARY_DIR}
)

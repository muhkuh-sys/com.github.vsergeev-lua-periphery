cmake_minimum_required(VERSION 3.2.3) #required for TARGET_INCLUDE_DIRECTORIES

PROJECT("cperiphery"
        VERSION ${VERSION_EXTERNAL})

#-----------------------------------------------------------------------------
#
# Build the mhash library.
#
SET(cperiphery_SOURCES
	src/gpio.c
	src/i2c.c
	src/mmio.c
	src/serial.c
	src/spi.c
	src/version.c
)

# Build the library.
ADD_LIBRARY(TARGET_cperiphery STATIC ${cperiphery_SOURCES})
#TARGET_INCLUDE_DIRECTORIES(TARGET_cperiphery PUBLIC src)

# This static library will be linked with a shared object later. Add "-fPIC" on GCC systems or X64 builds break.
IF("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU"  AND  "${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
	SET_TARGET_PROPERTIES(TARGET_cperiphery PROPERTIES COMPILE_FLAGS -fPIC)
ENDIF("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU"  AND  "${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")


#-----------------------------------------------------------------------------
#
# Install all components of the library.
# This is the library itself and the public headers.
#
SET(CMAKE_INSTALL_PREFIX /install)
INSTALL(TARGETS TARGET_cperiphery
        EXPORT EXPORT_package
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)
INSTALL(FILES src/gpio.h src/i2c.h src/mmio.h src/serial.h src/spi.h src/version.h
        DESTINATION include/c-periphery/src/)


#----------------------------------------------------------------------------
#
# Build the CMAKE package.
#
INCLUDE(CMakePackageConfigHelpers)

SET(INCLUDE_INSTALL_DIR include/)
CONFIGURE_PACKAGE_CONFIG_FILE(
	cmake/config.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/package/${PROJECT_NAME}-config.cmake"
	INSTALL_DESTINATION cmake
	PATH_VARS INCLUDE_INSTALL_DIR
)

WRITE_BASIC_PACKAGE_VERSION_FILE(
	"${CMAKE_CURRENT_BINARY_DIR}/package/${PROJECT_NAME}-version.cmake"
	COMPATIBILITY ExactVersion
)

EXPORT(
	EXPORT EXPORT_package
	FILE "${CMAKE_CURRENT_BINARY_DIR}/package/${PROJECT_NAME}-targets.cmake"
	NAMESPACE ${PROJECT_NAME}::
)

INSTALL(
	EXPORT EXPORT_package
	FILE ${PROJECT_NAME}-targets.cmake
	NAMESPACE ${PROJECT_NAME}::
	DESTINATION cmake
)
INSTALL(
	FILES "${CMAKE_CURRENT_BINARY_DIR}/package/${PROJECT_NAME}-config.cmake" "${CMAKE_CURRENT_BINARY_DIR}/package/${PROJECT_NAME}-version.cmake"
	DESTINATION cmake
	COMPONENT Devel
)

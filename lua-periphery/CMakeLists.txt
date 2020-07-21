cmake_minimum_required(VERSION 2.8)

PROJECT("lua-periphery")

INCLUDE(ExternalProject)

SET(VERSION_EXTERNAL "2.0.1")
SET(VERSION_PACKAGE  "1")
SET(PROJECT_VERSION  "${VERSION_EXTERNAL}.${VERSION_PACKAGE}")

# Get the VCS version for the jonchki configuration.
INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/version.cmake)

# Filter the jonchki configuration.
CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-periphery.xml
               ${CMAKE_BINARY_DIR}/lua${BUILDCFG_LUA_VERSION}-lua-periphery-${PROJECT_VERSION}.xml
               @ONLY)

#----------------------------------------------------------------------------
#
# Build the project.
#
IF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
	# Select the LUA version.
	INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/muhkuh_lua.cmake)

	SET(cperiphery_DIR "${CMAKE_BINARY_DIR}/../external/c-periphery/install/cmake")
	FIND_PACKAGE(cperiphery REQUIRED CONFIG)
	GET_TARGET_PROPERTY(CPERIPHERY_LIBS cperiphery::TARGET_cperiphery LOCATION)
	SET(CPERIPHERY_INCLUDE_DIR ${cperiphery_INCLUDE_DIR})

	SET(PRJ_CMAKE_ARGS "")
	LIST(APPEND PRJ_CMAKE_ARGS "-DVERSION_EXTERNAL=${VERSION_EXTERNAL}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DLUA_LIBRARIES=${LUA_LIBRARIES}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DLUA_INCLUDE_DIR=${LUA_INCLUDE_DIR}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DCPERIPHERY_INCLUDE_DIR=${CPERIPHERY_INCLUDE_DIR}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DCPERIPHERY_LIBS=${CPERIPHERY_LIBS}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=''")

	ExternalProject_Add(TARGET_luaperiphery
	                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/luaperiphery-${VERSION_EXTERNAL}
	                    URL ${CMAKE_CURRENT_SOURCE_DIR}/lua-periphery-${VERSION_EXTERNAL}.tar.gz
	                    URL_HASH SHA1=cc5d0dd8b6d1463c9e3c003df0c22116c6198d88
	                    PATCH_COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/apply_patches.py --working-folder ${CMAKE_CURRENT_BINARY_DIR}/luaperiphery-${VERSION_EXTERNAL}/src/TARGET_luaperiphery --copy-folder ${CMAKE_CURRENT_SOURCE_DIR}/copy
	                    CMAKE_ARGS ${PRJ_CMAKE_ARGS}
	                    INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/install
	)


	#----------------------------------------------------------------------------
	#
	# Build a package for the selected platform.
	#

	SET(CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/../../repository/com/github/vsergeev/lua-periphery/${PROJECT_VERSION})
	SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
	SET(CPACK_PACKAGE_FILE_NAME "lua${BUILDCFG_LUA_VERSION}-lua-periphery-${PROJECT_VERSION}-${JONCHKI_PLATFORM}")

	SET(CPACK_GENERATOR "TXZ")
	INCLUDE(CPack)

	# This is the folder where the external project TARGET_bitop installed the files.
	SET(SUB_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/install")

	# Package a linux build.
	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/install_linux.lua
	               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	               @ONLY)

	INSTALL(FILES ${SUB_INSTALL_DIR}/lua_plugins/periphery${CMAKE_SHARED_MODULE_SUFFIX}
	        DESTINATION lua_plugins/)

	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	        DESTINATION .)


	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-periphery.xml
	               ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-periphery-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	               @ONLY)


	#-----------------------------------------------------------------------------
	#
	# Create a custom target named "pack".
	# It builds the package and creates the hashes.
	#
	ADD_CUSTOM_TARGET(pack
	                  COMMAND ${CMAKE_MAKE_PROGRAM}
	                  COMMAND ${CMAKE_MAKE_PROGRAM} package
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-periphery-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-periphery-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.tar.xz
	                  COMMENT "Pack the artifact and generate the hashes...")


	#-----------------------------------------------------------------------------
	#
	# Filter the pom.xml file.
	#
	SET(POM_ARTIFACT_ID  "lua-periphery")
	SET(POM_GROUP_ID     "com.github.vsergeev")
	SET(POM_VERSION      "${PROJECT_VERSION}")
	SET(POM_PACKAGING    "tar.xz")
	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/pom.xml ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-periphery-${PROJECT_VERSION}.pom)
ENDIF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))

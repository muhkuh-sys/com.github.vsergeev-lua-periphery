CMAKE_MINIMUM_REQUIRED(VERSION 3.7)

PROJECT("lua-periphery")

ENABLE_TESTING()
FIND_PACKAGE(PythonInterp 2.7 REQUIRED)

SET(INSTALL_DIR_LUA_MODULES "lua_plugins")

SET(SOURCES
	src/lua_gpio.c
	src/lua_i2c.c
	src/lua_mmio.c
	src/lua_periphery.c
	src/lua_serial.c
	src/lua_spi.c
)

MESSAGE("LUA_INCLUDE_DIR: ${LUA_INCLUDE_DIR}")
MESSAGE("CPERIPHERY_INCLUDE_DIR: ${CPERIPHERY_INCLUDE_DIR}")

# Build the Lua module.
ADD_LIBRARY(TARGET_luaperiphery MODULE
            ${SOURCES})
TARGET_INCLUDE_DIRECTORIES(TARGET_luaperiphery
                           PRIVATE ${LUA_INCLUDE_DIR} ${CPERIPHERY_INCLUDE_DIR})
TARGET_LINK_LIBRARIES(TARGET_luaperiphery ${LUA_LIBRARIES} ${CPERIPHERY_LIBS})

# Set the name of the output file to "periphery".
SET_TARGET_PROPERTIES(TARGET_luaperiphery PROPERTIES PREFIX "" OUTPUT_NAME "periphery")

# On mingw link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_luaperiphery PROPERTY LINK_FLAGS "-static -static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install the lua module.
INSTALL(TARGETS TARGET_luaperiphery
        EXPORT EXPORT_package
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES}
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

# Add tests for this module.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	# Here are the MinGW specific tests.
	ADD_TEST(NAME luaperiphery_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXECUTABLE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 $<TARGET_FILE:TARGET_luaperiphery>)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

CMAKE_MINIMUM_REQUIRED(VERSION 3.7)

PROJECT("lua-periphery")

SET(INSTALL_DIR_LUA_MODULES "lua_plugins")

SET(SOURCES
	src/lua_periphery.c
	src/lua_mmio.c
	src/lua_gpio.c
	src/lua_led.c
	src/lua_pwm.c
	src/lua_spi.c
	src/lua_i2c.c
	src/lua_serial.c
)

MESSAGE("LUA_INCLUDE_DIR: ${LUA_INCLUDE_DIR}")
MESSAGE("CPERIPHERY_INCLUDE_DIR: ${CPERIPHERY_INCLUDE_DIR}")

# Build the Lua module.
ADD_LIBRARY(TARGET_luaperiphery MODULE
            ${SOURCES})
TARGET_INCLUDE_DIRECTORIES(TARGET_luaperiphery
                           PRIVATE ${LUA_INCLUDE_DIR} ${CPERIPHERY_INCLUDE_DIR})
TARGET_LINK_LIBRARIES(TARGET_luaperiphery ${CPERIPHERY_LIB})
TARGET_COMPILE_OPTIONS(TARGET_luaperiphery
                       PRIVATE -std=c99 -pedantic -D_XOPEN_SOURCE=700 -Wall -Wextra -Wno-unused-parameter -fPIC)

# Set the name of the output file to "periphery".
SET_TARGET_PROPERTIES(TARGET_luaperiphery PROPERTIES PREFIX "" OUTPUT_NAME "periphery")

# Install the lua module.
INSTALL(TARGETS TARGET_luaperiphery
        EXPORT EXPORT_package
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES}
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

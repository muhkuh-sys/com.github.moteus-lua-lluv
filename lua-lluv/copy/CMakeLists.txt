CMAKE_MINIMUM_REQUIRED(VERSION 3.7)

PROJECT("org.luajit-bitop")

ENABLE_TESTING()
FIND_PACKAGE(PythonInterp 2.7 REQUIRED)

SET(INSTALL_DIR_LUA_MODULES "lua_plugins")

# TODO: no absolute paths here.
#SET(LUA_INCLUDE_DIR /home/baccy/workspace/lua_lluv_experiments/jonchki_lua/dev/include/lua5.1)
#SET(LUA_LIBRARIES /home/baccy/workspace/lua_lluv_experiments/jonchki_lua/lua5.1.so)
#SET(LIBUV_INCLUDE_DIR /home/baccy/workspace/lua_lluv_experiments/libuv/libuv-1.22.0/build/install/usr/local/include)
#SET(LIBUV_LIBRARIES /home/baccy/workspace/lua_lluv_experiments/libuv/libuv-1.22.0/build/install/usr/local/lib/libuv_a.a -pthread)

SET(SOURCES
	src/lluv_utils.c
	src/lluv.c
	src/lluv_error.c
	src/lluv_fbuf.c
	src/lluv_fs.c
	src/lluv_handle.c
	src/lluv_stream.c
	src/lluv_idle.c
	src/lluv_loop.c
	src/lluv_tcp.c
	src/lluv_timer.c
	src/lluv_pipe.c
	src/lluv_tty.c
	src/lluv_udp.c
	src/lluv_prepare.c
	src/lluv_check.c
	src/lluv_poll.c
	src/lluv_signal.c
	src/lluv_fs_event.c
	src/lluv_fs_poll.c
	src/lluv_req.c
	src/lluv_misc.c
	src/lluv_process.c
	src/lluv_dns.c
	src/l52util.c
	src/lluv_list.c
)


IF(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	SET(LLUV_LIBS ${LIBUV_LIBRARIES} iphlpapi kernel32 psapi userenv ws2_32)
ELSE(CMAKE_SYSTEM_NAME STREQUAL "Windows")
	SET(LLUV_LIBS ${LIBUV_LIBRARIES} pthread)
ENDIF(CMAKE_SYSTEM_NAME STREQUAL "Windows")

# Build the Lua module.
ADD_LIBRARY(TARGET_lluv MODULE
            ${SOURCES})
TARGET_INCLUDE_DIRECTORIES(TARGET_lluv
                           PRIVATE ${LUA_INCLUDE_DIR} ${LIBUV_INCLUDE_DIR})
TARGET_LINK_LIBRARIES(TARGET_lluv ${LLUV_LIBS})
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	TARGET_LINK_LIBRARIES(TARGET_lluv ${LUA_LIBRARIES})
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Set the name of the output file to "lluv".
SET_TARGET_PROPERTIES(TARGET_lluv PROPERTIES PREFIX "" OUTPUT_NAME "lluv")

# On mingw link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_lluv PROPERTY LINK_FLAGS "-static -static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install the lua module.
INSTALL(TARGETS TARGET_lluv
        EXPORT EXPORT_package
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES}
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

# Add tests for this module.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	# Here are the MinGW specific tests.
	ADD_TEST(NAME lluv_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXECUTABLE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 $<TARGET_FILE:TARGET_lluv>)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

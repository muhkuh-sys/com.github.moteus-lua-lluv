CMAKE_MINIMUM_REQUIRED(VERSION 3.7)

PROJECT("com.github.moteus-lua-lluv")

SET(INSTALL_DIR_LUA_MODULES "lua_plugins")

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


SET(LLUV_LIBS ${LIBUV_LIBRARIES} pthread)

# Build the Lua module.
ADD_LIBRARY(TARGET_lluv MODULE
            ${SOURCES})
TARGET_INCLUDE_DIRECTORIES(TARGET_lluv
                           PRIVATE ${LUA_INCLUDE_DIR} ${LIBUV_INCLUDE_DIR})
TARGET_LINK_LIBRARIES(TARGET_lluv ${LLUV_LIBS})

# Set the name of the output file to "lluv".
SET_TARGET_PROPERTIES(TARGET_lluv PROPERTIES PREFIX "" OUTPUT_NAME "lluv")

# Install the lua module.
INSTALL(TARGETS TARGET_lluv
        EXPORT EXPORT_package
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES}
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

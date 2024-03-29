cmake_minimum_required(VERSION 3.2.3)

PROJECT("lua-lluv")

INCLUDE(ExternalProject)

SET(VERSION_EXTERNAL "0.1.10")
SET(VERSION_PACKAGE  "5")
SET(PROJECT_VERSION  "${VERSION_EXTERNAL}.${VERSION_PACKAGE}")

# Get the VCS version for the jonchki configuration.
INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/version.cmake)

# Filter the jonchki configuration.
CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-lluv.xml
               ${CMAKE_BINARY_DIR}/lua${BUILDCFG_LUA_VERSION}-lua-lluv-${PROJECT_VERSION}.xml
               @ONLY)


#----------------------------------------------------------------------------
#
# Build the project.
#

IF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
	# Select the LUA version.
	INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/muhkuh_lua.cmake)

	IF(CMAKE_VERSION VERSION_LESS 3.12.0)
		FIND_PACKAGE(PythonInterp 3.5 REQUIRED)
		SET(PYTHON_EXE "${PYTHON_EXECUTABLE}")
	ELSE(CMAKE_VERSION VERSION_LESS 3.12.0)
		FIND_PACKAGE(Python3 REQUIRED)
		SET(PYTHON_EXE "${Python3_EXECUTABLE}")
	ENDIF(CMAKE_VERSION VERSION_LESS 3.12.0)

	SET(SUB_CMAKE_ARGS "")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
	IF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
		LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_RC_COMPILER=${CMAKE_RC_COMPILER}")
	ENDIF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
	LIST(APPEND SUB_CMAKE_ARGS "-DLUA_LIBRARIES=${LUA_LIBRARIES}")
	LIST(APPEND SUB_CMAKE_ARGS "-DLUA_INCLUDE_DIR=${LUA_INCLUDE_DIR}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=''")
	LIST(APPEND SUB_CMAKE_ARGS "-DLIBUV_LIBRARIES=${CMAKE_BINARY_DIR}/build_requirements/jonchki/install/dev/lib/libuv.a")
	LIST(APPEND SUB_CMAKE_ARGS "-DLIBUV_INCLUDE_DIR=${CMAKE_BINARY_DIR}/build_requirements/jonchki/install/dev/include/")

	ExternalProject_Add(TARGET_lua_lluv
	                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/lua-lluv-${VERSION_EXTERNAL}
	                    URL ${CMAKE_CURRENT_SOURCE_DIR}/lua-lluv-${VERSION_EXTERNAL}.tar.gz
	                    URL_HASH SHA1=31a297bb76b04000db978cdd485c79bc543ce4db
	                    PATCH_COMMAND "${PYTHON_EXE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/apply_patches.py --working-folder ${CMAKE_CURRENT_BINARY_DIR}/lua-lluv-${VERSION_EXTERNAL}/src/TARGET_lua_lluv --copy-folder ${CMAKE_CURRENT_SOURCE_DIR}/copy
	                    CMAKE_ARGS ${SUB_CMAKE_ARGS}
	                    INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/install
	)


	#----------------------------------------------------------------------------
	#
	# Build a package for the selected platform.
	#

	SET(CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/../../repository/com/github/moteus/lua-lluv/${PROJECT_VERSION})
	SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
	SET(CPACK_PACKAGE_FILE_NAME "lua${BUILDCFG_LUA_VERSION}-lua-lluv-${PROJECT_VERSION}-${JONCHKI_PLATFORM}")

	SET(CPACK_GENERATOR "TXZ")
	INCLUDE(CPack)

	# This is the folder where the external project TARGET_bitop installed the files.
	SET(SUB_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/install")

	# Package a linux build.
	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/install_linux.lua
	               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	               @ONLY)

	INSTALL(FILES ${SUB_INSTALL_DIR}/lua_plugins/lluv${CMAKE_SHARED_MODULE_SUFFIX}
	        DESTINATION lua_plugins/)
	INSTALL(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/lua-lluv-${VERSION_EXTERNAL}/src/TARGET_lua_lluv/src/lua
	        DESTINATION .)

	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	        DESTINATION .)


	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-lluv.xml
	               ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-lluv-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	               @ONLY)


	#-----------------------------------------------------------------------------
	#
	# Create a custom target named "pack".
	# It builds the package and creates the hashes.
	#
	ADD_CUSTOM_TARGET(pack
	                  COMMAND ${CMAKE_MAKE_PROGRAM}
	                  COMMAND ${CMAKE_MAKE_PROGRAM} package
	                  COMMAND "${PYTHON_EXE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-lluv-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	                  COMMAND "${PYTHON_EXE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-lluv-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.tar.xz
	                  COMMENT "Pack the artifact and generate the hashes...")


	#-----------------------------------------------------------------------------
	#
	# Filter the pom.xml file.
	#
	SET(POM_ARTIFACT_ID  "lua-lluv")
	SET(POM_GROUP_ID     "com.github.moteus")
	SET(POM_VERSION      "${PROJECT_VERSION}")
	SET(POM_PACKAGING    "tar.xz")
	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/pom.xml ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-lluv-${PROJECT_VERSION}.pom)
ENDIF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))

# Python is needed for the hash tool.
FIND_PACKAGE(Python3 REQUIRED COMPONENTS Interpreter)
SET(PYTHON_EXECUTABLE "${Python3_EXECUTABLE}")

INCLUDE(ExternalProject)

# Filter the jonchki configuration.
CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-openssl.xml
               ${CMAKE_BINARY_DIR}/lua${BUILDCFG_LUA_VERSION}-lua-openssl-${PROJECT_VERSION}.xml
               @ONLY)

#----------------------------------------------------------------------------
#
# Build the project.
#
IF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
	# Select the LUA version.
	INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/muhkuh_lua.cmake)

	SET(PRJ_CMAKE_ARGS "")

	LIST(APPEND PRJ_CMAKE_ARGS "-Dlua5.4_DIR=${CMAKE_BINARY_DIR}/build_requirements/jonchki/install/dev/lib/cmake/org.lua.lua")
	LIST(APPEND PRJ_CMAKE_ARGS "-DLibreSSL_DIR=${CMAKE_BINARY_DIR}/build_requirements/jonchki/install/dev/lib/cmake/LibreSSL")

	LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=''")

	ExternalProject_Add(TARGET_luaopenssl
	                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/lua-openssl
	                    URL ${CMAKE_CURRENT_SOURCE_DIR}/lua-openssl-GIT3ed89e68d517.tar.gz
	                    URL_HASH SHA1=b11bbdecb356701ffb11a887d071ee6647a974fc
	                    PATCH_COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/apply_patches.py --working-folder ${CMAKE_CURRENT_BINARY_DIR}/lua-openssl/src/TARGET_luaopenssl --patch-folder ${CMAKE_CURRENT_SOURCE_DIR}/patches --strip 1
	                    CMAKE_ARGS ${PRJ_CMAKE_ARGS}
	                    INSTALL_COMMAND make install DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/install
	)

	#----------------------------------------------------------------------------
	#
	# Build a package for the selected platform.
	#

	SET(CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/../../repository/com/github/zhaozg/lua-openssl/${PROJECT_VERSION})
	SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
	SET(CPACK_PACKAGE_FILE_NAME "lua-openssl-${PROJECT_VERSION}-${JONCHKI_PLATFORM}")

	SET(CPACK_GENERATOR "TXZ")
	INCLUDE(CPack)

	# This is the folder where the external project installed the files.
	SET(SUB_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/install")
	MESSAGE("SUB_INSTALL_DIR: ${SUB_INSTALL_DIR}")

	INSTALL(FILES ${SUB_INSTALL_DIR}/lib/lua/openssl.so
	  DESTINATION ${INSTALL_DIR_LUA_MODULES}
	)

	CONFIGURE_FILE(installer/install.lua
	  ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	  @ONLY
	)

	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
	        DESTINATION .)


	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-openssl.xml
	               ${CPACK_OUTPUT_FILE_PREFIX}/lua-openssl-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	               @ONLY)


	#-----------------------------------------------------------------------------
	#
	# Create a custom target named "pack".
	# It builds the package and creates the hashes.
	#
	ADD_CUSTOM_TARGET(pack
	                  COMMAND ${CMAKE_MAKE_PROGRAM}
	                  COMMAND ${CMAKE_MAKE_PROGRAM} package
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua-openssl-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua-openssl-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.tar.xz
	                  COMMENT "Pack the artifact and generate the hashes...")
ENDIF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))

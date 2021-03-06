include(ExternalProject)

if(CMAKE_C_COMPILER)
	set(CMAKE_C_COMPILER_OPTION "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}")
endif()
if(CMAKE_CXX_COMPILER)
	set(CMAKE_CXX_COMPILER_OPTION "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
endif()

if(CAPSTONE_LOCAL_DIR)
	message(STATUS "Capstone: using local Capstone directory.")

	ExternalProject_Add(capstone-project
		DOWNLOAD_COMMAND ""
		SOURCE_DIR "${CAPSTONE_LOCAL_DIR}"
		CMAKE_ARGS
			# This does not work on MSVC, but may be useful on Linux.
			-DCMAKE_BUILD_TYPE=Release
			-DCAPSTONE_BUILD_STATIC=ON
			-DCAPSTONE_BUILD_SHARED=OFF
			-DCAPSTONE_BUILD_STATIC_RUNTIME=OFF
			-DCAPSTONE_BUILD_TESTS=OFF
			-DCAPSTONE_X86_ATT_DISABLE=OFF
			# Enabled architectures.
			-DCAPSTONE_ARM_SUPPORT=ON
			-DCAPSTONE_MIPS_SUPPORT=ON
			-DCAPSTONE_PPC_SUPPORT=ON
			-DCAPSTONE_X86_SUPPORT=ON
			# Disabled architectures.
			-DCAPSTONE_ARM64_SUPPORT=OFF
			-DCAPSTONE_M68K_SUPPORT=OFF
			-DCAPSTONE_SPARC_SUPPORT=OFF
			-DCAPSTONE_SYSZ_SUPPORT=OFF
			-DCAPSTONE_XCORE_SUPPORT=OFF
			-DCAPSTONE_TMS320C64X_SUPPORT=OFF
			-DCAPSTONE_M680X_SUPPORT=OFF
			# Force the use of the same compiler as used to build the top-level
			# project. Otherwise, the external project may pick up a different
			# compiler, which may result in link errors.
			"${CMAKE_C_COMPILER_OPTION}"
			"${CMAKE_CXX_COMPILER_OPTION}"
		# Patch the Capstone sources.
		PATCH_COMMAND
			${CMAKE_COMMAND} -Dcapstone_path=<SOURCE_DIR> -P ${CMAKE_CURRENT_SOURCE_DIR}/patch.cmake
		# Disable the update step.
		UPDATE_COMMAND ""
		# Disable the install step.
		INSTALL_COMMAND ""
	)
	force_configure_step(capstone-project)
else()
	message(STATUS "Capstone: using remote Capstone revision.")

	ExternalProject_Add(capstone-project
		URL ${CAPSTONE_URL}
		URL_HASH SHA256=${CAPSTONE_ARCHIVE_SHA256}
		DOWNLOAD_NAME capstone.zip
		CMAKE_ARGS
			# This does not work on MSVC, but may be useful on Linux.
			-DCMAKE_BUILD_TYPE=Release
			-DCAPSTONE_BUILD_STATIC=ON
			-DCAPSTONE_BUILD_SHARED=OFF
			-DCAPSTONE_BUILD_STATIC_RUNTIME=OFF
			-DCAPSTONE_BUILD_TESTS=OFF
			-DCAPSTONE_X86_ATT_DISABLE=OFF
			# Enabled architectures.
			-DCAPSTONE_ARM_SUPPORT=ON
			-DCAPSTONE_MIPS_SUPPORT=ON
			-DCAPSTONE_PPC_SUPPORT=ON
			-DCAPSTONE_X86_SUPPORT=ON
			-DCAPSTONE_ARM64_SUPPORT=ON
			# Disabled architectures.
			-DCAPSTONE_M68K_SUPPORT=OFF
			-DCAPSTONE_SPARC_SUPPORT=OFF
			-DCAPSTONE_SYSZ_SUPPORT=OFF
			-DCAPSTONE_XCORE_SUPPORT=OFF
			-DCAPSTONE_TMS320C64X_SUPPORT=OFF
			-DCAPSTONE_M680X_SUPPORT=OFF
			# Force the use of the same compiler as used to build the top-level
			# project. Otherwise, the external project may pick up a different
			# compiler, which may result in link errors.
			"${CMAKE_C_COMPILER_OPTION}"
			"${CMAKE_CXX_COMPILER_OPTION}"
		# Patch the Capstone sources.
		PATCH_COMMAND
			${CMAKE_COMMAND} -Dcapstone_path=<SOURCE_DIR> -P ${CMAKE_CURRENT_SOURCE_DIR}/patch.cmake
		# Disable the update step.
		UPDATE_COMMAND ""
		# Disable the install step.
		INSTALL_COMMAND ""
		LOG_DOWNLOAD ON
		LOG_CONFIGURE ON
		LOG_BUILD ON
	)
endif()

check_if_variable_changed(CAPSTONE_LOCAL_DIR CHANGED)
if(CHANGED)
	ExternalProject_Get_Property(capstone-project binary_dir)
	message(STATUS "Capstone: path to Capstone directory changed -> cleaning CMake files in ${binary_dir}.")
	clean_cmake_files(${binary_dir})
endif()

ExternalProject_Get_Property(capstone-project source_dir)
ExternalProject_Get_Property(capstone-project binary_dir)

# Add libraries.
add_library(capstone INTERFACE)
add_dependencies(capstone capstone-project)
target_include_directories(capstone SYSTEM INTERFACE ${source_dir}/include)
target_include_directories(capstone SYSTEM INTERFACE ${source_dir}/arch)
target_link_libraries(capstone INTERFACE debug ${binary_dir}/${DEBUG_DIR}${CMAKE_STATIC_LIBRARY_PREFIX}capstone${CMAKE_STATIC_LIBRARY_SUFFIX})
target_link_libraries(capstone INTERFACE optimized ${binary_dir}/${RELEASE_DIR}${CMAKE_STATIC_LIBRARY_PREFIX}capstone${CMAKE_STATIC_LIBRARY_SUFFIX})

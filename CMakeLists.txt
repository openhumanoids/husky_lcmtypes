project(husky-lcmtypes)
cmake_minimum_required(VERSION 3.1)

include(CMakePackageConfigHelpers)
include(GenerateExportHeader)

find_package(PythonInterp)
find_package(Java)

if(JAVA_FOUND)
  include(UseJava)
endif()

# Enable ELF hidden visibility
set(CMAKE_C_VISIBILITY_PRESET "hidden")
set(CMAKE_CXX_VISIBILITY_PRESET "hidden")
set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)

if(POLICY CMP0063)
  cmake_policy(SET CMP0063 NEW)
endif()

# TODO remove when minimum CMake >= 3.7
if(CMAKE_VERSION VERSION_LESS 3.7)
  list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
endif()

find_package(lcm)
if(lcm_FOUND)
	message(WARNING "Found CMake LCM (>1.3.1)")
	include(${LCM_USE_FILE})
else()
	message(WARNING "CMake-LCM not found, falling back to Autotools-LCM")
	
	find_package(PkgConfig REQUIRED)
    pkg_check_modules(LCM REQUIRED lcm)
    find_program(LCM_GEN_EXECUTABLE lcm-gen ${EXECUTABLE_OUTPUT_PATH} ${EXECUTABLE_INSTALL_PATH})
    if (NOT LCM_GEN_EXECUTABLE)
    	message(FATAL_ERROR "lcm-gen not found!\n")
    	return()
    endif()

	include(cmake/lcmtypes.cmake)
	lcmtypes_build()
endif()

add_subdirectory(lcmtypes)
add_subdirectory(pkgconfig)

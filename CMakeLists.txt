cmake_minimum_required(VERSION 3.16)

project(EZ_TEMPLATE
VERSION 0.1.0
LANGUAGES C CXX)

set(PACKAGE_NAME ez-template)
set(TARGET_NAME template)

include(GNUInstallDirs)
include(FetchContent)

option(BUILD_TESTS "Build the test executables" ON)
option(SEPARATE_DIRECTORY "Install the package into its own directory" ON)
set(CONFIG_DIR "${CMAKE_INSTALL_DATAROOTDIR}/${PACKAGE_NAME}" CACHE STRING "The relative directory to install package config files.")

FetchContent_Declare(
	ez-cmake
	GIT_REPOSITORY "https://github.com/errata-c/ez-cmake.git"
	GIT_TAG "a59477ebb9d8ce1f686b99118586310f47cb7a80"
)
FetchContent_MakeAvailable(ez-cmake)

# Find deps
#find_package(glm CONFIG REQUIRED)

# Create main target.
add_library(${TARGET_NAME} INTERFACE
	# Add source files
	#"src/IntersectFunctions.cpp"
)

# Add the include directory
target_include_directories(${TARGET_NAME} INTERFACE 

	# Build only include directories
	"$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"

	# Install include directories
	"$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)

# Select the c++ version to use.
target_compile_features(${TARGET_NAME} INTERFACE cxx_std_17)

# Just for internal compatibility, so that subprojects can use the namespaced version
add_library("ez::${TARGET_NAME}" ALIAS "${TARGET_NAME}")

# Build the tests once the option is enabled.
if(${BUILD_TESTS})
	add_subdirectory("${CMAKE_CURRENT_SOURCE_DIR}/tests")
endif()

# Set the proper installation path
if((DEFINED $ENV{CMAKE_PREFIX_PATH}) AND ($ENV{CMAKE_PREFIX_PATH} STREQUAL ${CMAKE_INSTALL_PREFIX}))
	set(CMAKE_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}/${CMAKE_PROJECT_NAME})
endif()

install(
	DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/"
	TYPE INCLUDE
	FILES_MATCHING
	PATTERN "*.h" PATTERN "*.hpp"
)

install(TARGETS ${TARGET_NAME}
	EXPORT ${TARGET_NAME}-targets
	RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}/$<CONFIG>"
	ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}/$<CONFIG>"
	LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}/$<CONFIG>"
)

install_package(
	NAME "${PACKAGE_NAME}"
	EXPORT "${TARGET_NAME}-targets"
	VERSION "${CMAKE_PROJECT_VERSION}"

	NAMESPACE "ez::"

	# Version compatiblity
	COMPATIBILITY "SameMajorVersion"

	# Relative install location of the package config files.
	DESTINATION "${CONFIG_DIR}"

	# Pre and Post configuration files for the packages. Files are run as scripts when the package is found.
	# Most dependencies must be found in the preconfig file, except non-target dependencies
	# Non-target dependencies should probably be found in the post config, and added onto the imported target manually.
	# Non-target dependencies kinda suck, avoid them if you can.

	#PRECONFIG "${CMAKE_CURRENT_SOURCE_DIR}/cmake/preconfig.cmake"
	#POSTCONFIG "${CMAKE_CURRENT_SOURCE_DIR}/cmake/postconfig.cmake"

	# Only add this option if the library is header only, or an interface library.
	ARCH_INDEPENDENT
)
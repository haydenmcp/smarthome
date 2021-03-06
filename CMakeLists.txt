cmake_minimum_required(VERSION 3.14)
project(
    smarthome-distribution
    DESCRIPTION "A stable and secure smart home implementation."
    HOMEPAGE_URL "https://github.com/haydenmcp/smarthome"
    LANGUAGES CXX
    VERSION 0.0.1)
include(CMakeDependentOption)
include(InstallRequiredSystemLibraries)
include(GNUInstallDirs)
include(FetchContent)
set(FETCHCONTENT_QUIET OFF)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D_GLIBCXX_DEBUG")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON CACHE BOOL "Ensure all windows symbols are exported." FORCE)


file(TO_CMAKE_PATH "$ENV{SystemDrive}/smarthome" SMARTHOME_HOME)
set(SMARTHOME_HOME "${SMARTHOME_HOME}" CACHE PATH "" FORCE)

file(TO_CMAKE_PATH "${SMARTHOME_HOME}/data" SMARTHOME_DATA)
set(SMARTHOME_DATA "${SMARTHOME_DATA}" CACHE PATH "" FORCE)

file(TO_CMAKE_PATH "${SMARTHOME_HOME}/programs" SMARTHOME_PROGRAMS)
set(SMARTHOME_PROGRAMS "${SMARTHOME_PROGRAMS}" CACHE PATH "" FORCE)

# On first build in a given tree initialize default install prefix
set(CMAKE_INSTALL_PREFIX "${SMARTHOME_PROGRAMS}/smarthome" CACHE PATH "" FORCE)

file(RELATIVE_PATH SMARTHOME_SUBPROJECT_INSTALL_RELATIVE_RPATH
    "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}"
    "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_INSTALL_RPATH "${ORIGIN}" "${ORIGIN}/${SMARTHOME_SUBPROJECT_INSTALL_RELATIVE_RPATH}")

option(SMARTHOME_MASTER_PROJECT "Indicates whether smarthome is currently the main project." OFF)
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    set(SMARTHOME_MASTER_PROJECT ON)
endif()

# Distribution options
option(INCLUDE_SMARTHOME_CLI "Include smarthome distribution command-line interface (CLI)." ${SMARTHOME_MASTER_PROJECT})
option(INCLUDE_SMARTHOME_DOCS "Include smarthome distribution documentation." ${SMARTHOME_MASTER_PROJECT})
option(INCLUDE_SMARTHOME_EXAMPLES "Include smarthome distribution examples." ${SMARTHOME_MASTER_PROJECT})
option(INCLUDE_SMARTHOME_TESTS "Include smarthome distribution tests." ${SMARTHOME_MASTER_PROJECT})

option(INCLUDE_DEVICE_PACKAGE "Include device package." ${SMARTHOME_MASTER_PROJECT})
option(INCLUDE_SENSOR_PACKAGE "Include sensor package." ${SMARTHOME_MASTER_PROJECT})

set(SMARTHOME_PACKAGE_DEPENDENCIES "git" "clang" "build-essential" "nodejs" "npm")

add_subdirectory("core")
add_subdirectory("logger")

if(INCLUDE_DEVICE_PACKAGE)
	add_subdirectory("device")
endif()

if(INCLUDE_SENSOR_PACKAGE)
	add_subdirectory("sensor")
endif()

if(INCLUDE_SMARTHOME_CLI)
	add_subdirectory("cli")
endif()

if(INCLUDE_SMARTHOME_DOCS)
	add_subdirectory("docs")
endif()

if(INCLUDE_SMARTHOME_EXAMPLES)
    add_subdirectory("example")
endif()

if(INCLUDE_SMARTHOME_TESTS)
    enable_testing()
endif()

if (GENERATE_SMARTHOME_PACKAGE)
    set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/license.txt")
    add_subdirectory("package")
endif()

include("setup/CMakeLists.txt")
cmake_minimum_required(VERSION 3.10)

project(ProjectName VERSION 1.0 DESCRIPTION "Project Description")

# CMake flags
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CTEST_ARGUMENTS "${CMAKE_CTEST_ARGUMENTS};--output-on-failure;--force-new-ctest-process;--verbose")

# C++ standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Prevent in-source builds
if (${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message(FATAL_ERROR "dumbass. lol (don't build in the source directory)")
endif()

# Default build type
if (CMAKE_BUILD_TYPE STREQUAL "")
  message(STATUS "Build type set to default (Debug)")
  set(CMAKE_BUILD_TYPE Debug)
endif()

# Output binaries to a subdirectory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Enable CMake testing
include(CTest)
enable_testing()

# Enable GoogleTest
include(GoogleTest)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/googletest googletest)
set(GTEST_OUTPUT "all")

# Enable Boost
set(Boost_USE_STATIC_LIBS ON)
find_package(Boost 1.50 REQUIRED COMPONENTS log log_setup)
message(STATUS "Boost version: ${Boost_VERSION}")
message(STATUS "Boost include: ${Boost_INCLUDE_DIRS}")
message(STATUS "Boost lib: ${Boost_LIBRARY_DIRS}")

# Check for -fno-elide-constructors support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-fno-elide-constructors" COMPILER_SUPPORTS_NO_ELIDE)

# Set optimization flags based on build type
set(CMAKE_CXX_FLAGS_RELEASE "-O2")
set(CMAKE_CXX_FLAGS_DEBUG "-O0 -fsanitize=address -fno-omit-frame-pointer")
set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} -fsanitize=address")

# Include source and tests
add_subdirectory(src)
add_subdirectory(tests)

include(cmake/CodeCoverageReportConfig.cmake)
generate_coverage_report(
  TARGETS
    # List of target executables
  TESTS
    # List of test executables
)

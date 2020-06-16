#-------------------------------------------------------------------------------------------
# Compile with GCC:
#   rm -rf build && CC=gcc CXX=g++ cmake -B build && cmake --build build --parallel
#
# Compile with Intel C++ Compiler:
#   rm -rf build && CC=icc CXX=icpc cmake -B build && cmake --build build --parallel
#
# Compile with Clang:
#   rm -rf build && CC=clang CXX=clang++ cmake -B build && cmake --build build --parallel
#-------------------------------------------------------------------------------------------


#-------------------------------------------------------------------------------------------
# Project configuration.
#-------------------------------------------------------------------------------------------

# Require a minimum version of cmake.
cmake_minimum_required(VERSION 3.13.5 FATAL_ERROR)

# Set the name of the project.
project("metamodsdk")

# Enable Clang-Tidy checks.
set(CLANG_TIDY -checks=-*,boost-*,bugprone-*,clang-analyzer-*,cppcoreguidelines-*,misc-*,modernize-*,mpi-*,openmp-*,performance-*,portability-*,readability-*,)

# Disable Clang-Tidy checks.
set(CLANG_TIDY_SUPPRESS -cppcoreguidelines-avoid-c-arrays*,-cppcoreguidelines-avoid-magic-numbers*,-cppcoreguidelines-pro-type-reinterpret-cast*,-cppcoreguidelines-pro-bounds-pointer-arithmetic*,-cppcoreguidelines-pro-type-vararg*,-cppcoreguidelines-pro-bounds-array-to-pointer-decay*,-cppcoreguidelines-pro-bounds-constant-array-index*,-cppcoreguidelines-owning-memory*,-modernize-avoid-c-arrays*,-modernize-use-trailing-return-type*,-readability-implicit-bool-conversion*,-readability-magic-numbers*,-readability-named-parameter*)

# Specify a semicolon-separated list containing a command line for the clang-tidy tool.
set(CMAKE_C_CLANG_TIDY clang-tidy ${CLANG_TIDY}${CLANG_TIDY_SUPPRESS})
set(CMAKE_CXX_CLANG_TIDY ${CMAKE_C_CLANG_TIDY})

# Add a library target to be built from the source files.
add_library(${CMAKE_PROJECT_NAME} SHARED)

# Options
option(OPT_DEBUG "Build with debugging information and runtime checks." ON)
option(OPT_NO_RTTI "Disable RTTI support." ON)
option(OPT_NO_EXCEPTIONS "Disable exception handling table generation." ON)

# Linking options
option(OPT_LINK_C "Linking with libc library." ON)
option(OPT_LINK_M "Linking with libm library." ON)
option(OPT_LINK_DL "Linking with libdl library." ON)
option(OPT_LINK_LIBGCC "Static linking with libgcc library." ON)
option(OPT_LINK_LIBSTDC "Static linking with libstdc++ library." ON)

# Set policies
cmake_policy(SET CMP0018 NEW)
cmake_policy(SET CMP0063 NEW)
cmake_policy(SET CMP0065 NEW)
cmake_policy(SET CMP0069 NEW)
cmake_policy(SET CMP0077 NEW)
cmake_policy(SET CMP0092 NEW)

# Settings
set(CMAKE_RULE_MESSAGES ON)
set(CMAKE_VERBOSE_MAKEFILE OFF)

include(cmake/Tools.cmake)
set_build_type()
set_output_binary_name()
set_default_parallel_jobs()
set_binary_output_directory("bin")


#-------------------------------------------------------------------------------------------
# Add include directories.
#-------------------------------------------------------------------------------------------

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    "sdk/cssdk/include"
    "metamod/include"
)


#-------------------------------------------------------------------------------------------
# Add source files.
#-------------------------------------------------------------------------------------------

find_source_files("metamod/src" sources_list)
target_sources(${CMAKE_PROJECT_NAME} PRIVATE ${sources_list})


#-------------------------------------------------------------------------------------------
# Set definitions.
#-------------------------------------------------------------------------------------------

include(cmake/Definitions.cmake)
target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE
    REHLDS_FIXES
    REGAMEDLL_FIXES
    REGAMEDLL_API
)


#-------------------------------------------------------------------------------------------
# Set compiler flags.
#-------------------------------------------------------------------------------------------

target_compile_features(${CMAKE_PROJECT_NAME} PUBLIC cxx_std_17)

include(CheckIPOSupported)
check_ipo_supported() # Fatal error if IPO is not supported.

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES
    POSITION_INDEPENDENT_CODE ON
    VISIBILITY_INLINES_HIDDEN ON
    CXX_VISIBILITY_PRESET hidden
    INTERPROCEDURAL_OPTIMIZATION_DEBUG OFF
    INTERPROCEDURAL_OPTIMIZATION_RELEASE ON
    INTERPROCEDURAL_OPTIMIZATION_MINSIZEREL ON
    INTERPROCEDURAL_OPTIMIZATION_RELWITHDEBINFO OFF
)

if (${CMAKE_CXX_COMPILER_ID} MATCHES "GNU")
    include(cmake/GCC.cmake)
elseif(${CMAKE_CXX_COMPILER_ID} MATCHES "Intel")
    include(cmake/Intel.cmake)
elseif(${CMAKE_CXX_COMPILER_ID} MATCHES "Clang")
    include(cmake/Clang.cmake)
endif()

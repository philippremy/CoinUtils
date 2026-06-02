# =============================================================================
# Coin-Or Coin Utilities
# EPL-2.0
# =============================================================================

# 3.23 for File Sets in CMake
cmake_minimum_required(VERSION 3.23)

project(CoinUtils
  LANGUAGES C CXX
  VERSION 3.11.13
)

# =============================================================================
# Detect if this is a subbuild within another project
# =============================================================================
if(${CMAKE_CURRENT_SOURCE_DIR} STREQUAL ${CMAKE_SOURCE_DIR})
  set(COINUTILS_IS_TOPLEVEL ON)
else()
  set(COINUTILS_IS_TOPLEVEL OFF)
endif()

# =============================================================================
# Language settings
# =============================================================================
set(COINUTILS_CPLUSPLUS11 NO)
foreach(CXX_FEATURE IN LISTS CMAKE_CXX_COMPILE_FEATURES)
  if(${CXX_FEATURE} STREQUAL cxx_std_11)
    set(COINUTILS_CPLUSPLUS11 YES)
    break()
  endif()
endforeach()
if(COINUTILS_CPLUSPLUS11)
  set(CMAKE_CXX_STANDARD 11)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# =============================================================================
# Project configuration
# =============================================================================
option(BUILD_SHARED_LIBS "Whether to build shared or static libraries" ON)
option(COINUTILS_ENABLE_INSTALL "Whether the project should create install rules" ${COINUTILS_IS_TOPLEVEL})
option(COINUTILS_ENABLE_BIGINDEX "Use a long integer type for CoinBigIndex" OFF)
option(COINUTILS_ENABLE_THREADS "Enables compilation of thread aware CoinUtils (mempool so far)" OFF)
option(COINUTILS_MEMPOOL_OVERRIDE_NEW "Redirects global new/delete to CoinUtils mempool" OFF)
option(COINUTILS_USE_ZLIB "Compile with compression library zlib" ON)
option(COINUTILS_USE_BZLIB "Compile with compression library bzlib" ON)
option(COINUTILS_USE_READLINE "Compile with library readline" ON)
option(COINUTILS_USE_LAPACK "Compile with LAPACK" ON)
option(COINUTILS_USE_GLPK "Compile with Glpk" ON)

set(COINUTILS_MEMPOOL_MAXPOOLED NO CACHE STRING "Specify the size in bytes of the largest block that will be allocated from the CoinUtils mempool. If negative or NO then the memory pool is disabled completely.")
set(COINUTILS_CHECK_LEVEL 0 CACHE STRING "Specify the sanity check level")
set(COINUTILS_VERBOSITY 0 CACHE STRING "Specify the debug verbosity level")
set(COINUTILS_LAPACK_FUNC "name ## _" CACHE STRING "Specifies the macro for calling Fortran code (don't edit unless using MSVC AND Intel Fortran compiler on Windows!)")

# Add CMake module path
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

# =============================================================================
# Find packages
# =============================================================================
if(COINUTILS_ENABLE_THREADS)
  set(COINUTILS_PTHREADS OFF)
  find_package(Threads)
  if(Threads_FOUND)
    set(COINUTILS_PTHREADS ON)
  endif()
endif()

if(COINUTILS_USE_ZLIB)
  set(COINUTILS_HAS_ZLIB OFF)
  find_package(ZLIB)
  if(ZLIB_FOUND)
    set(COINUTILS_HAS_ZLIB ON)
    set(HAVE_LIBZ ON)
  endif()
endif()

if(COINUTILS_USE_BZLIB)
  set(COINUTILS_HAS_BZLIB OFF)
  find_package(BZip2)
  if(BZip2_FOUND)
    set(COINUTILS_HAS_BZLIB ON)
    set(HAVE_LIBBZ2 ON)
  endif()
endif()

if(COINUTILS_USE_READLINE)
  set(COINUTILS_HAS_READLINE OFF)
  find_package(Readline)
  if(Readline_FOUND)
    set(COINUTILS_HAS_READLINE ON)
    set(HAVE_LIBREADLINE ON)
  endif()
endif()

if(COINUTILS_USE_LAPACK)
  set(COINUTILS_HAS_LAPACK OFF)
  find_package(LAPACK)
  if(LAPACK_FOUND)
    set(COINUTILS_HAS_LAPACK ON)
  endif()
endif()

if(COINUTILS_USE_GLPK)
  set(COINUTILS_HAS_GLPK OFF)
  find_package(GLPK)
  if(GLPK_FOUND)
    set(COINUTILS_HAS_GLPK ON)
  endif()
endif()

# =============================================================================
# Perform compiler checks
# =============================================================================
include(CheckIncludeFile)
include(CheckIncludeFileCXX)
include(CheckCSourceCompiles)
include(CheckCXXSourceCompiles)
include(CheckTypeSize)

# Check headers
check_include_file("float.h" COINUTILS_HAS_FLOAT_H)
check_include_file("ieeefp.h" COINUTILS_HAS_IEEEFP_H)
check_include_file("math.h" COINUTILS_HAS_MATH_H)
check_include_file("stdarg.h" COINUTILS_HAS_STDARG_H)
check_include_file("stdint.h" COINUTILS_HAS_STDINT_H)
check_include_file("stdio.h" COINUTILS_HAS_STDIO_H)
check_include_file("stdlib.h" COINUTILS_HAS_STDLIB_H)
check_include_file("string.h" COINUTILS_HAS_STRING_H)
check_include_file("inttypes.h" COINUTILS_HAS_INTTYPES_H)
check_include_file("strings.h" COINUTILS_HAS_STRINGS_H)
check_include_file("sys/stat.h" COINUTILS_HAS_SYS_STAT_H)
check_include_file("sys/types.h" COINUTILS_HAS_SYS_TYPES_H)
check_include_file("unistd.h" COINUTILS_HAS_UNISTD_H)
check_include_file("dlfcn.h" COINUTILS_HAS_DLFCN_H)
check_include_file("windows.h" COINUTILS_HAS_WINDOWS_H)
check_include_file("endian.h" COINUTILS_HAS_ENDIAN_H)
check_include_file("execinfo.h" COINUTILS_HAS_EXECINFO_H)
check_include_file_cxx("cstdint" COINUTILS_HAS_CSTDINT_H)
check_include_file_cxx("cmath" COINUTILS_HAS_CMATH_H)
check_include_file_cxx("cfloat" COINUTILS_HAS_CFLOAT_H)
check_include_file_cxx("cieeefp" COINUTILS_HAS_CIEEEFP_H)

# Check symbols
check_cxx_source_compiles("
#include <cmath>
int main() {
    return (int)(int64_t)0;
}
" COINUTILS_HAS_INT64_T)
check_cxx_source_compiles("
#include <cmath>
int main() {
    return (int)(uint64_t)0;
}
" COINUTILS_HAS_UINT64_T)
check_cxx_source_compiles("
#include <cmath>
int main() {
    return (int)(intptr_t)0;
}
" COINUTILS_HAS_INTPTR_T)
check_cxx_source_compiles("
#include <cmath>
int main() {
    return (int)0;
}
" COINUTILS_HAS_INT)
check_cxx_source_compiles("
#include <cmath>
int main() {
    return (int)std::isfinite(1.0);
}
" COINUTILS_HAS_STD_ISFINITE)
check_cxx_source_compiles("
#include <cmath>
int main() {
    return (int)std::isnan(1.0);
}
" COINUTILS_HAS_STD_ISNAN)

# Check STDC_HEADERS per autoconf
# See https://www.gnu.org/software/autoconf/manual/autoconf-2.67/html_node/Particular-Headers.html
if(COINUTILS_HAS_STDLIB_H
   AND COINUTILS_HAS_STDARG_H
   AND COINUTILS_HAS_STRING_H
   AND COINUTILS_HAS_FLOAT_H)

  # Check for memchr
  check_c_source_compiles("
  #include <string.h>
  #include <stdlib.h>
  int main() {
      void* a = memchr(NULL, 0, 0);
      return 0;
  }
  " COINUTILS_HAS_MEMCHR)

  # Check for malloc and free
  check_c_source_compiles("
  #include <stdlib.h>
  int main() {
      void* a = malloc(64);
      if(a)
        free(a);
      return 0;
  }
  " COINUTILS_HAS_MALLOC_FREE)

  if(COINUTILS_HAS_MEMCHR
     AND COINUTILS_HAS_MALLOC_FREE)

    set(STDC_HEADERS ON)

  endif()

endif()

# Check if the compiler supports __attribute__((__visibility__("default")))
check_cxx_source_compiles("
__attribute__((__visibility__(\"default\"))) void test_visibility() {}
int main() { return 0; }
" COINUTILS_SUPPORTS_VISIBILITY_ATTRIBUTE)

# Check type sizes
check_type_size("int"
  COINUTILS_SIZEOF_INT
  LANGUAGE CXX
)
check_type_size("int*"
  COINUTILS_SIZEOF_INT_P
  LANGUAGE CXX
)
check_type_size("long"
  COINUTILS_SIZEOF_LONG
  LANGUAGE CXX
)
check_type_size("long long"
  COINUTILS_SIZEOF_LONG_LONG
  LANGUAGE CXX
)

if(COINUTILS_HAS_CFLOAT_H)
  set(HAVE_CFLOAT ON)
endif()

if(COINUTILS_HAS_CIEEEFP_H)
  set(HAVE_CIEEEFP ON)
endif()

if(COINUTILS_HAS_CMATH_H)
  set(HAVE_CMATH ON)
endif()

if(COINUTILS_HAS_CSTDINT_H)
  set(HAVE_CSTDINT ON)
endif()

if(COINUTILS_HAS_DLFCN_H)
  set(HAVE_DLFCN_H ON)
endif()

if(COINUTILS_HAS_ENDIAN_H)
  set(HAVE_ENDIAN_H ON)
endif()

if(COINUTILS_HAS_EXECINFO_H)
  set(HAVE_EXECINFO_H ON)
endif()

if(COINUTILS_HAS_FLOAT_H)
  set(HAVE_FLOAT_H ON)
endif()

if(COINUTILS_HAS_IEEEFP_H)
  set(HAVE_IEEEFP_H ON)
endif()

if(COINUTILS_HAS_INTTYPES_H)
  set(HAVE_INTTYPES_H ON)
endif()

if(COINUTILS_HAS_MATH_H)
  set(HAVE_MATH_H ON)
endif()

if(COINUTILS_HAS_STDINT_H)
  set(HAVE_STDINT_H ON)
endif()

if(COINUTILS_HAS_STDLIB_H)
  set(HAVE_STDLIB_H ON)
endif()

if(COINUTILS_HAS_STDIO_H)
  set(HAVE_STDIO_H ON)
endif()

if(COINUTILS_HAS_STRINGS_H)
  set(HAVE_STRINGS_H ON)
endif()

if(COINUTILS_HAS_STRING_H)
  set(HAVE_STRING_H ON)
endif()

if(COINUTILS_HAS_SYS_STAT_H)
  set(HAVE_SYS_STAT_H ON)
endif()

if(COINUTILS_HAS_SYS_TYPES_H)
  set(HAVE_SYS_TYPES_H ON)
endif()

if(COINUTILS_HAS_UNISTD_H)
  set(HAVE_UNISTD_H ON)
endif()

if(COINUTILS_HAS_WINDOWS_H)
  set(HAVE_WINDOWS_H ON)
endif()

if(COINUTILS_HAS_CSTDINT_H)
  set(COINUTILS_HAS_CSTDINT ON)
endif()

if(COINUTILS_HAS_INT64_T)
  set(COINUTILS_INT64_T "int64_t")
endif()

if(COINUTILS_HAS_UINT64_T)
  set(COINUTILS_UINT64_T "uint64_t")
endif()

if(COINUTILS_HAS_INTPTR_T)
  set(COINUTILS_INTPTR_T "intptr_t")
endif()

if(COINUTILS_HAS_STD_ISFINITE)
  set(COINUTILS_C_FINITE "std::isfinite")
endif()

if(COINUTILS_HAS_STD_ISNAN)
  set(COINUTILS_C_ISNAN "std::isnan")
endif()

set(SIZEOF_INT ${COINUTILS_SIZEOF_INT})
set(SIZEOF_INT_P ${COINUTILS_SIZEOF_INT_P})
set(SIZEOF_LONG ${COINUTILS_SIZEOF_LONG})
set(SIZEOF_LONG_LONG ${COINUTILS_SIZEOF_LONG_LONG})

# =============================================================================
# Remaining option evaluation
# =============================================================================
if(COINUTILS_ENABLE_BIGINDEX)
  set(COINUTILS_BIGINDEX_T "long long")
  set(COINUTILS_BIGINDEX_IS_INT NO)
else()
  set(COINUTILS_BIGINDEX_T "int")
  set(COINUTILS_BIGINDEX_IS_INT YES)
endif()

set(COINUTILS_CHECKLEVEL ${COINUTILS_CHECK_LEVEL})

# =============================================================================
# Project information
# =============================================================================
set(COINUTILS_VERSION "${PROJECT_VERSION}")
set(COINUTILS_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(COINUTILS_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(COINUTILS_VERSION_RELEASE "${PROJECT_VERSION_PATCH}")
set(PACKAGE_BUGREPORT "https://github.com/coin-or/CoinUtils/issues/new")
set(PACKAGE_NAME "${PROJECT_NAME}")
set(PACKAGE_STRING "${PROJECT_NAME} ${PROJECT_VERSION}")
set(PACKAGE_TARNAME "coin-or-coinutils")
set(PACKAGE_URL "https://github.com/coin-or/CoinUtils")
set(PACKAGE_VERSION "${PROJECT_VERSION}")

# =============================================================================
# Symbol visibility
# =============================================================================
if(MSVC)
  set(COINUTILSLIB_EXPORT "__declspec(dllexport)")
  set(CMAKE_CXX_VISIBILITY_PRESET hidden)
elseif(COINUTILS_SUPPORTS_VISIBILITY_ATTRIBUTE)
  set(COINUTILSLIB_EXPORT "__attribute__((__visibility__(\"default\")))")
  set(CMAKE_CXX_VISIBILITY_PRESET hidden)
endif()

# =============================================================================
# Configure config.h and config_coinutils.h for building
# =============================================================================
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/config.h.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/config.h
)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/config_coinutils.h.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/config_coinutils.h
)

# =============================================================================
# Configure CoinUtilsConfig.h for installation
# =============================================================================
if(MSVC)
  set(COINUTILSLIB_EXPORT "__declspec(dllimport)")
else()
  unset(COINUTILSLIB_EXPORT)
endif()
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/config_coinutils.h.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/install/CoinUtilsConfig.h
)

# =============================================================================
# Add the source subdirectory
# =============================================================================
add_subdirectory(src)

# =============================================================================
# Installation if required/requested
# =============================================================================
if(COINUTILS_ENABLE_INSTALL)

  # GNU Installation directories
  include(GNUInstallDirs)

  install(TARGETS CoinUtils
    EXPORT ${PROJECT_NAME}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    FILE_SET public_headers_src DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/coinutils/coin
    FILE_SET public_headers_generated DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/coinutils/coin
  )

  # Create coinutils-config.cmake file
  include(CMakePackageConfigHelpers)
  configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/coinutils-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/coinutils-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
  )

   # Create coinutils-config-version.cmake file
  write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/coinutils-config-version.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
  )

  # Create coinutils.pc file
  configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/coinutils.pc.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/coinutils.pc
    @ONLY
  )

  # Install coinutils-config.cmake
  # Install coinutils-config-version.cmake
  install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/coinutils-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/coinutils-config-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
  )

  # Install coinutils-config.cmake
  install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/coinutils.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
  )

  # Install target files
  install(EXPORT ${PROJECT_NAME}
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
    NAMESPACE ${PROJECT_NAME}::
  )

  # Write a CPS file if CMake >= 4.3
  if(CMAKE_VERSION VERSION_GREATER_EQUAL 4.3)

    install(PACKAGE_INFO ${PROJECT_NAME}
      EXPORT ${PROJECT_NAME}
      PROJECT ${PROJECT_NAME}
      VERSION ${PROJECT_VERSION}
      COMPAT_VERSION ${PROJECT_VERSION_MAJOR}.0.0
      VERSION_SCHEMA simple
      DEFAULT_TARGETS CoinUtils
      LICENSE EPL-2.0
      DEFAULT_LICENSE EPL-2.0
      DESCRIPTION "COIN-OR Utilities: CoinUtils is an open-source collection of classes and helper functions that are generally useful to multiple COIN-OR projects."
      HOMEPAGE_URL "https://github.com/coin-or/CoinUtils"
    )

  endif()

endif()

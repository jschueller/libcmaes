cmake_minimum_required (VERSION 2.8)

if (NOT DEFINED CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type")
endif ()

project (libcmaes)

list (APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)


option (BUILD_SHARED_LIBS "Build LIBCMAES as a shared library" ON)
option (BUILD_PYTHON "build python bindings" ON)
option (USE_OPENMP "Use OpenMP for multithreading" ON)

# Offer the user the choice of overriding the installation directories
set (INSTALL_LIB_DIR     lib${LIB_SUFFIX} CACHE PATH "Installation directory for libraries")
set (INSTALL_BIN_DIR     bin              CACHE PATH "Installation directory for executables")
set (INSTALL_INCLUDE_DIR include          CACHE PATH "Installation directory for header files")
set (INSTALL_DATA_DIR    share/nlopt       CACHE PATH "Installation directory for data files")
set (INSTALL_CMAKE_DIR   ${INSTALL_LIB_DIR}/cmake/nlopt CACHE PATH "Installation directory for cmake config files")

# Make relative paths absolute (needed later on)
foreach (p LIB BIN INCLUDE DATA CMAKE)
  set (var INSTALL_${p}_DIR)
  set (RELATIVE_INSTALL_${p}_DIR ${INSTALL_${p}_DIR})
  if (NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif ()
endforeach ()


set (CMAKE_INSTALL_RPATH ${INSTALL_LIB_DIR})
set (CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
if(POLICY CMP0042)
  # Set MACOSX_RPATH to ON
  cmake_policy(SET CMP0042 NEW)
endif()

include (CheckIncludeFile)
# include (CheckFunctionExists)
# include (CheckTypeSize)
# include (CheckCCompilerFlag)
# 
# #==============================================================================
# # COMPILATION CHECKINGS and CONFIGURATION GENERATION
# #==============================================================================
check_include_file (dlfcn.h HAVE_DLFCN_H)
# check_include_file (getopt.h HAVE_GETOPT_H)
# check_include_file (unistd.h HAVE_UNISTD_H)
check_include_file (string.h HAVE_STRING_H)
check_include_file (strings.h HAVE_STRINGS_H)
check_include_file (inttypes.h HAVE_INTTYPES_H)
check_include_file (memory.h HAVE_MEMORY_H)
check_include_file (stdlib.h HAVE_STDLIB_H)
check_include_file (stdint.h HAVE_STDINT_H)
# check_include_file (time.h HAVE_TIME_H)
check_include_file (sys/types.h HAVE_SYS_TYPES_H)
check_include_file (sys/stat.h HAVE_SYS_STAT_H)
# check_include_file (sys/time.h HAVE_SYS_TIME_H)
# # if (HAVE_TIME_H AND HAVE_SYS_TIME_H)
# #   set (TIME_WITH_SYS_TIME TRUE)
# # endif ()
# check_function_exists (getpid HAVE_GETPID)
# check_function_exists (syscall HAVE_GETTID_SYSCALL)
# check_function_exists (isinf HAVE_ISINF)
# check_function_exists (isnan HAVE_ISNAN)
# check_function_exists (gettimeofday HAVE_GETTIMEOFDAY)
# check_function_exists (qsort_r HAVE_QSORT_R)
# check_function_exists (time HAVE_TIME)
# check_function_exists (copysign HAVE_COPYSIGN)

find_package (Eigen3 NO_MODULE REQUIRED)
include (${EIGEN3_USE_FILE})

if (USE_OPENMP)
  find_package (OpenMP)
  if (OPENMP_FOUND)
    set (CMAKE_CXX_FLAGS "${OpenMP_CXX_FLAGS} ${CMAKE_CXX_FLAGS}")
  endif ()
endif ()

if (BUILD_PYTHON)
  find_package (PythonInterp)
  find_package (PythonLibs)
  find_package (NumPy)

  if (NOT DEFINED PYTHON_SITE_DIR AND PythonInterp_FOUND)
    execute_process (COMMAND ${PYTHON_EXECUTABLE} -c "from distutils import sysconfig; print(sysconfig.get_python_lib(plat_specific=True, prefix='${CMAKE_INSTALL_PREFIX}'))"
                         OUTPUT_VARIABLE _ABS_PYTHON_SITE_DIR
                         RESULT_VARIABLE _exit_code
                         OUTPUT_STRIP_TRAILING_WHITESPACE )

    if (NOT ${_exit_code})
      get_filename_component (_ABS_PYTHON_SITE_DIR ${_ABS_PYTHON_SITE_DIR} ABSOLUTE)
      file (RELATIVE_PATH _REL_PYTHON_SITE_DIR ${CMAKE_INSTALL_PREFIX} ${_ABS_PYTHON_SITE_DIR})
      set (PYTHON_SITE_DIR ${_REL_PYTHON_SITE_DIR})
    else ()
      message (SEND_ERROR "Could not run ${PYTHON_EXECUTABLE}")
    endif ()
  endif ()


  
  set (BOOST_PYTHON_COMPONENT_NAMES python${PYTHON_VERSION_MAJOR} python)
  foreach (NAME ${BOOST_PYTHON_COMPONENT_NAMES})
    if (NOT Boost_FOUND)
      message (STATUS "Looking for component ${NAME}")
      find_package(Boost QUIET COMPONENTS ${NAME})
    endif ()
  endforeach ()
  set (HAVE_BOOST_PYTHON ${Boost_FOUND})
endif ()

set (LIBCMAES_MAJOR_VERSION "0")
set (LIBCMAES_MINOR_VERSION "9")
set (LIBCMAES_BUGFIX_VERSION "5")
set (LIBCMAES_VERSION_STRING ${LIBCMAES_MAJOR_VERSION}.${LIBCMAES_MINOR_VERSION}.${LIBCMAES_BUGFIX_VERSION})


configure_file (libcmaes_config.h.in libcmaes_config.h)
include_directories (${CMAKE_CURRENT_BINARY_DIR})

enable_testing ()

add_subdirectory (src)

if (Boost_FOUND AND NUMPY_FOUND)
  add_subdirectory (python)
endif ()

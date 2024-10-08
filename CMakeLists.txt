cmake_minimum_required(VERSION 2.8)
project(ParMETIS)

set(GKLIB_PATH ${CMAKE_CURRENT_SOURCE_DIR}/metis/GKlib)
set(METIS_PATH ${CMAKE_CURRENT_SOURCE_DIR}/metis)
set(SHARED ${BUILD_SHARED_LIBS})

# Prepare libraries.
if(SHARED)
  set(ParMETIS_LIBRARY_TYPE SHARED)
  # CMake RPATH "always full" configuration, see:
  # https://cmake.org/Wiki/CMake_RPATH_handling#Always_full_RPATH
  # use, i.e. don't skip the full RPATH for the build tree
  set(CMAKE_SKIP_BUILD_RPATH False)
  # when building, don't use the install RPATH already
  # (but later on when installing)
  set(CMAKE_BUILD_WITH_INSTALL_RPATH False)
  # the RPATH to be used when installing, but only if it's not a system directory
  list(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES
       "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
  if("${isSystemDir}" STREQUAL "-1")
    set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
  endif()
  # add the automatically determined parts of the RPATH
  # which point to directories outside the build tree to the install RPATH
  set(CMAKE_INSTALL_RPATH_USE_LINK_PATH True)
else()
  set(ParMETIS_LIBRARY_TYPE STATIC)
endif()

include(${GKLIB_PATH}/GKlibSystem.cmake)

find_package(MPI REQUIRED)
if(UNIX)
    # set(CMAKE_CXX_COMPILER mpicxx CACHE FILEPATH "compiler with your MPI wrapping")
    # set(CMAKE_FIND_LIBRARY_SUFFIXES ".a") # ! using static libs
elseif(MSVC OR WIN32 OR MINGW)
    # nothing
    
    if(MPI_CXX_FOUND)
        set (CMAKE_EXE_LINKER_FLAGS "${MPI_C_LINK_FLAGS} ${CMAKE_EXE_LINKER_FLAGS}")
    else()
        message((FATAL_ERROR "MPI NOT FOUND"))
    endif()
else()
    message(FATAL_ERROR "NOT YET IMPLEMENTED HERE")
endif()
# List of paths that the compiler will search for header files.
# i.e., the -I equivalent
include_directories(include)
include_directories(${MPI_INCLUDE_PATH})
include_directories(${GKLIB_PATH})
include_directories(${METIS_PATH}/include)

# List of directories that cmake will look for CMakeLists.txt
add_subdirectory(metis)
add_subdirectory(include)
add_subdirectory(libparmetis)
add_subdirectory(programs)

# This is for testing during development and is not being distributed
#add_subdirectory(test)

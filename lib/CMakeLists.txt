cmake_minimum_required(VERSION 3.7.0)
include(CheckLibraryExists)

# LibXml2 Dependency
find_package(PkgConfig)
#pkg_check_modules(LIBXML2 libxml-2.0 IMPORTED_TARGET)

include_directories(${CMAKE_OSX_SYSROOT}/usr/include/libxml2)

if (NOT TARGET LLVMCore)
  # LLVM Dependency
  message(STATUS "LLVM Hint: ${LLVM_HINT}")
  find_package(LLVM REQUIRED CONFIG HINTS ${LLVM_HINT})
  message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")
endif()
include_directories(${LLVM_INCLUDE_DIRS})

if ("LLVM" IN_LIST LLVM_AVAILABLE_LIBS)
  set (EBC_LLVM_LIBRARIES
    LLVM
  )
else()
  set (EBC_LLVM_LIBRARIES
    LLVMSupport
    LLVMObject
  )
endif()

# LLVM compiler flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-uninitialized -fno-rtti -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS")

# Xar
set(BUILD_WITH_XAR ON CACHE BOOL "Build with xar support")
if(BUILD_WITH_XAR)
  find_library(XAR_LIBRARY_NAME NAMES xar)
  if(XAR_LIBRARY_NAME)
    set(HAVE_LIBXAR true)
    set(XAR_LIBRARY ${XAR_LIBRARY_NAME})
  else ()
    check_library_exists(xar xar_open "" HAVE_LIBXAR)
    if(HAVE_LIBXAR)
      set(XAR_LIBRARY xar)
    endif()
  endif()
endif()

if(HAVE_LIBXAR)
  message(STATUS "Building with libxar: " ${XAR_LIBRARY})
else()
  message(STATUS "Building without libxar")
endif()

# Configure header
configure_file (
  "${PROJECT_SOURCE_DIR}/lib/include/ebc/Config.h.in"
  "${PROJECT_BINARY_DIR}/ebc/Config.h")

# LibEBC
set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(HEADER_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

file(GLOB_RECURSE SOURCE_FILES ${SOURCE_DIR}/*.cpp)

add_library(ebc ${SOURCE_FILES})
target_link_libraries(ebc
  ${UUID_LIBRARY}
  ${ICONV_LIBRARY}
  ${XAR_LIBRARY}
  ${EBC_LLVM_LIBRARIES}
  xml2
  ${LLVM_LINK_LIBRARIES}
  ${LIBXML2_STATIC_LDFLAGS})

target_include_directories(ebc PRIVATE  ${PROJECT_BINARY_DIR})
target_include_directories(ebc PUBLIC ${HEADER_DIR})

# Use C++14
set_property(TARGET ebc PROPERTY CXX_STANDARD 14)

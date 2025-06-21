cmake_minimum_required(VERSION 3.21)

project(wex-db VERSION 1.0)

find_package(ODBC REQUIRED)
find_package(wex 25.10 QUIET)

include_directories(${wex_INCLUDE_DIR} ${ODBC_INCLUDE_DIR})
link_directories(${wex_LIB_DIR})

file(GLOB SRCS "*.cpp")

if(wexBUILD_TESTS)
  enable_testing()
endif()

if(wexBUILD_TESTS)
  add_definitions(-DwexBUILD_TESTS)

  add_subdirectory(test)
endif()

if(wexBUILD_SHARED)
  set(BUILD_SHARED_LIBS ON)
  add_definitions(-DBOOST_LOG_DYN_LINK)

  if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    set(
      CMAKE_SHARED_LIBRARY_CREATE_CXX_FLAGS
      "${CMAKE_SHARED_LIBRARY_CREATE_CXX_FLAGS} -undefined dynamic_lookup"
    )
  endif()
endif()

add_library(${PROJECT_NAME} ${SRCS})

set_target_properties(
  ${PROJECT_NAME}
  PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR}
)

# see wex common.cmake, should be shared
if(MSVC)
  file(GLOB_RECURSE wex_own_LIBRARIES ${CMAKE_BINARY_DIR}/*.lib)
else()
  if(wexBUILD_SHARED)
    if(APPLE)
      file(
        GLOB_RECURSE wex_own_LIBRARIES
        ${CMAKE_BINARY_DIR}/*.dylib
        ${CMAKE_BINARY_DIR}/*.a
      )
    else()
      file(
        GLOB_RECURSE wex_own_LIBRARIES
        ${CMAKE_BINARY_DIR}/*.so*
        ${CMAKE_BINARY_DIR}/*.a
      )
    endif()
  else()
    file(GLOB_RECURSE wex_own_LIBRARIES ${CMAKE_BINARY_DIR}/*.a)
  endif()
endif()

install(FILES ${wex_own_LIBRARIES} DESTINATION "lib")

if(ODBC_FOUND)
  install(FILES ${CMAKE_SOURCE_DIR}/odbc.h DESTINATION ${wex_INCLUDE_DIR})
  install(FILES ${CMAKE_SOURCE_DIR}/otlv4.h DESTINATION "include")
endif()

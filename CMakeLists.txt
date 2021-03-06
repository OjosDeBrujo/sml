# Copyright (c) 2014 Kartik Kumar (me@kartikkumar.com)
# Distributed under the MIT License.
# See accompanying file LICENSE.md or copy at http://opensource.org/licenses/MIT

cmake_minimum_required(VERSION 2.6)

# Change this section to correspond with your project info.
project(SML C CXX)
set(MYPROJ_DESC                              "A simple maths library")
set(MYPROJ_UPPER                             "SML")
#set(MYPROJ_EXE                               "sml_main")
#set(MYPROJ_LIB                               "sml")
set(MYPROJ_TEST                              "test_sml")
set(MYPROJ_TEST_COVERAGE                     "test_coverage")
set(${CMAKE_PROJECT_NAME}_VENDOR_NAME        "Kartik Kumar")
set(${CMAKE_PROJECT_NAME}_VENDOR_CONTACT     "me@kartikkumar.com")

set(${CMAKE_PROJECT_NAME}_VERSION_MAJOR      0)
set(${CMAKE_PROJECT_NAME}_VERSION_MINOR      1)
set(${CMAKE_PROJECT_NAME}_VERSION_PATCH      0)
set(MYPROJ_VERSION
  "${${CMAKE_PROJECT_NAME}_VERSION_MAJOR}.${${CMAKE_PROJECT_NAME}_VERSION_MINOR}.${${CMAKE_PROJECT_NAME}_VERSION_PATCH}")

set(MYPROJ_PATH                              "${CMAKE_CURRENT_SOURCE_DIR}")
set(MYEXT_PATH                               "${MYPROJ_PATH}/external")
set(MYINC_PATH                               "${MYPROJ_PATH}/include")
set(MYSRC_PATH                               "${MYPROJ_PATH}/src")
set(MYTEST_SRC_PATH                          "${MYPROJ_PATH}/tests")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}   "${MYPROJ_PATH}/cmake/Modules")

#OPTION(BUILD_MAIN                            "Build main" OFF)
OPTION(BUILD_DOCS                            "Build docs"                     OFF)
OPTION(BUILD_TESTS                           "Build tests"                    OFF)
OPTION(BUILD_TESTS_WITH_EIGEN                "Build tests with Eigen library" OFF)
OPTION(FORCE_DEPENDENCIES_BUILD              "Force build of dependencies"    OFF)
OPTION(RUN_TEST_COVERAGE                     "Run test coverage analysis"     OFF)

#set(MYPROJ_SRC
#)

#set(MYPROJ_EXE_SRC
#)

set(MYPROJ_TEST_SRC
  "${MYTEST_SRC_PATH}/testSml.cpp"
  "${MYTEST_SRC_PATH}/testBasicFunctions.cpp"
  "${MYTEST_SRC_PATH}/testConstants.cpp"
  "${MYTEST_SRC_PATH}/testLinearAlgebra.cpp"
)

# Set CMake build-type. If it not supplied by the user, the default built type is "Release".
if((NOT CMAKE_BUILD_TYPE) OR (CMAKE_BUILD_TYPE STREQUAL "Release"))
    message(STATUS "WARNING: building release version!")
    set(CMAKE_BUILD_TYPE Release)
elseif(CMAKE_BUILD_TYPE STREQUAL "Debug")
    message(STATUS "WARNING: building debug version!")
endif((NOT CMAKE_BUILD_TYPE) OR (CMAKE_BUILD_TYPE STREQUAL "Release"))

# Set platform-specific compiler flags.
if(WIN32)
  if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc /Ox /W3 /FC -D_SCL_SECURE_NO_WARNINGS")
    # We need to force MSVC to use static runtime:
    # (from http://www.cmake.org/Wiki/CMake_FAQ#Dynamic_Replace).
    foreach(flag_var
      CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
      CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
      # Find all dynamic runtime (MD) references and replace with static (MT)
      if(${flag_var} MATCHES "/MD")
        string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
      endif(${flag_var} MATCHES "/MD")
    endforeach(flag_var)
  else(MSVC)
    set(CMAKE_CXX_FLAGS
      "${CMAKE_CXX_FLAGS} -Wall -Woverloaded-virtual -Wold-style-cast -Wnon-virtual-dtor")
  endif(MSVC)
else(WIN32)
  set(CMAKE_CXX_FLAGS
    "${CMAKE_CXX_FLAGS} -Wall -Woverloaded-virtual -Wold-style-cast -Wnon-virtual-dtor")
endif(WIN32)

# Set up test coverage.
# N.B.: This only works with gcc and requires both lcov and gcov.
# Ref: http://stackoverflow.com/questions/13116488/detailed-guide-on-using-gcov-with-cmake-cdash
if(RUN_TEST_COVERAGE)
  include(CodeCoverage)

  set(COVERAGE_EXTRACT
    '${MYPROJ_PATH}/include/*'
    '${MYPROJ_PATH}/src/*'
    '${MYPROJ_PATH}/tests/*'
  )

  set(CMAKE_CXX_FLAGS "-g -O0 -fprofile-arcs -ftest-coverage")
  set(CMAKE_C_FLAGS "-g -O0 -fprofile-arcs -ftest-coverage")
  set(CMAKE_EXE_LINKER_FLAGS="-fprofile-arcs -ftest-coverage")

  setup_target_for_coverage(${MYPROJ_TEST_COVERAGE} ${MYPROJ_TEST} coverage)
endif(RUN_TEST_COVERAGE)

include_directories(AFTER "${MYINC_PATH}")

# Include script to build external library with CMake.
include(ExternalProject)

#if(NOT MYLIB_PATH)
#  set(MYLIB_PATH "${PROJECT_BINARY_DIR}/lib")
#endif(NOT MYLIB_PATH)
#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${MYLIB_PATH})
#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${MYLIB_PATH})
#add_library(${MYPROJ_LIB} ${MYPROJ_SRC})

#if(BUILD_MAIN)
#  if(NOT MYBIN_PATH)
#    set(MYBIN_PATH "${PROJECT_BINARY_DIR}/bin")
#  endif(NOT MYBIN_PATH)
#  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${MYBIN_PATH})
#  add_executable(${MYPROJ_EXE} ${MYPROJ_EXE_SRC})
#  target_link_libraries(${MYPROJ_EXE} ${MYPROJ_LIB})
#endif(BUILD_MAIN)

if(BUILD_DOCS)
  find_package(Doxygen)

  if(NOT DOXYGEN_FOUND)
    message(FATAL_ERROR
            "Doxygen is needed to build the documentation. Please make sure it is installed.")
  endif(NOT DOXYGEN_FOUND)

  configure_file(${MYPROJ_PATH}/Doxyfile.in
                 ${PROJECT_BINARY_DIR}/Doxyfile @ONLY IMMEDIATE)

  # Add a custom target to run Doxygen whenever the project is built.
  # If you do NOT want the documentation to be generated EVERY time you build the project
  # then leave out the 'ALL' keyword from the command below.
  add_custom_target(docs ALL COMMAND ${DOXYGEN_EXECUTABLE}
                    ${PROJECT_BINARY_DIR}/Doxyfile
                    SOURCES ${PROJECT_BINARY_DIR}/Doxyfile)
endif(BUILD_DOCS)

if(BUILD_TESTS)
  enable_testing()

  if(NOT MYTESTS_PATH)
    set(MYTESTS_PATH "${PROJECT_BINARY_DIR}/tests")
  endif(NOT MYTESTS_PATH)

  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${MYTESTS_PATH})

  if(NOT FORCE_DEPENDENCIES_BUILD)
    find_package(CATCH)
  endif(NOT FORCE_DEPENDENCIES_BUILD)

  if(NOT CATCH_FOUND)
    message(STATUS "Catch will be downloaded when ${CMAKE_PROJECT_NAME} is built")
    ExternalProject_Add(catch
      PREFIX ${MYEXT_PATH}/Catch
      #--Download step--------------
      URL https://github.com/kartikkumar/Catch/archive/master.zip
      TIMEOUT 30
      #--Update/Patch step----------
      UPDATE_COMMAND ""
      PATCH_COMMAND ""
      #--Configure step-------------
      CONFIGURE_COMMAND ""
      #--Build step-----------------
      BUILD_COMMAND ""
      #--Install step---------------
      INSTALL_COMMAND ""
      #--Output logging-------------
      LOG_DOWNLOAD ON
    )
    ExternalProject_Get_Property(catch source_dir)
    set(CATCH_INCLUDE_DIRS ${source_dir}/include CACHE INTERNAL "Path to include folder for Catch")
  endif(NOT CATCH_FOUND)

  if(NOT APPLE)
    include_directories(SYSTEM AFTER "${CATCH_INCLUDE_DIRS}")
  else(APPLE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -isystem \"${CATCH_INCLUDE_DIRS}\"")
  endif(NOT APPLE)

  if(BUILD_TESTS_WITH_EIGEN)
    if(NOT FORCE_DEPENDENCIES_BUILD)
      find_package(Eigen3)
    endif(NOT FORCE_DEPENDENCIES_BUILD)

    if(NOT EIGEN3_FOUND)
      message(STATUS "Eigen will be downloaded when ${CMAKE_PROJECT_NAME} is built")
      ExternalProject_Add(eigen-lib
        PREFIX ${MYEXT_PATH}/Eigen
        #--Download step--------------
        URL http://bitbucket.org/eigen/eigen/get/3.2.2.tar.gz
        TIMEOUT 30
        #--Update/Patch step----------
        UPDATE_COMMAND ""
        PATCH_COMMAND ""
        #--Configure step-------------
        CONFIGURE_COMMAND ""
        #--Build step-----------------
        BUILD_COMMAND ""
        #--Install step---------------
        INSTALL_COMMAND ""
        #--Output logging-------------
        LOG_DOWNLOAD ON
      )
      ExternalProject_Get_Property(eigen-lib source_dir)
      set(EIGEN3_INCLUDE_DIR ${source_dir} CACHE INTERNAL "Path to include folder for Eigen")
    endif(NOT EIGEN3_FOUND)

    if(NOT APPLE)
      include_directories(SYSTEM AFTER "${EIGEN3_INCLUDE_DIR}")
    else(APPLE)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -isystem \"${EIGEN3_INCLUDE_DIR}\"")
    endif(NOT APPLE)

    string(REPLACE "Algebra" "AlgebraEigen" TESTS_SRC_EIGEN "${MYPROJ_TEST_SRC}")
    add_executable(${MYPROJ_TEST}_eigen ${TESTS_SRC_EIGEN})
    #add_dependencies(${MYPROJ_TEST}_eigen)
    #target_link_libraries(${MYPROJ_TEST} ${MYPROJ_LIB})
    add_test(NAME ${MYPROJ_TEST}_eigen COMMAND "${MYTESTS_PATH}/${MYPROJ_TEST}_eigen")

    string(REPLACE "Algebra" "AlgebraEigenFixedVector" TESTS_SRC_EIGEN_FIXED "${MYPROJ_TEST_SRC}")
    add_executable(${MYPROJ_TEST}_eigen_fixed ${TESTS_SRC_EIGEN_FIXED})
    #add_dependencies(${MYPROJ_TEST}_eigen_fixed)
    #target_link_libraries(${MYPROJ_TEST} ${MYPROJ_LIB})
    add_test(NAME ${MYPROJ_TEST}_eigen_fixed
      COMMAND "${MYTESTS_PATH}/${MYPROJ_TEST}_eigen_fixed")
  endif(BUILD_TESTS_WITH_EIGEN)

  add_executable(${MYPROJ_TEST} ${MYPROJ_TEST_SRC})
  #add_dependencies(${MYPROJ_TEST})
  #target_link_libraries(${MYPROJ_TEST} ${MYPROJ_LIB})
  add_test(NAME ${MYPROJ_TEST} COMMAND "${MYTESTS_PATH}/${MYPROJ_TEST}")
endif(BUILD_TESTS)

# Install header files and library.
# Destination is set by CMAKE_INSTALL_PREFIX and defaults to usual locations, unless overridden by
# user.
install(DIRECTORY ${MYINC_PATH}/${CMAKE_PROJECT_NAME}
        DESTINATION include
        FILES_MATCHING PATTERN "*.hpp")
#install(TARGETS ${MYPROJ_LIB} DESTINATION lib)

# Set up packager.
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${CMAKE_PROJECT_NAME}")
set(CPACK_PACKAGE_VENDOR "${CMAKE_PROJECT_NAME}_VENDOR_NAME")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${MYPROJ_PATH}/README.md")
set(CPACK_RESOURCE_FILE_LICENSE "${MYPROJ_PATH}/LICENSE.md")
set(CPACK_PACKAGE_VERSION_MAJOR "${${CMAKE_PROJECT_NAME}_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${${CMAKE_PROJECT_NAME}_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${${CMAKE_PROJECT_NAME}_VERSION_PATCH}")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "${CMAKE_PROJECT_NAME}")
if(WIN32 AND NOT UNIX)
  # There is a bug in NSI that does not handle full unix paths properly. Make
  # sure there is at least one set of four (4) backlasshes.
  set(CPACK_NSIS_DISPLAY_NAME "${CPACK_PACKAGE_INSTALL_DIRECTORY}")
  set(CPACK_NSIS_CONTACT "${CMAKE_PROJECT_NAME}_VENDOR_CONTACT")
  set(CPACK_NSIS_MODIFY_PATH ON)
else(WIN32 AND NOT UNIX)
endif(WIN32 AND NOT UNIX)
include(CPack)

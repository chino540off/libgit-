CMAKE_MINIMUM_REQUIRED(VERSION 3.1)

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

PROJECT(hugtak)

INCLUDE(CheckIncludeFiles)

# For some external project macros
INCLUDE(ExternalProject)

# Download libgit2 from git and build it

EXTERNALPROJECT_ADD(libgit2
                    PREFIX /tmp/build/libgit2
                    GIT_REPOSITORY https://github.com/libgit2/libgit2.git
                    GIT_TAG "v0.25.1"
                    CONFIGURE_COMMAND ${CMAKE_COMMAND} .
                                        -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
                                        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
                    BUILD_COMMAND make
                    BUILD_IN_SOURCE 1
                    INSTALL_COMMAND make install)
SET(LIBGIT2_INCLUDE_DIRS  ${CMAKE_INSTALL_PREFIX}/include)
SET(LIBGIT2_LIBRARIES     ${CMAKE_INSTALL_PREFIX}/lib/libgit2.so)

SET(CMAKE_CXX_STANDARD 14)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)

SET(CMAKE_CXX_FLAGS         "${CMAKE_CXX_FLAGS} -W -Wall -Wextra -Werror -Wno-multichar -pedantic")
SET(CMAKE_CXX_FLAGS_DEBUG   " -O0 -ggdb -fprofile-arcs -ftest-coverage")
SET(CMAKE_CXX_FLAGS_RELEASE " -O2")

IF("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  SET(CMAKE_CXX_FLAGS       "${CMAKE_CXX_FLAGS} -stdlib=libc++")
ENDIF()

INCLUDE_DIRECTORIES(
  ${PROJECT_SOURCE_DIR}/include
  ${LIBGIT2_INCLUDE_DIRS}
)

#
# Tests
#
IF("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
  # MEMORYCHECK_COMMAND_OPTIONS needs to place prior to CTEST_MEMORYCHECK_COMMAND
  SET(MEMORYCHECK_COMMAND_OPTIONS "--leak-check=full
                                   --show-leak-kinds=definite,possible
                                   --error-exitcode=1
                                   --suppressions=${LIBGIT2_SOURCE_DIR}/libgit2_clar.supp")
  SET(CTEST_MEMORYCHECK_COMMAND valgrind)

  INCLUDE(CTest)

  ADD_SUBDIRECTORY(tests)
ENDIF()


INSTALL(DIRECTORY include/ DESTINATION include)

# Much inspiration here borrow from
# https://gitlab.com/CLIUtils/modern-cmake/tree/master/examples/extended-project
# https://cliutils.gitlab.io/modern-cmake/chapters/basics/structure.html
cmake_minimum_required(VERSION 3.18)
set (CMAKE_CXX_STANDARD 11)

# See https://stackoverflow.com/questions/37128555/getting-cmake-to-build-shared-library-for-msvc
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS 1)

project(HyperArrow VERSION 0.1)

option(ARROW_LINK_SHARED "Link to the Arrow shared library" ON)

find_package(Arrow REQUIRED)

if (NOT DEFINED CMAKE_PREFIX_PATH)
  set(CMAKE_PREFIX_PATH $ENV{TABLEAU_CMAKE_PATH} CACHE PATH "Cmake prefix path" FORCE)
endif ()
  
find_package(tableauhyperapi-cxx REQUIRED CONFIG)

add_subdirectory(src)

# Boost isn't required for a bare bones installation
# currently only for test / Python builds
if (DEFINED HYPERARROW_TESTING)
  unset(HYPERARROW_TESTING)
  find_package(Boost COMPONENTS system REQUIRED)
  enable_testing()
  include(CTest)
  add_subdirectory(tests)
endif ()

find_package(Python COMPONENTS Interpreter)

if (Python_FOUND)
  add_custom_target(python
    COMMAND ${Python_EXECUTABLE} setup.py bdist_wheel
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/python
    DEPENDS hyperarrow_writer hyperarrow_reader
    )
endif()

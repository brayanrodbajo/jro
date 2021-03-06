cmake_minimum_required(VERSION 3.0)
project(CEGO)

# We heavily use C++11, C++11 support is required
set (CMAKE_CXX_STANDARD 11)

# Add the pybind11 stuff
if (NOT CEGO_NO_PYTHON)
  add_subdirectory("${CMAKE_SOURCE_DIR}/externals/pybind11" "pybind11")
endif()

macro(add_nix_libraries target)
    # See https://stackoverflow.com/a/29871891
    # Add DL and pthreads
    FIND_PACKAGE ( Threads REQUIRED )
    find_package(Threads REQUIRED)
    if(THREADS_HAVE_PTHREAD_ARG)
      target_compile_options(${target} PRIVATE "-pthread")
    endif()
    if(CMAKE_THREAD_LIBS_INIT)
      target_link_libraries(${target} PRIVATE "${CMAKE_THREAD_LIBS_INIT}")
    endif()
endmacro()

function(attach_includes target)
  target_include_directories(${target} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/include")
  target_include_directories(${target} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/externals/Eigen")
  target_include_directories(${target} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/externals/Catch/single_include")
  target_include_directories(${target} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/externals/ThreadPool2")
  target_include_directories(${target} PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/externals/nlohmann")
  if (NOT MSVC)
    add_nix_libraries(${target})
  endif()
endfunction()

if (NOT CEGO_NO_PYTHON)
  # Build pybind11 python module
  pybind11_add_module(PyCEGO "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cxx")
  target_compile_definitions(PyCEGO PUBLIC -DPYBIND11)
  attach_includes(PyCEGO)
endif()

if (CEGO_ALL)
  set(APP_SOURCES)
  if (MSVC)
      list(APPEND APP_SOURCES "${CMAKE_SOURCE_DIR}/externals/Eigen/debug/msvc/eigen.natvis")
  endif()

  # Collect all the snippets in the src folder
  file(GLOB_RECURSE snippets "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cxx")
  message(STATUS "snippets found = ${snippets}")

  foreach (snippet ${snippets})
    get_filename_component(snippet_exe ${snippet} NAME_WE)
    add_executable(${snippet_exe} ${snippet})
    if (NOT CEGO_NO_PYTHON)
      target_link_libraries (${snippet_exe} PRIVATE pybind11::embed)
    endif()
    attach_includes(${snippet_exe})
  endforeach()

endif()
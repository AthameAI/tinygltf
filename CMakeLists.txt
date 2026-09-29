cmake_minimum_required(VERSION 3.10)

project(tinygltf_v3 C)

include(GNUInstallDirs)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED On)
set(CMAKE_C_EXTENSIONS Off)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(TINYGLTF3_BUILD_TESTS "Build and run tinygltf v3 C unit tests" ON)
option(TINYGLTF3_INSTALL "Install tinygltf v3 files during install step" ON)

set(TINYGLTF3_INSTALL_DIR ${CMAKE_INSTALL_INCLUDEDIR})

if (TINYGLTF3_BUILD_TESTS)
  enable_testing()

  function(add_tinygltf_v3_c_test target)
    add_executable(${target} ${ARGN})
    target_include_directories(${target} PRIVATE
      ${CMAKE_CURRENT_SOURCE_DIR}
      ${CMAKE_CURRENT_SOURCE_DIR}/tests
    )
    set_target_properties(${target} PROPERTIES
      C_STANDARD 11
      C_STANDARD_REQUIRED ON
      C_EXTENSIONS OFF
    )
    add_test(NAME ${target} COMMAND ${target} WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/tests)
  endfunction()

  add_tinygltf_v3_c_test(tester_v3_c tests/tester_v3_c.c tiny_gltf_v3.c)
  target_compile_definitions(tester_v3_c PRIVATE TINYGLTF3_ENABLE_FS)

  add_tinygltf_v3_c_test(tester_v3_c_v1port tests/tester_v3_c_v1port.c tiny_gltf_v3.c)
  target_compile_definitions(tester_v3_c_v1port PRIVATE TINYGLTF3_ENABLE_FS)

  add_tinygltf_v3_c_test(tester_v3_json_c tests/tester_v3_json_c.c)

  add_tinygltf_v3_c_test(tester_v3_freestanding tests/tester_v3_freestanding.c)
  if (CMAKE_C_COMPILER_ID MATCHES "Clang|GNU")
    target_compile_options(tester_v3_freestanding PRIVATE -ffreestanding)
  endif()
endif ()

if (TINYGLTF3_INSTALL)
  install(FILES
    tiny_gltf_v3.h
    tiny_gltf_v3.c
    tinygltf_json_c.h
    DESTINATION
    ${TINYGLTF3_INSTALL_DIR}
  )
endif ()

# AthameAI/gatein: a compiled library and a CMake package, so consumers can
#   find_package(tinygltf3 CONFIG REQUIRED)
#   target_link_libraries(app PRIVATE tinygltf3::tinygltf3)
# instead of compiling tiny_gltf_v3.c into each project. Static and
# position-independent, so it links into shared libraries too; the
# stdio-backed filesystem helpers are on (tg3_parse_file and friends).
option(TINYGLTF3_BUILD_LIBRARY "Build and install the tinygltf3 library and CMake package" OFF)

if (TINYGLTF3_BUILD_LIBRARY)
  add_library(tinygltf3 STATIC tiny_gltf_v3.c)
  add_library(tinygltf3::tinygltf3 ALIAS tinygltf3)
  set_target_properties(tinygltf3 PROPERTIES
    C_STANDARD 11
    C_STANDARD_REQUIRED ON
    C_EXTENSIONS OFF
    POSITION_INDEPENDENT_CODE ON
  )
  target_compile_definitions(tinygltf3 PRIVATE TINYGLTF3_ENABLE_FS)
  target_include_directories(tinygltf3 PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )
  if (UNIX)
    target_link_libraries(tinygltf3 PUBLIC m)
  endif ()

  include(CMakePackageConfigHelpers)
  install(TARGETS tinygltf3 EXPORT tinygltf3Targets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )
  install(EXPORT tinygltf3Targets
    NAMESPACE tinygltf3::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tinygltf3
  )
  file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/tinygltf3Config.cmake
    "include(\${CMAKE_CURRENT_LIST_DIR}/tinygltf3Targets.cmake)\n")
  write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/tinygltf3ConfigVersion.cmake
    VERSION 3.0.1 COMPATIBILITY SameMajorVersion)
  install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/tinygltf3Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/tinygltf3ConfigVersion.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tinygltf3
  )
endif ()

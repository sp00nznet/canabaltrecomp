cmake_minimum_required(VERSION 3.20)
project(canabaltrecomp C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# The Mach-O loader, the framework shims, the ObjC runtime and the emitter are
# title-agnostic and live in iparecomp. Only what is specific to this game
# belongs here -- right now that is contract/, and next the host bridge.
add_subdirectory(iparecomp)

add_executable(canabalt_host iparecomp/tools/ipa_host.cpp)
target_link_libraries(canabalt_host PRIVATE iparecomp)

# The lifted program, when it has been generated. It is not in the repository
# and never will be: it is machine code from a binary you supplied, so it is
# built locally and .gitignore keeps it out.
#
#   python iparecomp/tools/lifter.py Canabalt.ipa --out generated/
#
# Eight translation units because one 4 MB file is a slow compile and a worse
# incremental one; the count is arbitrary and `--shards` changes it.
file(GLOB CANABALT_LIFTED CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/generated/*.c")
if(CANABALT_LIFTED)
  add_library(canabalt_lifted STATIC ${CANABALT_LIFTED})
  target_include_directories(canabalt_lifted PUBLIC
    "${CMAKE_CURRENT_SOURCE_DIR}/generated")
  target_link_libraries(canabalt_lifted PUBLIC iparecomp)
  # Lifted code is generated, not written. Warnings about it are noise about a
  # decision the emitter made, and the place to fix one is the emitter.
  if(MSVC)
    target_compile_options(canabalt_lifted PRIVATE /W0)
  else()
    target_compile_options(canabalt_lifted PRIVATE -w)
  endif()
  target_link_libraries(canabalt_host PRIVATE canabalt_lifted)
  message(STATUS "canabaltrecomp: lifted program found -- linking it in")
else()
  message(STATUS "canabaltrecomp: no generated/ -- run the lifter to make one")
endif()

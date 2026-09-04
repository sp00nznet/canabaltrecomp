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

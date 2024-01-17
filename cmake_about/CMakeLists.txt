#Example of standart CMakeLists.txt
####FIRST YOU NEED TO CREATE build FOLDER IN THIS DIRECTORY:
#
#   mkdir -p build
#   cd build
#   cmake -DCMAKE_BUILD_TYPE=Debug ..       ###--OR--###      cmake -DCMAKE_BUILD_TYPE=Release ..
#   cmake --build .
#---(launch the "game")---
#
##########################
#---STRUCTURE OF PROJECT:
# (you)-|
#       |--->build
#       |
#       |--->src
#       |
#       |--->inc
##########################
cmake_minimum_required(VERSION 3.16)

project(RPGame)

# Disable C ++ extensions to follow the standard
if(NOT CMAKE_CXX_EXTENSIONS)
    set(CMAKE_CXX_EXTENSIONS OFF)
endif()

# Set compile flags
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")#For DEBUG
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")#For RELEASE
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wextra")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wpedantic")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wcast-align")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wcast-qual")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wconversion")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wctor-dtor-privacy")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wduplicated-branches")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wduplicated-cond")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wextra-semi")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wfloat-equal")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wlogical-op")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wnon-virtual-dtor")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wold-style-cast")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Woverloaded-virtual")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wredundant-decls")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wsign-conversion")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wenum-compare")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wsign-promo")

message(${CMAKE_CXX_FLAGS})

set(APP game)
set(SOURCE_DIR "${CMAKE_SOURCE_DIR}/src/")
set(INCLUDE_DIR "${CMAKE_SOURCE_DIR}/inc/")

message("Source directory: ${SOURCE_DIR}")
message("Include directory: ${INCLUDE_DIR}")

# Add libriries and their include directories
add_library(game_logic STATIC "${SOURCE_DIR}game_logic.cpp")
target_include_directories(game_logic PUBLIC "${INCLUDE_DIR}")

add_library(level STATIC "${SOURCE_DIR}level.cpp")
target_include_directories(level PRIVATE "${INCLUDE_DIR}")

add_library(player STATIC "${SOURCE_DIR}player.cpp")
target_include_directories(player PRIVATE "${INCLUDE_DIR}")

add_library(npc STATIC "${SOURCE_DIR}npc.cpp")
target_include_directories(npc PRIVATE "${INCLUDE_DIR}")

add_executable("${APP}" "${SOURCE_DIR}main.cpp")
target_link_libraries("${APP}"  game_logic 
                                level 
                                player
                                npc)

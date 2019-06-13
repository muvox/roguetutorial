cmake_minimum_required(VERSION 3.14) #Version of cmake
project(roguetutorial)              #Project name

set(CMAKE_CXX_STANDARD 14)  #Standard

set (CMAKE_CXX_FLAGS "-lmingw32 -static-libgcc -static-libstdc++")

set(CMAKE_MODULE_PATH "C:/Dev/CPP/CMake/Modules") #Location of custom modules, i.e. FindSDL2.cmake

set(SDL2_PATH "C:/Dev/CPP/SDL2-2.0.9/x86_64-w64-mingw32")   #Set path for SDL2 finder(i guess?)

find_package(SDL2 REQUIRED)                                 #Use the actual FindSDL2

include_directories(${SDL2_INCLUDE_DIR})                    #Include include directory(headers)
include_directories(${PROJECT_SOURCE_DIR}/include)          #Include project include directory(headers)
link_directories(${PROJECT_SOURCE_DIR}/lib)                 #Include lib folder (for .lib 's and .a 's)

file(GLOB TCODINC ${PROJECT_SOURCE_DIR}/include/*.h ${PROJECT_SOURCE_DIR}/include/*.hpp) #Point the directory of libtcod header files for variable TCODINC

set(SOURCE_FILES src/main.cpp ${TCODINC} )                  #Include TCODINC in the source files

add_executable(roguetutorial ${SOURCE_FILES} )                      #Project name, all necessary source files

target_link_libraries(${CMAKE_PROJECT_NAME} ${SDL2_LIBRARY} tcod) #Link the goddamn libraries

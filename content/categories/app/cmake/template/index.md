+++

title= "cmake 模板"
description= "cmake模板"
date= 2022-04-19T15:57:41+08:00
author= "chao"
draft= false
image= "" 
math= true
categories= [
    "app"
]

tags=  [
    " project","cmake"
]

+++

# Cmake模板

~~~cmake

cmake_minimum_required(VERSION 3.1)

project(PROJECT_XXX VERSION 0.0.0.0 )

#C/C++标准
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_C_STANDARD 11)

#设置编译器
set (CMAKE_C_COMPILER "/usr/bin/gcc")
set (CMAKE_CXX_COMPILER "/usr/bin/g++")

#lib&&bin
set(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}/bin)

#release debug
set(CMAKE_BUILD_TYPE Debug#[[Release | Debug| RelWithDebInfo |MinSizeRel]])
#add_compile_options()#等同CMAKE_CXXFLAGS_RELESE,前者可以对所有的编译器设置，后者只能是C++编译器


include_directories(
目录
)

aux_source_directory(目录 变量)

#FIND_LIBRARY(#变量 libceres.so #目录)

add_executable(${PROJECT_NAME} )
#target_link_libraries(${PROJECT_NAME} 
#/usr/local/lib/libmodbus.so)


~~~


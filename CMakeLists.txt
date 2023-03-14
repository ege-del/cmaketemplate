cmake_minimum_required(VERSION 3.20)
project(MyProject)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON) #Needed for LSPs 

# example usage of dependencies
# add_subdirectory(Vendor/json)

# example usage of dependencies
# include_directories(Vendor/json/include/)

# for simple dependencies without CMakeLists.txt
# file(GLOB VENDOR_SOURCE Vendor/simpleVendor/src/file.c
#                         Vendor/simpleVendor2/src/file.c)

file(GLOB_RECURSE SOURCE Source/*.cpp)
file(GLOB_RECURSE HEADER Source/*.hpp)
file(GLOB_RECURSE M_SOURCE Modules/*.cpp)
file(GLOB_RECURSE M_HEADER Modules/*.hpp)

# Helps visual studio (Windows) with file tabs
source_group("Modules" FILES ${M_SOURCE})
source_group("Source" FILES ${SOURCE})


set(TARGET_RUNTIME ${SOURCE} ${HEADER})

add_executable(${PROJECT_NAME}
                ${SOURCE}
                ${HEADER}
								${VENDOR_SOURCE}
                ${M_SOURCE}
                ${M_HEADER})

# json  example link
target_link_libraries(${PROJECT_NAME})

# add_definitions(-D PROJECT_SOURCE_DIR=\"${PROJECT_SOURCE_DIR}\")

target_include_directories(${PROJECT_NAME} PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/Source"
                                                  "${CMAKE_CURRENT_SOURCE_DIR}/Modules")

set_target_properties(${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE})

if(MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /std=c++17")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
endif()


add_custom_command(
    TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/Resources $<TARGET_FILE_DIR:${PROJECT_NAME}>)


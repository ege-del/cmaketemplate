cmake_minimum_required(VERSION 3.20)
project(MyProject)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON) #Needed for LSPs 

# set(ENABLE_VCPKG OFF)

# find_program(CCACHE_PROGRAM ccache)

# example usage of dependencies
# add_subdirectory(Vendor/json)

# example usage of dependencies
# include_directories(Vendor/json/include/)

# for simple dependencies without CMakeLists.txt
# file(GLOB VENDOR_SOURCE Vendor/simpleVendor/src/file.c
#                         Vendor/simpleVendor2/src/file.c)

option(PROJECT_CUSTOM_DEF "My compile definition" ON)

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

# link 'json' target_link_libraries(${PROJECT_NAME} json)
target_link_libraries(${PROJECT_NAME} )

add_definitions(-D PROJECT_SOURCE_DIR=\"${PROJECT_SOURCE_DIR}\")

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}
	CACHE PATH
	"Single Directory for all Libraries"
)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin/${CMAKE_BUILD_TYPE}
	CACHE PATH
	"Single Directory for all Executables."
)

set_target_properties(${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/Bin/${CMAKE_BUILD_TYPE})
target_include_directories(${PROJECT_NAME} PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/Source" "${CMAKE_CURRENT_SOURCE_DIR}/Modules")

message(STATUS "## Listing Options #####################################")
message(STATUS "## PROJECT_CUSTOM_DEF      ${PROJECT_CUSTOM_DEF}")
message(STATUS "########################################################")

add_custom_command( TARGET ${PROJECT_NAME} POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/Resources $<TARGET_FILE_DIR:${PROJECT_NAME}>)

# visual studio build fix for lsp
add_custom_target(move_compile_commands
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
    ${${PROJECT_NAME}_BINARY_DIR}/compile_commands.json
    ${${PROJECT_NAME}_BINARY_DIR}/../compile_commands.json
  DEPENDS ${${PROJECT_NAME}_BINARY_DIR}/compile_commands.json
)
# Add a dependency on move_compile_commands to ensure it runs before build
add_dependencies(${PROJECT_NAME} move_compile_commands)
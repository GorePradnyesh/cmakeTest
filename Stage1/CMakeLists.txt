cmake_minimum_required(VERSION 3.20)
project(Stage1)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
    add_compile_options(/W4)
else()
    add_compile_options(-Wall)
endif()

file(GLOB SRC CONFIGURE_DEPENDS 
    src/*.cpp
    src/Logging/*.cpp)

file(GLOB INC CONFIGURE_DEPENDS 
    inc/*.h 
	inc/*.hpp)

# source_group("Header Files\\Inc" FILES ${INC})

function(assign_source_group)
    foreach(_source IN ITEMS ${ARGN})
        if (IS_ABSOLUTE "${_source}")
            file(RELATIVE_PATH _source_rel "${CMAKE_CURRENT_SOURCE_DIR}" "${_source}")
        else()
            set(_source_rel "${_source}")
        endif()
        get_filename_component(_source_path "${_source_rel}" PATH)
        string(REPLACE "/" "\\" _source_path "${_source_path}")
        source_group("${_source_path}" FILES "${_source}")
    endforeach()
endfunction(assign_source_group)

assign_source_group(${SRC})
assign_source_group(${INC})

add_executable(Stage1 
    ${SRC} 
    ${INC})

target_include_directories(Stage1 PRIVATE inc)
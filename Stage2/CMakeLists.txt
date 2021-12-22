cmake_minimum_required(VERSION 3.20)
project(Stage2)

# includes 
include(FetchContent)
include(${CMAKE_CURRENT_SOURCE_DIR}/../Common/CMakeLists.txt)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
    add_compile_options(/W4)
else()
    add_compile_options(-Wall)
endif()

set(BOOST_URL "https://boostorg.jfrog.io/artifactory/main/release/1.78.0/source/boost_1_78_0.tar.bz2" CACHE STRING "Boost 1.78.0 download URL")
set(BOOST_URL_SHA256 "8681f175d4bdb26c52222665793eef08490d7758529330f98d3b29dd0735bccc" CACHE STRING "Boost download URL SHA256 checksum")

FetchContent_Declare(
    Boost
    URL ${BOOST_URL}
    URL_HASH SHA256=${BOOST_URL_SHA256}
)
FetchContent_GetProperties(Boost)
if(NOT EXISTS ${PROJECT_SOURCE_DIR}/external/boost/lib)
    message(STATUS "Fetching Boost")
    FetchContent_Populate(Boost)
    message(STATUS "Fetching Boost - done")

    # set options here if necessary
    set(BOOTSTRAP_CMD ${boost_SOURCE_DIR}/bootstrap.sh)
    set(PREFIX --prefix=${CMAKE_CURRENT_SOURCE_DIR}/external/boost)
    message(VERBOSE "prefix : " ${BOOTSTRAP_CMD})
    message(VERBOSE "Executing bootstrap cmd: ${BOOTSTRAP_CMD} ${PREFIX}")
    execute_process(COMMAND ${BOOTSTRAP_CMD} ${PREFIX} WORKING_DIRECTORY ${boost_SOURCE_DIR} COMMAND_ECHO STDOUT RESULT_VARIABLE ret)
    
    # TODO Move to helper function
    if(NOT ret EQUAL 0)
        message( FATAL_ERROR "Bootstrap failed. Bad exit status ${ret}")
    else()
        message(VERBOSE "Bootstrap completed successfully")
    endif()


    set(B2_CMD ${boost_SOURCE_DIR}/b2)
    set(B2_ARGS install)
    message(VERBOSE "Executing b2 cmd: ${B2_CMD} ${B2_ARGS}")
    execute_process(COMMAND ${B2_CMD} ${B2_ARGS} WORKING_DIRECTORY ${boost_SOURCE_DIR} COMMAND_ECHO STDOUT RESULT_VARIABLE ret)

    # TODO Move to helper function
    if(NOT ret EQUAL 0)
        message( FATAL_ERROR "b2 failed. Bad exit status ${ret}")
    else()
        message(VERBOSE "b2 completed successfully")
    endif()
endif()



file(GLOB SRC CONFIGURE_DEPENDS 
    src/*.cpp)

file(GLOB INC CONFIGURE_DEPENDS 
    inc/*.h 
	inc/*.hpp)

file(GLOB BOOST_INC CONFIGURE_DEPENDS
    external/boost/include/*.hppl)


assign_source_group(${SRC})
assign_source_group(${INC})

add_executable(Stage2
    ${SRC} 
    ${INC})

# include paths
target_include_directories(Stage2 PRIVATE inc external/boost/include)

# lib search dir
target_link_directories(Stage2 PRIVATE ${PROJECT_SOURCE_DIR}/external/boost/lib)

# link libs 
target_link_libraries(Stage2 boost_program_options)
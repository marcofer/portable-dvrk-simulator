cmake_minimum_required(VERSION 3.5)
project(dvrk-portable-simulator VERSION 4.1 LANGUAGES C CXX)

# Set external system dependencies for each module
set(WITH_GEOMAGIC OFF)
set(WITH_OCULUS OFF)
set(WITH_ZEROMQ ON)

# CMake definitions
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT WITH_ZEROMQ)
    add_definitions(
        -DDO_NOT_USE_SHARED_MEMORY
        -D_CRT_SECURE_NO_WARNINGS
        -DNON_MATLAB_PARSING
        -DMAX_EXT_API_CONNECTIONS=255
    )
else()
    find_package(Boost REQUIRED)
    include_directories(${Boost_INCLUDE_DIR})
endif()




# Set header directories for each module
set(GEOMAGIC_HEADERS ${CMAKE_CURRENT_LIST_DIR}/include/Geomagic/)
set(OCULUS_HEADERS ${CMAKE_CURRENT_LIST_DIR}/include/Oculus/)
set(COPPELIASIM_DIR $ENV{COPPELIASIM_ROOT_DIR})
set(LIBZMQ $ENV{LIBZMQ})
string(REPLACE "\\" "/" COPPELIASIM_DIR "${COPPELIASIM_DIR}")
string(REPLACE "\\" "/" LIBZMQ "${LIBZMQ}")

# Specify source files for each external module
set(GEOMAGIC_SRCS 
    ${CMAKE_CURRENT_LIST_DIR}/src/Geomagic/*.cpp
    ${GEOMAGIC_HEADERS}/*.h*
)

set(OCULUS_SRCS 
    ${CMAKE_CURRENT_LIST_DIR}/src/Oculus/*.cpp
    ${OCULUS_HEADERS}/*.h*
)

### Default include directories
include_directories(
    ${CMAKE_CURRENT_LIST_DIR}/include
    $ENV{EIGEN}
)



if(NOT WITH_ZEROMQ)
    include_directories(
        $ENV{COPPELIASIM_ROOT_DIR}/programming/legacyRemoteApi/remoteApi/
        $ENV{COPPELIASIM_ROOT_DIR}/programming/include/
    )
    set(SRCS_FILES 
        ${SRCS_FILES} 
        ${COPPELIASIM_DIR}/programming/legacyRemoteApi/remoteApi/*.c
    )
else()
    include_directories(
        #${CMAKE_CURRENT_LIST_DIR}/include/CoppeliaSim/zmqRemoteApi/
        ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/
        ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/build/cppzmq
        ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/build/jsoncons/include/
        $ENV{LIBZMQ}\\include
    )
    add_compile_definitions(WITH_ZEROMQ)
endif()

if (WITH_GEOMAGIC)
    include_directories(
        ${GEOMAGIC_HEADERS}
        $ENV{OPEN_HAPTICS}/include
        $ENV{OPEN_HAPTICS}/utilities/include
    )
    set(SRCS_FILES ${SRCS_FILES} ${GEOMAGIC_SRCS})
    add_compile_definitions(WITH_GEOMAGIC)
endif()

if(WITH_OCULUS)
    include_directories(
        ${OCULUS_HEADERS}
        $ENV{OPEN_XR}/include
    )
    set(SRCS_FILES ${SRCS_FILES} ${OCULUS_SRCS})
    add_compile_definitions(WITH_OCULUS)
endif()

# Collect the whole set of sources to be built
set(SRCS_FILES
    ${SRCS_FILES}
    ${CMAKE_CURRENT_LIST_DIR}/src/*.cpp
    #${CMAKE_CURRENT_LIST_DIR}/src/CoppeliaSim/legacyRemoteApi/*.c
)
file(GLOB SRCS ${SRCS_FILES})

message(${SRCS_FILES})

# Project compile definitions
add_compile_definitions(PROJECT_NAME)

# Specify exe file to generate
add_executable(${PROJECT_NAME} ${SRCS})
target_compile_definitions(${PROJECT_NAME} PUBLIC -DSIM_REMOTEAPICLIENT_OBJECTS)

# Set compiling and linker options
#set_target_properties(${PROJECT_NAME} PROPERTIES COMMON_LANGUAGE_RUNTIME "")
#target_compile_options(${PROJECT_NAME} PUBLIC "/Gy;/Oi;/sdl;/MP;/ZW;")

#target_compile_options(${PROJECT_NAME} PUBLIC "/AI$ENV{WINDOWS_WINMD_PATH}")
#target_compile_options(${PROJECT_NAME} PUBLIC "/AI$ENV{PLATFORM_WINMD_PATH}")
#message("/AI$ENV{WINDOWS_WINMD_PATH};/AI$ENV{PLATFORM_WINMD_PATH}")

#set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /FORCE:MULTIPLE /SUBSYSTEM:CONSOLE /ENTRY:mainCRTStartup")

#### Collect the external library dependency list
#set(EXT_LIBS ${EXT_LIBS}  
#    ${CMAKE_CURRENT_LIST_DIR}/lib/LWR_Dynamic_Model_Lib.lib)
if(WITH_OCULUS)

endif()
if(WITH_GEOMAGIC)
    link_directories(
        $ENV{OPEN_HAPTICS}/lib/Win32/Release
        $ENV{OPEN_HAPTICS}/utilities/lib/Win32/Release
    )
    set(EXT_LIBS ${EXT_LIBS}
    $ENV{OPEN_HAPTICS}\\lib\\Win32\\Release\\hd.lib
    $ENV{OPEN_HAPTICS}\\lib\\Win32\\Release\\hl.lib
    $ENV{OPEN_HAPTICS}\\utilities\\lib\\Win32\\Release\\hdu.lib
    $ENV{OPEN_HAPTICS}\\utilities\\lib\\Win32\\Release\\hlu.lib
    $ENV{OPEN_HAPTICS}\\utilities\\lib\\Win32\\Release\\glut32.lib
    $ENV{OPEN_HAPTICS}\\utilities\\lib\\Win32\\Release\\glui32.lib
    )
endif()

### Link the list of libraries
message(${CMAKE_CONFIGURATION_TYPES})
target_link_libraries(${PROJECT_NAME} ${EXT_LIBS})
target_link_libraries(${PROJECT_NAME} ${COPPELIASIM_DIR}/programming/zmqRemoteApi/clients/cpp/build/${CMAKE_CONFIGURATION_TYPES}/RemoteAPIClient.lib)

if(${CMAKE_CONFIGURATION_TYPES} STREQUAL "Debug")
    set(LIBZMQ_LIB_NAME libzmq-v143-mt-gd-4_3_6.lib)
elseif(${CMAKE_CONFIGURATION_TYPES} STREQUAL "Release")
    set(LIBZMQ_LIB_NAME libzmq-v143-mt-4_3_6.lib)
endif()
target_link_libraries(${PROJECT_NAME} $ENV{LIBZMQ}/build/lib/${CMAKE_CONFIGURATION_TYPES}/${LIBZMQ_LIB_NAME})

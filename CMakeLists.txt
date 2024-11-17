cmake_minimum_required(VERSION 3.20)

project(iws_simu VERSION 1.0
    DESCRIPTION "Indoor Weather Station simulator"
    LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_C_STANDARD_REQUIRED ON)


# ADD LVGL THIRD-PARTY
add_subdirectory(third-party/lvgl)

set(LVGL_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/third-party/lvgl)
set(LV_CONF_PATH ${CMAKE_CURRENT_SOURCE_DIR}/config/lv_conf.h)
set(LV_BUILD_EXAMPLES OFF)

target_compile_definitions(lvgl PUBLIC LV_CONF_PATH=${LV_CONF_PATH})


# ADD SDL2
if(APPLE)
    find_package(SDL2 REQUIRED)
    include_directories(${SDL2_INCLUDE_DIRS})
    link_directories(${SDL2_LIBRARIES})
else()
    find_package(SDL2 REQUIRED)
endif()


# ADD LV_DRIVERS
add_subdirectory(third-party/lv_drivers)

set(LV_DRIVERS_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/third-party/lv_drivers)
set(LV_DRV_CONF_PATH ${CMAKE_CURRENT_SOURCE_DIR}/config/)

target_include_directories(lv_drivers BEFORE PUBLIC ${LV_DRV_CONF_PATH})


# ADD GUI AND PROXY
add_subdirectory(src/gui)
add_subdirectory(src/proxy)


# ADD .cpp & .h
set(SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

set(SRC
    ${SRC_DIR}/main.cpp
)


# LINK
add_executable(${PROJECT_NAME} ${SRC})
target_include_directories(${PROJECT_NAME} PUBLIC ${LVGL_HEADERS} ${LV_DRIVERS_HEADERS} ${SRC_DIR})
target_link_libraries(${PROJECT_NAME} PRIVATE lvgl lv_drivers ${SDL2_LIBRARIES} gui proxy)


# BUILD OPT
target_compile_definitions(${PROJECT_NAME} PRIVATE $<$<CONFIG:Debug>:DEBUG>)
if (CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic)
endif()

cmake_minimum_required(VERSION 3.8)
project(serial)

# Find dependencies
find_package(ament_cmake REQUIRED)

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit REQUIRED)
    find_library(FOUNDATION_LIBRARY Foundation REQUIRED)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)

if(APPLE)
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX AND NOT APPLE)
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} SHARED ${serial_SRCS})
if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX AND NOT APPLE)
    target_link_libraries(${PROJECT_NAME} pthread rt)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

## Include directories
target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

## Install library and headers
install(
    TARGETS ${PROJECT_NAME}
    EXPORT export_${PROJECT_NAME}
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

install(
    DIRECTORY include/
    DESTINATION include
)

## Add serial example executable
add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example ${PROJECT_NAME})

## Install example executable
install(TARGETS serial_example
    DESTINATION lib/${PROJECT_NAME}
)

## Tests
if(BUILD_TESTING)
    find_package(ament_cmake_gtest REQUIRED)
    add_subdirectory(tests)
endif()

## Ament package registration
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_package()

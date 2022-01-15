cmake_minimum_required(VERSION 3.16)
project(serial)

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

if(UNIX AND NOT APPLE)
    find_package(Threads)  
endif()

## Sources
set(serial_SRCS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/serial.cc
)
if(APPLE)
    # If OSX
    list(APPEND serial_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/impl/unix.cc)
    list(APPEND serial_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/impl/unix.cc)
    list(APPEND serial_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/impl/win.cc)
    list(APPEND serial_SRCS ${CMAKE_CURRENT_SOURCE_DIR}/src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})
if(APPLE)
    target_link_libraries(${PROJECT_NAME} PUBLIC ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} INTERFACE Threads::Threads)
else()
    target_link_libraries(${PROJECT_NAME} PUBLIC setupapi)
endif()

## Uncomment for example
add_executable(serial_example ${CMAKE_CURRENT_SOURCE_DIR}/examples/serial_example.cc)
target_link_libraries(serial_example PRIVATE ${PROJECT_NAME})

## Include headers
target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

## Install executable
#install(TARGETS ${PROJECT_NAME}
#    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#    RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
#)

## Install headers
#install(FILES include/serial/serial.h include/serial/v8stdint.h
#  DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}/serial)

## Tests
if(ENABLE_TESTING)
    add_subdirectory(tests)
endif()

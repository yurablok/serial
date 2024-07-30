cmake_minimum_required(VERSION 3.12)
project(serial)

set(serial_SRCS
    "src/serial.cc"
    "include/serial/serial.h"
    "include/serial/v8stdint.h"
)
if(APPLE) # If OSX
    list(APPEND serial_SRCS "src/impl/unix.cc")
    list(APPEND serial_SRCS "src/impl/list_ports/list_ports_osx.cc")
elseif(UNIX) # If unix
    list(APPEND serial_SRCS "src/impl/unix.cc")
    list(APPEND serial_SRCS "src/impl/list_ports/list_ports_linux.cc")
else() # If windows
    list(APPEND serial_SRCS "src/impl/win.cc")
    list(APPEND serial_SRCS "src/impl/list_ports/list_ports_win.cc")
endif()

add_library(${PROJECT_NAME} STATIC ${serial_SRCS})

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

if(SERIAL_WITH_EXAMPLES)
    add_executable(serial_example "examples/serial_example.cc")
    add_dependencies(serial_example ${PROJECT_NAME})
    target_link_libraries(serial_example ${PROJECT_NAME})
endif()

target_include_directories(${PROJECT_NAME} PUBLIC "include")

if(CATKIN_ENABLE_TESTING)
    add_subdirectory(tests)
endif()

unset(serial_SRCS)

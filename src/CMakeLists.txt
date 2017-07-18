cmake_minimum_required (VERSION 3.3)

include_directories(${LIBUSB_INCLUDE_DIRS} ${PTHREAD_INCLUDE_DIRS})

if(WIN32)
    add_definitions(-DHAVE_WINDOWS_H)
else()
    add_definitions(-DHAVE_CONFIG_H)
    add_definitions(-DPACKAGE_STRING="dfu-util 0.9")
    add_definitions(-DPACKAGE_BUGREPORT="http://sourceforge.net/p/dfu-util/tickets/")
    add_definitions(-DPACKAGE="dfu-util")
    add_definitions(-DPACKAGE_VERSION="0.9")
    add_definitions(-DHAVE_UNISTD_H)
    add_definitions(-DHAVE_NANOSLEEP)
    add_definitions(-DHAVE_ERR)
    add_definitions(-DHAVE_SYSEXITS_H)
endif()

set(THREADS_PREFER_PTHREAD_FLAG ON)
set(THREADS_USE_PTHREADS_WIN32 ON)
find_package(Threads REQUIRED)

#target_link_libraries(dfu PUBLIC ${LIBUSB_LIBRARIES})

add_executable(dfu-util
               dfu.c
               dfu_file.c
               dfu_load.c
               dfuse.c
               dfuse_mem.c
               quirks.c
               dfu_util.c
               main.c)

add_dependencies(dfu-util libusb)
target_link_libraries(dfu-util PUBLIC
    ${LIBUSB_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT}
)

if(WIN32)
  include_directories(${PTHREAD_INCLUDE_DIRS})
  target_link_libraries(dfu-util PUBLIC ${PTHREAD_LIBRARIES})
else()
   target_link_libraries(dfu-util PUBLIC Threads::Threads)
endif()

install(TARGETS dfu-util
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/static)

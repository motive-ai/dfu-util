cmake_minimum_required (VERSION 3.3)
PROJECT (dfu C CXX )
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/config.h "")

set(THREADS_PREFER_PTHREAD_FLAG ON)
set(THREADS_USE_PTHREADS_WIN32 ON)
find_package(Threads REQUIRED)

add_library(dfulib dfu.c
                dfu_file.c
                dfu_load.c
                dfuse.c
                dfuse_mem.c
                quirks.c
                dfu_util.c)

target_link_libraries(dfulib PUBLIC Usb::Usb)

if(WIN32)
    target_compile_definitions(dfulib PUBLIC HAVE_WINDOWS_H)
else()
    target_compile_definitions(dfulib PUBLIC
                HAVE_CONFIG_H
                HAVE_UNISTD_H
                HAVE_NANOSLEEP
                HAVE_ERR
                HAVE_SYSEXITS_H
                PACKAGE_STRING="dfu-util 0.9"
                PACKAGE_BUGREPORT="http://sourceforge.net/p/dfu-util/tickets/"
                PACKAGE="dfu-util"
                PACKAGE_VERSION="0.9"
            )
endif()

target_include_directories(dfulib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

add_executable(dfu-util
               main.c)

add_executable(dfu-suffix
               suffix.c)

foreach(program dfu-util dfu-suffix)
    target_link_libraries(${program} PRIVATE
         ${CMAKE_THREAD_LIBS_INIT} dfulib
    )
    if(WIN32)
      target_include_directories(${program} PUBLIC ${PTHREAD_INCLUDE_DIRS})
      target_link_libraries(${program} PUBLIC ${PTHREAD_LIBRARIES})
    endif()
endforeach()

set(DFU_SUFFIX_EXE $<TARGET_FILE:dfu-suffix> CACHE INTERNAL "dfu-suffix executable")

install(TARGETS dfu-util dfu-suffix
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/static)

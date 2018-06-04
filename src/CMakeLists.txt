cmake_minimum_required (VERSION 3.3)

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/config.h "")

set(THREADS_PREFER_PTHREAD_FLAG ON)
set(THREADS_USE_PTHREADS_WIN32 ON)
find_package(Threads REQUIRED)

set(DFU_LIB_SRC dfu.c
                dfu_file.c
                dfu_load.c
                dfuse.c
                dfuse_mem.c
                quirks.c
                dfu_util.c)

add_executable(dfu-util
               ${DFU_LIB_SRC}
               main.c)

add_executable(dfu-suffix
               dfu_file.c
               dfu_load.c
               dfu.c
               suffix.c)

foreach(program dfu-util dfu-suffix)
    target_include_directories(${program} PUBLIC
        ${CMAKE_CURRENT_BINARY_DIR}
        ${LIBUSB_INCLUDE_DIRS}
        ${PTHREAD_INCLUDE_DIRS}
    )
    target_link_libraries(${program} PRIVATE
        Usb::Usb ${CMAKE_THREAD_LIBS_INIT}
    )
    if(WIN32)
      target_include_directories(${program} PUBLIC ${PTHREAD_INCLUDE_DIRS})
      target_link_libraries(${program} PUBLIC ${PTHREAD_LIBRARIES})
    else()
       target_link_libraries(${program} PUBLIC Threads::Threads)
    endif()
    if(WIN32)
        target_compile_definitions(${program} PRIVATE HAVE_WINDOWS_H)
    else()
        target_compile_definitions(${program} PRIVATE
            HAVE_CONFIG_H
            PACKAGE_STRING="dfu-util 0.9"
            PACKAGE_BUGREPORT="http://sourceforge.net/p/dfu-util/tickets/"
            PACKAGE="dfu-util"
            PACKAGE_VERSION="0.9"
            HAVE_UNISTD_H
            HAVE_NANOSLEEP
            HAVE_ERR
            HAVE_SYSEXITS_H
        )
    endif()
endforeach()

set(DFU_SUFFIX_EXE $<TARGET_FILE:dfu-suffix> CACHE INTERNAL "dfu-suffix executable")

install(TARGETS dfu-util dfu-suffix
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib/static)

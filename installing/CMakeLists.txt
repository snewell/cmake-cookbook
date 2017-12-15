cmake_minimum_required(VERSION 3.0)
cmake_policy(VERSION 3.0)

project("hello-world"
        LANGUAGES
            C
        VERSION
            1.0.0
       )

# CMake provides the default directories for this system if we pull in a helper
# file
include(GNUInstallDirs)

add_library(hello-static STATIC hello.c)
set_target_properties(hello-static PROPERTIES
                      OUTPUT_NAME hello
                     )

add_library(hello-shared SHARED hello.c)
set_target_properties(hello-shared PROPERTIES
                      OUTPUT_NAME hello
                      SOVERSION ${PROJECT_VERSION_MAJOR}
                      VERSION ${PROJECT_VERSION}
                     )

add_executable(hello hello-world.c)
target_link_libraries(hello
                        hello-shared
                     )

install(TARGETS
            hello
            hello-shared
            hello-static
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}  # static libraries
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}  # shared libraries
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}  # programs
       )
install(FILES
            hello.h
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
       )

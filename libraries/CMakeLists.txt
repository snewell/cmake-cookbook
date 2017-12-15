cmake_minimum_required(VERSION 3.0)
cmake_policy(VERSION 3.0)

project("hello-world"
        LANGUAGES
            C
        # we'll actually use the version in this example
        VERSION
            1.0.0
       )

add_library(hello-static STATIC hello.c)
set_target_properties(hello-static PROPERTIES
                      OUTPUT_NAME hello
                     )

add_library(hello-shared SHARED hello.c)
set_target_properties(hello-shared PROPERTIES
                      OUTPUT_NAME hello
                      # PROJECT_VERSION and PROJECT_VERSION_MAJOR are
                      # automatically populated based on the VERSION argument
                      # provided in the project command
                      SOVERSION ${PROJECT_VERSION_MAJOR}
                      VERSION ${PROJECT_VERSION}
                     )

add_executable(hello hello-world.c)
target_link_libraries(hello
                        hello-shared
                     )
add_executable(hellob hello-world.c)
target_link_libraries(hellob
                        hello-static
                     )

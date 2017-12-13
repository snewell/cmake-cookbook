# we want to use the defaults as of cmake 3.0
cmake_minimum_required(VERSION 3.0)
cmake_policy(VERSION 3.0)

project("hello-world"
        # the LANGUAGES option is technically optional, but it's considered
        # good practice to include it
        LANGUAGES
            C
        # we'll leverage project versions in the future, so get used to adding
        # them
        VERSION 1.0.0
       )

# We have to give our program a name (hello) and a list of source files
# (hello.c).  Even if you're on a machine that has a prefix/suffix for
# executables (e.g., Windows uses .exe) just put the unique part of the name;
# CMake will automatically add what's needed.
add_executable(hello hello.c)

# you can list as many source files as you want
add_executable(hello2
                hello2.c
                hello-world2.c
              )

# you can also use variables to set a list of source files
set(hello2_srcs
        hello2.c
        hello-world2.c
   )
add_executable(hello2b ${hello2_srcs})

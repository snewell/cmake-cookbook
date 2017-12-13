Simple Executables
==================

Getting Started
---------------
All CMake projects are driven by a file with a very specific name:
:code:`CMakeLists.txt` (note the capitalization).  Everything your project
needs to run has to go in these files.  Eventually we'll build complex
projects with many files across directories, but for now we'll keep things
simple.


Single File
-----------
Let's start with a simple hello world program.  Assuming you have your
implementation in the file :code:`hello.c`, enter the following in
:code:`CMakeLists.txt`:

.. code:: CMake

  cmake_minimum_required(VERSION 3.0)
  cmake_policy(VERSION 3.0)

  project("hello-world"
          LANGUAGES
            C
          VERSION 1.0.0
         )

  add_executable(hello hello.c)

Everything besides the :code:`add_executable` line is technically optional, but
it's good form to include it.  The CMake version you list should be old enough
that anybody building your project will have it available, and for the most
part you're safe going with an older version (3.0 isn't a bad place to start).
Once you have your file ready, it's time to configure your project.

Make a build directory (CMake prefers out-of-place builds) and run
:code:`cmake`.

.. code::

  $ cmake /path/to/your/project
  -- The C compiler identification is GNU 6.4.0
  -- Check for working C compiler: /usr/bin/cc
  -- Check for working C compiler: /usr/bin/cc -- works
  -- Detecting C compiler ABI info
  -- Detecting C compiler ABI info - done
  -- Detecting C compile features
  -- Detecting C compile features - done
  -- Configuring done
  -- Generating done
  -- Build files have been written to: /tmp/build

If you don't get any errors, you're ready to build your project.  If you're on
a Unix system CMake will default to makefiles, so go ahead and run
:code:`make`.

.. code::

  $ make
  [ 50%] Building C object CMakeFiles/hello.dir/hello.c.o
  [100%] Linking C executable hello
  [100%] Built target hello
  $ ./hello
  Hello, CMake

Anything you list using :code:`add_executable` will be a target in the
generated Makefile, so :code:`make hello` would also work.

You can also let CMake drive the build process for you using the
:code:`--build` flag.  This will work regardless of the build genertaor CMake
used.  This option requires telling :code:`cmake` what build directory to use,
so if you're still in the configuration directory your command will look like
this (specifying :code:`--target` is optional).

.. code::

  $ cmake --build . --target hello
  [ 50%] Building C object CMakeFiles/hello.dir/hello.c.o
  [100%] Linking C executable hello
  [100%] Built target hello


Multiple Files
--------------
In most cases your project won't be a single file, (let's assume your
:code:`main` calls a function to do the actual printing).  CMake lets you list
as many source files as you wish with :code:`add_executable`, so add something
like this to your :code:`CMakeLists.txt`:

.. code:: CMake

  add_executable(hello2
                   hello2.c
                   hello-world2.c
                )

If all went well, you should be able to just run the build command again and
have new program (whether you run :code:`make` or :code:`cmake --build`, it'll
check to see if the build configuration changed).

You can also use a variables to help if you have lots of sources.  This is the
approach I use for anything but the most trivial cases, mostly because it's
easier to manage as projects become more complex.

.. code:: CMake

  set(hello2_srcs
        hello2.c
        hello-world2.c
     )
  add_executable(hello2b ${hello2_srcs})


Example Files
-------------
- `CMakeLists.txt`_ used for these examples
- `hello.c`_ from our first example
- `hello2.c`_, `hello2.h`_, and `hello-world2.c`_ from the second example

.. _CMakeLists.txt: CMakeLists.txt
.. _hello.c: hello.c
.. _hello2.c: hello2.c
.. _hello2.h: hello2.h
.. _hello-world2.c: hello-world2.c

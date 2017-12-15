Installing
==========
We've built some stuff now, but nobody can use it since it only lives in build
directories.  Let's teach CMake how to install the awesome code we're writing.

If we start from a project similar to our `libraries example`_, we have four
things that need to be installed:

1. :code:`hello-static` (a static library)
2. :code:`hello-shared` (a shared library)
3. :code:`hello` (an executable program)
4. :code:`hello.h` (a header file clients of :code:`hello-static` and
   :code:`hello-shared` will use)


Installing Targets
------------------
CMake targets (e.g., anything you build via :code:`add_executable` or
:code:`add_library`) can be insatlled using the :code:`install` command.

.. code:: CMake

  install(TARGETS
            hello
            hello-shared
            hello-static
          ARCHIVE DESTINATION lib
          LIBRARY DESTINATION lib
          RUNTIME DESTINATION bin
         )

It looks more complicated than it actually is.  First, we have to tell CMake
that we're using this to install targets (we'll cover other things to install
later).  Next, we simply list the targets (items 1 - 3 above).

After that we need to tell CMake where to install our files.  We provide the
locations by saying where each type of file should be installed using one of
the following specifiers:

ARCHIVE
  Static libraries.  On a Unix system, this is traditionally :code:`lib`.

LIBRARY
  Shared libraries.  This is also :code:`lib` on traditional Unix systems.

RUNTIME
  Programs (and on Windows, dlls).  If you're going with defaults, this should
  be :code:`bin`.

You only have to provide :code:`DESTINATION` values for relevant targets you're
installing, but CMake will ignore anything it doesn't need for the command,
meaning it's easier to specify all three so you don't have to worry about
OS-specific things (like dlls falling under :code:`RUNTIME`).  You're also free
to use either absolute or relative paths, although relative is strongly
encouraged.

Now that our :code:`CMakeLists.txt` is up to date, give :code:`make install` a
try.

You'll probably get an error.

.. code::

  $ make install
  [ 33%] Built target hello-shared
  [ 66%] Built target hello
  [100%] Built target hello-static
  Install the project...
  -- Install configuration: ""
  -- Installing: /usr/local/bin/hello
  CMake Error at cmake_install.cmake:42 (file):
    file INSTALL cannot copy file "/your/build/dir/hello" to
    "/usr/local/bin/hello".


  make: *** [Makefile:86: install] Error 1

The issue is that you're (hopefully) running as a user who doesn't have
permission to write to the root filesystem.  For now, you can specify
:code:`DESTDIR` on the command line to have CMake install in a temporary
directory (this is useful if you want to tar your installation up).  If you're
using something other than Makefiles (e.g., Ninja) you'll have to consult the
documentation to see what to use instead of :code:`DESTDIR`.

.. code::

  $ make install DESTDIR=_install_
  [ 33%] Built target hello-shared
  [ 66%] Built target hello
  [100%] Built target hello-static
  Install the project...
  -- Install configuration: ""
  -- Installing: _install_/usr/local/bin/hello
  -- Set runtime path of "_install_/usr/local/bin/hello" to ""
  -- Installing: _install_/usr/local/lib/libhello.so.1.0.0
  -- Installing: _install_/usr/local/lib/libhello.so.1
  -- Installing: _install_/usr/local/lib/libhello.so
  -- Installing: _install_/usr/local/lib/libhello.a
  $ ls -l _install_/usr/local/lib/
  total 12
  -rw-r--r-- 1 stephen stephen 1676 Dec 14 21:21 libhello.a
  lrwxrwxrwx 1 stephen stephen   13 Dec 14 21:22 libhello.so -> libhello.so.1
  lrwxrwxrwx 1 stephen stephen   17 Dec 14 21:22 libhello.so.1 -> libhello.so.1.0.0
  -rwxr-xr-x 1 stephen stephen 7992 Dec 14 21:21 libhello.so.1.0.0
  $ ldd _install_/usr/local/bin/hello
          linux-vdso.so.1 (0x00007ffdc9dba000)
          libhello.so.1 => not found
          libc.so.6 => /lib64/libc.so.6 (0x00007fdc20cfc000)
          /lib64/ld-linux-x86-64.so.2 (0x00007fdc210ab000)

A couple interesting things happened here.

- CMake automatically handled the library symlinks for us.
- CMake stripped rpath info from our binaries.  This means if you try to run
  :code:`hello` from the installation directory, it won't work.


Installing Files
----------------
The only thing left to do is install our header; again, this is really easy.

.. code:: CMake

  install(FILES
            hello.h
          DESTINATION include
         )

Everything should be pretty self-explanatory.  If we do another
:code:`make install`, our header will be added to the install path.

.. code::

  $ make install DESTDIR=_install_/
  [ 33%] Built target hello-shared
  [ 66%] Built target hello
  [100%] Built target hello-static
  Install the project...
  -- Install configuration: ""
  -- Up-to-date: _install_/usr/local/bin/hello
  -- Up-to-date: _install_/usr/local/lib/libhello.so.1.0.0
  -- Up-to-date: _install_/usr/local/lib/libhello.so.1
  -- Up-to-date: _install_/usr/local/lib/libhello.so
  -- Up-to-date: _install_/usr/local/lib/libhello.a
  -- Installing: _install_/usr/local/include/hello.h


Customizing the Prefix
----------------------
The astute reader will notice :code:`/usr/local/` coming up a lot; that's
because the installation paths we specified are all relative paths.  When
using relative paths, CMake will append them to the variable
:code:`CMAKE_INSTALL_PREFIX` (by default on my system: :code:`/usr/local`).  If
you want to specify the prefix, you can do that when you configure your
build directory, or switch it when you want.

.. code::

  $ cmake -DCMAKE_INSTALL_PREFIX=/some/weird/path .-- Configuring done
  -- Generating done
  -- Build files have been written to: /your/build/path
  $ make install DESTDIR=_install_2_
  [ 33%] Built target hello-shared
  [ 66%] Built target hello
  [100%] Built target hello-static
  Install the project...
  -- Install configuration: ""
  -- Installing: _install_2_/some/weird/path/bin/hello
  -- Set runtime path of "_install_2_/some/weird/path/bin/hello" to ""
  -- Installing: _install_2_/some/weird/path/lib/libhello.so.1.0.0
  -- Installing: _install_2_/some/weird/path/lib/libhello.so.1
  -- Installing: _install_2_/some/weird/path/lib/libhello.so
  -- Installing: _install_2_/some/weird/path/lib/libhello.a
  -- Installing: _install_2_/some/weird/path/include/hello.h

You'll be safe ignoring this most of the time (it's something packagers and
build systems care about more than day-to-day developers), but included for
the sake of completeness.


Standard Installation Paths
---------------------------
There's one last thing to cover: standard installation paths.  While
:code:`lib`, :code:`bin`, and :code:`include` are usually the defaults, there
are some systems where this isn't the case (e.g., 64-bit systems often using
:code:`lib64`).  Once again, CMake has us covered.

First, we need to get some help; add the following line to your
:code:`CMakeLists.txt`:

.. code:: CMake

  include(GNUInstallDirs)

You can add that pretty much wherever you want, but I like it near the top
(usually right under the :code:`project` command); this keeps it out of the way
of things that change more often (targets and source files).  Now that it's
available, we have some extra variables that provide our system defaults; let's rewite our :code:`install` commands to leverage this:

.. code:: CMake

  install(TARGETS
            hello
            hello-shared
            hello-static
          ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
          LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
          RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
         )
  install(FILES
            hello.h
          DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
         )

If it's not obvious, the pattern for the variable is
:code:`CMAKE_INSTALL_XXXDIR` (check the GNUInstallDirs_ documentation for the
full list of variables).  Now our :code:`make install` should look something
like this:

.. code::

  $ make install DESTDIR=_install_
  [ 33%] Built target hello-shared
  [ 66%] Built target hello
  [100%] Built target hello-static
  Install the project...
  -- Install configuration: ""
  -- Installing: _install_/some/weird/path/bin/hello
  -- Set runtime path of "_install_/some/weird/path/bin/hello" to ""
  -- Installing: _install_/some/weird/path/lib64/libhello.so.1.0.0
  -- Installing: _install_/some/weird/path/lib64/libhello.so.1
  -- Installing: _install_/some/weird/path/lib64/libhello.so
  -- Installing: _install_/some/weird/path/lib64/libhello.a
  -- Installing: _install_/some/weird/path/include/hello.h

The observant readers will notice the libraries installed to :code:`lib64`
instead of :code:`lib`; that's because CMake figured out it's being run for a
64-bit environment.  Using these variables also makes it much easier for
distribution packagers and build systems to customize the path if necessary.


Example Files
-------------
- `CMakeLists.txt`_ used for these examples
- `hello.c`_, `hello.h`_, and `hello-world.c`_ used for our libraries and
  executables

.. _CMakeLists.txt: CMakeLists.txt
.. _GNUInstallDirs: https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html
.. _hello.c: hello.c
.. _hello.h: hello.h
.. _hello-world.c: hello-world.c
.. _libraries example: ../libraries/CMakeLists.txt

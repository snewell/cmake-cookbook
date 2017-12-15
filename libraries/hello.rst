Compiled Libraries
==================

Static Libraries
----------------
As you may expect, creating a library is similar to our executable example.
The only real differences are that we use :code:`add_library` and that we have
to tell CMake what kind of library we want to build.  Just like with an
executable, we don't need to tell CMake to add any system-specific prefix or
suffix.

.. code:: CMake

  add_library(hello-static STATIC hello.c)

Great, we have a static library.  Unfortunately, it doesn't have a great name:

.. code::

  $ make
  [ 50%] Building C object CMakeFiles/hello-static.dir/hello.c.o
  [100%] Linking C static library libhello-static.a
  [100%] Built target hello-static
  $ ls | grep hello
  libhello-static.a

The reason for this is because we gave it the name :code:`hello-static`.
Luckily, CMake makes it easy to change the name it actually uses.

.. code:: CMake

  set_target_properties(hello-static PROPERTIES
                        OUTPUT_NAME hello
                       )

Now you can keep your target names explicit, but you can get output that passes
the sniff test.

.. code::

  $ make hello-static
  [ 50%] Building C object CMakeFiles/hello-static.dir/hello.c.o
  [100%] Linking C static library libhello.a
  [100%] Built target hello-static


Shared Libraries
----------------
Shared libraries are almost identical to static libraries, so you can probably
write the first part by yourself by modifying the above example.  For the sake
of completeness though, here's what it'll look like.

.. code:: CMake

  add_library(hello-shared SHARED hello.c)
  set_target_properties(hello-shared PROPERTIES
                        OUTPUT_NAME hello
                       )

If you're building from a clean environment, you'll notice something
interesting: CMake compiled :code:`hello.c` twice.  Although this sounds
wasteful, CMake is actually looking out for you.  Shared libraries usually
require position independant code whereas static libraries don't, so CMake
automatically uses the flags required for each target.  In addition, you can
customize the flags used to build shared libraries versus static libraries, so
the two compilations are required.

.. code::

  $ ls -l | grep hello.so
  -rwxr-xr-x 1 stephen stephen  7992 Dec 14 19:26 libhello.so

If you're a long-time Unix user, you've spotted something less than satisfying
with our library: we only have one file.  Usually we'd want a library to
contain version information and have the appropriate symlinks so we can install
multiple versions side by side.

.. code::

  # a random example from my system
  $ ls -l /usr/lib/libkaccounts.so*
  lrwxrwxrwx 1 root root    17 Dec  2 05:44 /usr/lib/libkaccounts.so -> libkaccounts.so.1
  lrwxrwxrwx 1 root root    23 Dec  2 05:44 /usr/lib/libkaccounts.so.1 -> libkaccounts.so.17.08.3
  -rwxr-xr-x 1 root root 48040 Dec  2 05:44 /usr/lib/libkaccounts.so.17.08.3

As you may expect, this is really easy with CMake; all we have to do is set a
few new properties.

.. code:: CMake

  set_target_properties(hello-shared PROPERTIES
                        OUTPUT_NAME hello
                        SOVERSION ${PROJECT_VERSION_MAJOR}
                        VERSION ${PROJECT_VERSION}
                       )

Now if we build we'll get the symlinks we expect.

.. code::

  $ ls -l | grep hello.so
  lrwxrwxrwx 1 stephen stephen    13 Dec 14 19:36 libhello.so -> libhello.so.1
  lrwxrwxrwx 1 stephen stephen    17 Dec 14 19:36 libhello.so.1 -> libhello.so.1.0.0
  -rwxr-xr-x 1 stephen stephen  7992 Dec 14 19:36 libhello.so.1.0.0

If it's not obvious, :code:`SOVERSION` maps to middle entry
(:code:`libhello.so.${SOVERSION}`) and :code:`VERSION` maps to the fully
versioned one.  You're free to set any values you'd like, including
non-numeric ones, but I find :code:`PROJECT_VERSION` and
:code:`PROJECT_VERSION_MAJOR` work well in practice.


Using a Library
---------------
Libraries don't do much unless you use them, so let's put our libraries to work.

.. code:: CMake

  add_executable(hello hello-world.c)

And when we try to build...

.. code::

  $ make
  [ 16%] Building C object CMakeFiles/hello.dir/hello-world.c.o
  [ 33%] Linking C executable hello
  CMakeFiles/hello.dir/hello-world.c.o: In function `main':
  hello-world.c:(.text+0x5): undefined reference to `print_hello'
  collect2: error: ld returned 1 exit status
  make[2]: *** [CMakeFiles/hello.dir/build.make:95: hello] Error 1
  make[1]: *** [CMakeFiles/Makefile2:68: CMakeFiles/hello.dir/all] Error 2
  make: *** [Makefile:84: all] Error 2

Well that didn't work.  Turns out we need to actually link against the library
too, so we'll use :code:`target_link_libraries` to handle the heavy lifting.

.. code:: CMake

  target_link_libraries(hello
                          hello-shared
                       )

Now the build should finish and we can great CMake the way it deserves.  We can
even use :code:`ldd` to verify it's using our library.

.. code::

  $ ldd hello
  linux-vdso.so.1 (0x00007ffe40e5b000)
  libhello.so.1 => /your/build/dir/libhello.so.1 (0x00007f47730a3000)
  libc.so.6 => /lib64/libc.so.6 (0x00007f4772cf4000)
  /lib64/ld-linux-x86-64.so.2 (0x00007f47732a5000)

You can add as many libraries as you wish to a single
:code:`target_link_libraries` command if you have a complex build.

We can use our static library too, and you've probably already guessed the
exact code we'd have to write.

.. code:: CMake

  add_executable(hellob hello-world.c)
  target_link_libraries(hellob
                          hello-static
                       )

If we consult :code:`ldd`, we can verify it linked against the static version
of :code:`libhello`.

.. code::

  $ ldd hellob
  linux-vdso.so.1 (0x00007fff739d5000)
  libc.so.6 => /lib64/libc.so.6 (0x00007f67af5eb000)
  /lib64/ld-linux-x86-64.so.2 (0x00007f67af99a000)


Example Files
-------------
- `CMakeLists.txt`_ used for these examples
- `hello.c`_, `hello.h`_, and `hello-world.c`_ used for our libraries and
  executables

.. _CMakeLists.txt: CMakeLists.txt
.. _hello.c: hello.c
.. _hello.h: hello.h
.. _hello-world.c: hello-world.c

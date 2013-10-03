<meta charset="utf-8">

oosmake
=======

Out-of-source make wrapper

## What is `oosmake`? Why is it useful?

`oosmake` is a shell script that wraps `make` by trying to find the
correct out-of-source build directory for a project. (An out-of-source
build directory is a directory used for separating build files, such as
executables and object files, from the canonical files in a source
project. Out-of-source builds are common when using CMake.)

You may find `oosmake` useful if you're (1) doing out-of-source builds
and (2) using Vim. Rather than changing directories to run `make`, or
using `make`'s `-C` parameter, you may instead set Vim's `makeprg`
setting to `oosmake` and the Vim command `:make` will "do the right
thing"—regardless whether you're doing an out-of-source build or doing
a normal, in-source build.

## How does it work?

`oosmake` works by recursively ascending the file system tree, starting
from the current directory, until either it finds an out-of-source
makefile or else runs out of directories. `oosmake`'s behavior is shown
in the following examples.

### Example 1

Given a project directory, `$P`, with an out-of-source build directory
`$P/build/` containing a makefile, as shown below:

    $P/                         (no makefile in this directory)
    $P/build/makefile

Running `oosmake` from `$P` is like running:

    cd $P/build && make

Or, alternately:

    make -C $P/build

### Example 2

Given a project with an out-of-source build directory, but the root
project directory has a makefile, too:

    $P/makefile
    $P/build/makefile

Running `oosmake` from `$P` is like running `make` from the `$P`
directory. In other words, the out-of-source build directory is ignored.

### Example 3

Given a project with subdirectories, with all makefiles in the
out-of-source build directory:

    $P/                         (no makefile in this directory)
    $P/subdir1/                 (no makefile in this directory)
    $P/subdir2/                 (no makefile in this directory)
    $P/build/makefile
    $P/build/subdir1/makefile
    $P/build/subdir2/           (no makefile in this directory)

Running `oosmake` from `$P` is like running:

    cd $P/build && make

Running `oosmake` from `$P/subdir1` is like running:

    cd $P/build/subdir1 && make

Running `oosmake` from `$P/subdir2` will fail to find a makefile to run
because both `$P/subdir2` and `$P/build/subdir2` lack a makefile.
However, see the next example.

### Example 4

Nested out-of-source directories lacking a makefile will be skipped.

    $P/                         (no makefile in this directory)
    $P/subdir1/                 (no makefile in this directory)
    $P/subdir1/subdir2/         (no makefile in this directory)
    $P/build/makefile
    $P/build/subdir1/makefile
    $P/build/subdir1/subdir2/   (no makefile in this directory)

Running `oosmake` from `$P/subdir1/subdir2` is like running:
    cd $P/build/subdir1 && make

Running `oosmake` from `$P/subdir1` is like running:
    cd $P/build/subdir1 && make

In other words, if the out-of-source build subdirectory corresponding to
the current directory lacks a makefile, then `oosmake` will continue
ascending out-of-source directories until either a makefile is found or
the root project directory is been reached.

## Features and limitations

- All command line options are passed as-are to the sub-`make` process.
  This includes `-C` and `-f`, which may mess up what you're trying to
  do.

- Out-of-source builds are detected by testing for the existence of well
  named makefiles—`GNUmakefile`, `Makefile`, and `makefile`. Thus,
  running `oosmake -f my-custom-makefile` in the directory where
  `my-custom-makefile` is located will fail to run `make` in the current
  directory because `oosmake` looks for well named makefiles, not your
  custom-named makefile.

- The root out-of-source build directory must be named `build`.
  Additional idiomatic patterns could be added in the future, as those
  patterns are discovered.

- There's no limit to the number of nested subdirectories. The recursive
  ascent algorithm will continue until either a makefile is found or
  else the root file system directory (`/`) is attempted. The guard
  against mistakenly running an out-of-project makefile is that the
  out-of-source build directory must match the ascent path. For example,
  given the current working directory
  `/home/craig/my_project/libfoo`, and no makefiles, then when `oosmake`
  tries the root directory it will search for a makefile in
  `/build/home/craig/my_project/libfoo`. (See example 3 above.) This
  guard isn't failsafe, but it makes the probability of running
  an out-of-project makefile small, as for that to happen you must have
  a spurious `build` directory containing a matching directory
  hierarchy.


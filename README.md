<meta charset="utf-8">
<!-- vim: set tw=72: -->

oosmake
=======

Out-of-source make wrapper

## What is `oosmake`? Why is it useful?

`oosmake` is a shell script that wraps `make` by finding the correct
out-of-source build directory or non-recursive makefile for the current
directory. An out-of-source build directory is a directory used for
separating build files, such as executables and object files, from the
canonical files in a source project. Out-of-source builds are common
when using CMake.

You may find `oosmake` useful if you're (1) doing out-of-source or
non-recursive builds and (2) using Vim. Rather than changing directories
to run `make`, or using `make`'s `-C` parameter, you may instead set
Vim's `makeprg` option to `oosmake`, and the Vim command `:make` will
“do the right thing”—regardless whether you're doing an out-of-source
build, a non-recursive build, or a normal in-source build.

## How does it work?

`oosmake` works by recursively ascending the file system tree, starting
in the current directory, until either it finds a makefile, an
out-of-source makefile, or else runs out of directories. The following
examples show `oosmake`'s behavior.

### Example 1

Given a project directory, `$P`, with an out-of-source build directory
`$P/build/` containing a makefile, as shown below:

    $P/                         (no makefile in this directory)
    $P/build/makefile

Running `oosmake` from `$P` is like running:

    cd $P/build && make; cd -

Or, alternately:

    make -C $P/build

### Example 2

Given a makefile in the current directory:

    $P/makefile

Running `oosmake` from `$P` is like running `make` from the `$P`
directory.

### Example 3

Given a project with an out-of-source build directory, but the root
project directory has a makefile, too:

    $P/makefile
    $P/build/makefile

Running `oosmake` from `$P` is like running `make` from the `$P`
directory. In other words, `oosmake` ignores the out-of-source build
directory.

### Example 4

Given a project with subdirectories, with all makefiles in the
out-of-source build directory:

    $P/                         (no makefile in this directory)
    $P/alpha/                   (no makefile in this directory)
    $P/bravo/                   (no makefile in this directory)
    $P/build/makefile
    $P/build/alpha/makefile
    $P/build/bravo/             (no makefile in this directory)

Running `oosmake` from `$P` is like running:

    make -C $P/build

Running `oosmake` from `$P/alpha` is like running:

    make -C $P/build/alpha

Running `oosmake` from `$P/bravo` is like running:

    make -C $P/build
    
In other words, `oosmake` finds the most-nested out-of-source build
directory with a makefile.

### Example 5

`oosmake` gives priority to makefiles in a direct ancestor over
out-of-source makefiles.

    $P/makefile
    $P/alpha/                   (no makefile in this directory)
    $P/build/makefile
    $P/build/alpha/makefile

Running `oosmake` from `$P/alpha` is like running:

    make -C $P

## Features and limitations

* All command line options are passed as-are to the sub-`make` process.
  This includes `-C` and `-f`, which may mess up what you're trying to
  do.

* The root out-of-source build directory must be named `build`, or else
  any of the directories specified in the variable
  ${OOSMAKE_BUILD_DIRS}. Additional idiomatic patterns may be added in
  the future.

* There's no limit to the number of nested subdirectories. The recursive
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


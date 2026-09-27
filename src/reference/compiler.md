# Appendix: The Emojicode Compiler

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

This chapter is dedicated to the official Emojicode Compiler.

It is inteded to give more detail on certain options and by no
mean comprehensive. To obtain a full list of all command line options run
`emojicodec --help`.

## Compiling Files to an Executable Binary

The most obvious purpose of the compiler is the compilation of Emojicode source
files into a binary. The compiler expects the path to a single file, which
is the main file of the `_` package. E.g.

```sh
emojicodec hello.emojic
```

By default, the output file will have the same name as the main file. You
can explicitly set an output name with the `-o` option:

```sh
emojicodec aFile.emojic -o awesomeProgram
```

## Compiling a Package

To compile a package to a static library you have to specify a package name with
`-p`. E.g.

```sh
emojicodec -p catsimulator main.emojic
```

The output will have the name of your package, prefixed with `lib` and suffixed
with `.a`. You should not change the name of the output, but you can do so
with the `-o` option. The output file name must be suffixed with `.a`.

## Compiling to an Object File

You can tell the compiler to generate an object file instad of a static library
(archive) or exectuable with the `-c` flag. E.g

```
emojicodec -p catsimulator main.emojic -c
emojicodec hello.emojic -c
```

By default the output file will have the name of the main file with suffix `.o`
instead of `.emojic`. You can change the output path with `-o`.

## Compiling with Optimizations

By default, the compiler will compile your package with only some very basic
optimizations. To get out the most, you can turn on optimizations with the `-O`
flag. E.g

```
emojicodec -p catsimulator main.emojic -O
emojicodec hello.emojic -O
```

## Linking

To link an executable, the compiler runs the C++ compiler named by the
environment variable `CXX`, or `c++` if it is not set. Packages are archived
with `AR`, or `ar` if it is not set. C and Objective-C sources that link hints
name are compiled with `CC`, or `cc` if it is not set, and C++ sources with
`CXX`. The value of these variables is run by the shell, so a value like
`ccache c++` works too.

The linker is passed the libraries that the [link hints](packages.html#specifying-shared-libraries-to-link)
of the imported packages *and* of the main package ask for. If the linker or
archiver cannot be run or fails, the compiler reports an error like the one
below and exits with a non-zero status, so a failed link never goes unnoticed:

```text
ld: library 'doesnotexist' not found
clang++: error: linker command failed with exit code 1 (use -v to see invocation)
🚨 error: c++ failed with exit code 1.
```

## Package Search Paths

When you import a package, the compiler will search the package search paths for
the requested package. The search paths are:

1. `./packages` relative to the current directory.
2. Paths added with command line option `-S` in order of apperance from left to
  right.
3. Contents of the environment variable `EMOJICODE_PACKAGES_PATH` if set.
4. The default package search path, which is `/usr/local/EmojicodePackages/` but
  can be changed when building the compiler.

The compiler will look for directory named after the requested package in the
first package search path. If such a directory is found, the package is
considered found and an archive named as describe in “Compiling A Packge” is
expected. If such a directory is not found, the compiler tries with the next
package search paths. If search paths are exhausted an error is raised.

### Example

```sh
emojicodec test.emojic -S /opt/a -S /opt/b
```

If you run the above command in directory `/home/me/` with the environment
variable `EMOJICODE_PACKAGES_PATH` set to `/etc/packages` and `test.emojic`
imports a package called `dog` the compiler will look for it in the following
locations:

1. `/home/me/packages/dog`
2. `/opt/a/dog`
3. `/opt/b/dog`
4. `/etc/packages/dog`
5. `/usr/local/EmojicodePackages/dog`

## Switch the Compiler into JSON Mode

The option `--json` can be used to switch the compiler into JSON mode. When
working in JSON mode the compiler will print all errors and warnings to
standard error as a JSON array.

## Package Report

To generate a JSON report of the package the compiler compiled, you can pass the
`-r` option. These reports are used to build this documentation.

## Showing Messages in Color

The compiler colors its error messages and warnings when it prints them to a
terminal. Pass `--color` to use colors even when the output goes somewhere
else, for instance through a pipe into a pager.

## Writing the Interface File

When you compile a package, the compiler writes the package’s interface, a file
named `🏛`, next to the archive. The interface is what other packages read when
they import the package. You can choose another path with the `-i` option:

```sh
emojicodec -p catsimulator main.emojic -i catsimulator.emojii
```

## Emitting LLVM IR

With `--emit-llvm` the compiler generates code as usual, but writes the LLVM
intermediate representation to a file instead of producing an object file,
library or executable. The file is placed next to the main file and named
after it with the suffix `.ll`, e.g.

```sh
emojicodec hello.emojic --emit-llvm  # writes hello.ll
```

>!N Although `emojicodec --help` says the IR is printed to the standard output,
>!N it is written to the `.ll` file. The `-o` option doesn’t change its name.

## Checking Only the Syntax

`--parse-only` stops the compiler after it has parsed the main file, the files
it includes and the interfaces of the packages it imports. Only syntax errors
are reported, so this is a fast way to check whether the compiler can read your
code:

```sh
emojicodec --parse-only hello.emojic
```

The exit status is 0 if the file parsed and 1 otherwise.

## Printing the Tokens of a File

`--dump-tokens` prints the tokens into which the lexer divides the main file,
one per line, and exits without compiling anything. Each line contains the
position at which the token starts, counted in Unicode code points from the
start of the file, the kind of the token and the code points of the token’s
text in hexadecimal. For example, `🏁 🍇 😀 🔤Hey!🔤❗️ 🍉` is printed as:

```text
0	Identifier	1f3c1
2	BlockBegin	1f347
4	Identifier	1f600
6	String	48 65 79 21
12	EndOfArguments	2757
15	BlockEnd	1f349
```

Note that the U+FE0F after ❗ doesn’t appear in the token, as it is
[ignored](syntax.html#source-text). If the lexer finds an error, the last line
starts with `error` followed by the line and column of the error and the
message, and the compiler exits with status 1.

This is useful to find out how the compiler reads a particular piece of code,
e.g. where one emoji name ends and the next one begins.

## Formatting Source Code

`--format` parses the main file and rewrites it, and the files it includes, in
the standard layout with two spaces of indentation. Nothing is compiled.

```sh
emojicodec --format hello.emojic
```

>!N The formatter rewrites your files in place and does not keep everything
>!N about them: some comments, for instance a comment between two top-level
>!N declarations, are lost. Only use it on files that are under version
>!N control or that you have a copy of.

## Editor Support

The compiler comes with `emojicode-lsp`, a language server that uses the
compiler to show errors as you type and to provide hover information, go to
definition and completion in editors. See
[Editor Support](../guides/editors.html) for how to set it up.


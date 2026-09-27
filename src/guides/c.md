# Calling C with 🎍🌊

>!N **AI-generated:** This page was written by AI (Claude) from the source code and tests of this fork of
>!N Emojicode, and has not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

Emojicode can call C functions directly, pass C structs and C function
pointers back and forth, and let C code call functions written in Emojicode.
You declare the C side in Emojicode with the decorator 🎍🌊 and the types of
the `c` package. No glue code in C or C++ is needed: a whole library, such as
SQLite, can be wrapped in plain Emojicode.

>!H This guide assumes that you know C and that you are familiar with
>!H [Packages](../reference/packages.html) and
>!H [Safe and Unsafe Code](../reference/safety.html).

## 🎍🌊 Compared to the C++ API

Emojicode has two ways of running code that is not written in Emojicode.

- **🎍🌊 functions** use the plain C calling convention. A 🎍🌊 function is
  declared with exactly the parameters the C function has and nothing else,
  and it can only use types that exist in C: numbers, pointers, C structs and
  C function pointers. This makes it possible to call existing C libraries,
  like `libc`, `libm` or SQLite, without writing any C. It is also how C code
  calls into Emojicode: callbacks, exported functions and programs that embed
  Emojicode.
- **Methods implemented with 📻**, as described in
  [Foreign Function Interface and the C++ API](api.html), receive Emojicode
  values as they are represented at run time: objects, 🔡, callables, the
  class of a type method or the value a method is called on, and a raiser for
  errors. Such methods are usually implemented in C++ against the Emojicode
  run-time headers.

Use 🎍🌊 to bind a C library or to exchange plain data with C. Use the C++ API
when the native code itself must create or inspect Emojicode objects, raise
Emojicode errors, or back a *foreign class* whose storage cannot be described
in Emojicode, as the s package does for 🧵. Both kinds can live in the same
package, and a [link hint](#linking-c-code) can compile the C or C++ sources
of either.

## The C Package

The `c` package provides the C types that 🎍🌊 functions use. It is shipped
with the compiler. By convention it is imported into the namespace 🌊:

```
📦 c 🌊
```

The package contains:

- **C number types**, such as 🔶🌊🔢 for `int` and 🔶🌊📏 for `size_t`. They are
  listed [below](#c-types).
- **Pointers**: [📍](../packages/c/1f4cd.html)🐚T🍆 is C’s `T *`, and
  [🕳](../packages/c/1f573.html) is C’s `void *`. 📍 can load, store and
  advance, and 🕳 can carry an Emojicode object to C as *user data*.
- [🧶](../packages/c/1f9f6.html), which converts between 🔡 and C strings.
- [🔧](../packages/c/1f527.html), which declares `malloc`, `free`, `memcpy`
  and `strlen` as 🛒, 🗑, 📋 and 📏.

See the [API documentation of the c package](../packages/c/index.html) for
all methods. The package itself is written in Emojicode with 🎍🌊 and is a
good place to look for examples.

## Declaring C Functions

A C function is declared as a type method of a value type or an enumeration.
It is attributed with 🎍🌊, must be a type method (🐇) and must be unsafe (☣️).
Instead of a body, it names the C symbol after 📻:

```
📦 c 🌊

🕊 🧮 🍇
  🎍🌊 🐇☣️❗️ 🏧 value 🔶🌊🔢 ➡️ 🔶🌊🔢 📻 🔤abs🔤
  🎍🌊 🐇☣️❗️ 🌱 value 💯 ➡️ 💯 📻 🔤sqrt🔤
🍉

🏁 🍇
  ☣️ 🍇
    😀 🔤🧲🏧🕊🧮 -42❗️🧲🔤❗️
    😀 🔡 🌱🕊🧮 2.0❗️ 5❗️❗️
  🍉
🍉
```

This prints:

```
42
1.41421
```

These declarations correspond to `int abs(int)` and `double sqrt(double)`.
The value type 🧮 only serves as a namespace for the functions; it needs no
instances. Both functions are part of the C standard library, which every
Emojicode program is linked with. For any other library, add a
[link hint](#linking-c-code).

Because the compiler cannot check what a C function does, every 🎍🌊 function
is unsafe and can only be called inside a ☣️ block (or from another unsafe
function):

```!
📦 c 🌊

🕊 🧮 🍇
  🎍🌊 🐇☣️❗️ 🏧 value 🔶🌊🔢 ➡️ 🔶🌊🔢 📻 🔤abs🔤
🍉

🏁 🍇
  😀 🔤🧲🏧🕊🧮 -42❗️🧲🔤❗️  💭 Use of unsafe function 🏧 requires ☣️ block.
🍉
```

A good practice is to declare the raw C functions in one value type and to
wrap them in safe Emojicode classes and methods that contain the ☣️ blocks, as
the [SQLite example](#example-wrapping-sqlite) does.

### Rules for C Functions

A 🎍🌊 function is called with exactly the parameters it declares, using the C
calling convention: there is no hidden argument for the type or the callee.
The compiler enforces the following:

- It must be a type method (🐇) marked ☣️ of a value type or an enumeration.
  Classes and instance methods cannot declare C functions.
- It cannot be generic, and it cannot be declared in a generic type.
- It cannot be error-prone (🚧).
- Each parameter type and the return type must be a C type: a number type
  with a C representation, a pointer, an optional pointer, a C struct or a C
  function pointer. 🔡, classes, lists, ordinary callables and other Emojicode
  types cannot be passed to C.

```!
🕊 🧮 🍇
  🎍🌊 🐇☣️❗️ 📏 string 🔡 ➡️ 🔢 📻 🔤strlen🔤  💭 🔡 cannot be used in a function with 🎍🌊
🍉

🏁 🍇
🍉
```

The same C symbol may be declared several times, for instance in different
types, but all declarations must agree on its signature, including whether
its integers are signed.

## C Types

A value can be passed to C if its type has a *C representation*. The
following types have one:

| Emojicode type | C type |
|----------------|--------|
| 🔢 | `int64_t` |
| 💯 | `double` |
| 💧 | `int8_t` |
| 👌 | `bool` |
| 🔶🌊🔢 | `int` |
| 🔶🌊🔢🔸🔼 | `unsigned int` |
| 🔶🌊🐁 | `short` |
| 🔶🌊🐁🔸🔼 | `unsigned short` |
| 🔶🌊🐘 | `long` |
| 🔶🌊🐘🔸🔼 | `unsigned long` |
| 🔶🌊🦕 | `long long` |
| 🔶🌊🦕🔸🔼 | `unsigned long long` |
| 🔶🌊🔠 | `char` |
| 🔶🌊🔠🔸🔼 | `unsigned char` |
| 🔶🌊🔠🔸🔽 | `signed char` |
| 🔶🌊📏 | `size_t` |
| 🔶🌊📐 | `ssize_t` |
| 🔶🌊⏰ | `time_t` |
| 🔶🌊🎈 | `float` |
| 🔶🌊🕳 | `void *` |
| 🔶🌊📍🐚T🍆 | `T *` |

The first four types are the s package’s own types. All others come from the
`c` package, shown here imported into 🌊. The sizes are those of the C
compiler of the platform you compile on, so 🔶🌊🐘 has 64 bits on 64-bit Linux
and macOS.

C structs and C function pointers, which you declare yourself, are described
[below](#c-structs).

### Numbers

C number types behave like their C counterparts:

- Integer and real literals adopt the C type that is expected. The compiler
  warns if a literal does not fit into that type.
- The arithmetic, comparison, bitwise and shift operators work on two values
  of the same C type. Unsigned types compare, divide and compute remainders
  as unsigned numbers, and right shifts of signed types keep the sign.
- 🆕 with ▶️🔢 or ▶️💯 converts a 🔢 or 💯 to the type as a C cast would.
  The methods 🔢 and 💯 convert back.
- ❎ returns the bitwise complement, 🔋 the negation, and 🔡 the value as
  text. All C number types conform to ↘️🔸🔡, so they can be used in string
  interpolation.

```
📦 c 🌊

🏁 🍇
  🆕🔶🌊🔢🔸🔼▶️🔢 -1❗️ ➡️ max
  😀 🔤🧲max🧲 🧲max ➗ 2🧲🔤❗️

  🖍🆕 small 🔶🌊🐁
  1000 ➡️ 🖍small
  small ⬅️✖️ 3
  😀 🔤🧲small🧲 🧲🔢small❗️ ➕ 1🧲🔤❗️

  🆕🔶🌊🎈▶️💯 1.5❗️ ➡️ float
  😀 🔤🧲float ✖️ 3🧲 🧲💯float❗️🧲🔤❗️
🍉
```

```
4294967295 2147483647
3000 3001
4.500000 1.500000
```

C types are distinct from each other and from 🔢 and 💯. There are no
implicit conversions, so you must convert explicitly:

```!
📦 c 🌊

🏁 🍇
  🆕🔶🌊🔢▶️🔢 7❗️ ➡️ seven
  5 ➡️ five
  😀 🔤🧲seven ➕ five🧲🔤❗️  💭 five is a 🔢, not a 🔶🌊🔢
🍉
```

### Defining Your Own C Types

If you need a C type that the `c` package does not provide, such as
`uint32_t`, you can declare it. A primitive value type (📻) can name its C
representation in a string after 📻:

```
📻 🔤uint32_t🔤 🕊 🦆 🍇
  🆕 ▶️🔢 value 🔢 📻 🔤ejcBuiltIn🔤
  ❗️ 🔢 ➡️ 🔢 📻 🔤ejcBuiltIn🔤
🍉

🏁 🍇
  🆕🦆▶️🔢 -1❗️ ➡️ largest
  😀 🔡 🔢 largest❗️ 10❗️❗️
🍉
```

The compiler knows the types `char`, `signed char`, `unsigned char`, `short`,
`unsigned short`, `int`, `unsigned int`, `long`, `unsigned long`, `long long`,
`unsigned long long`, `size_t`, `ssize_t`, `ptrdiff_t`, `intptr_t`,
`uintptr_t`, `time_t`, `int8_t`, `int16_t`, `int32_t`, `int64_t`, `uint8_t`,
`uint16_t`, `uint32_t`, `uint64_t`, `bool`, `float`, `double` and `void *`.
Any other name is an error.

Such a type gets literals and operators automatically. Conversions and the
other built-in methods are only available if you declare them with
`📻 🔤ejcBuiltIn🔤` instead of a body:

- An initializer that takes one value of a type with a C representation is a
  conversion that works like a C cast, like ▶️🔢 and ▶️💯 in the `c` package.
- The methods 🔢 and 💯, which take no arguments, convert to 🔢 and 💯, ❎
  returns the bitwise complement and 🔋 the negation.

The types of the `c` package are declared exactly like this, so they are a
good template.

## Pointers

📍🐚T🍆 is a pointer to a value of type T, and 🕳 is a pointer without a type.
The most important methods are:

| Method | Meaning in C |
|--------|--------------|
| `🐽 pointer❗️` | `*pointer` |
| `value ➡️🐽 pointer❗️` | `*pointer = value` |
| `⏭ pointer count❗️` | `pointer + count` |
| `🕳 pointer❗️` | `(void *)pointer` |
| `🎭🐚U🍆 pointer❗️` | `(U *)pointer` |
| `📍🐚T🍆 voidPointer❗️` | `(T *)voidPointer` |
| `🔢 pointer❗️` | `(int64_t)pointer` |

Pointers can also be created from a numeric address (▶️🔢) or from the memory
of a 🧠 (▶️🧠). Creating, loading, storing and advancing are ☣️, while
converting between pointer types is not. The pointee type of a pointer you
load from or store through must be a concrete type, not a generic type
variable.

This example allocates an array of four `int`s with `malloc`, fills it and
frees it again:

```
📦 c 🌊

🏁 🍇
  ☣️ 🍇
    🍺🛒🕊🔶🌊🔧 🆕🔶🌊📏▶️🔢 4 ✖️ ⚖️🔶🌊🔢❗️❗️ ➡️ memory
    📍🐚🔶🌊🔢🍆memory❗️ ➡️ numbers
    🔂 i 🆕⏩ 0 4❗️ 🍇
      🆕🔶🌊🔢▶️🔢 i ✖️ 10❗️ ➡️🐽 ⏭numbers i❗️❗️
    🍉
    😀 🔤🧲🐽numbers❗️🧲 🧲🐽⏭numbers 3❗️❗️🧲🔤❗️
    🗑🕊🔶🌊🔧 memory❗️
  🍉
🍉
```

```
0 30
```

[⚖️](../reference/types.html#-size-of-type-instance) returns the size of a
type, like `sizeof` in C.

>!N Pointers are not managed. Neither the memory they point to nor the values
>!N stored there are retained or released, and nothing stops you from using a
>!N pointer to memory that has been freed. Memory from 🛒 must be freed with 🗑.

### Nullable Pointers

An optional pointer, like 🍬🔶🌊🕳 or 🍬🔶🌊📍🐚T🍆, is a pointer that can be
`NULL`: 🤷‍♀️ is `NULL`. Declare every parameter and return value that can be
`NULL` as optional, so that Emojicode makes you check it before use, and use
non-optional pointers only where C never passes or returns `NULL`.

### Out-Parameters

C functions often return a value through a pointer argument, as in
`int sqlite3_open(const char *path, sqlite3 **database)`. Allocate room for
the value, pass a pointer to it and read it back. These excerpts are from the
[SQLite example](#example-wrapping-sqlite), which declares the SQLite
functions in the value type 🗄. The `sqlite3 *` that is returned is a
🍬🔶🌊🕳, because it is an opaque pointer that may be `NULL`:

```
🎍🌊 🐇☣️❗️ 📂 path 🔶🌊📍🐚🔶🌊🔠🍆 database 🔶🌊📍🐚🍬🔶🌊🕳🍆 ➡️ 🔶🌊🔢 📻 🔤sqlite3_open🔤
```

```
🍺🛒🕊🔶🌊🔧 🆕🔶🌊📏▶️🔢 ⚖️🔶🌊🕳❗️❗️ ➡️ slot
📍🐚🍬🔶🌊🕳🍆slot❗️ ➡️ out
🧶🕊🔶🌊🧶 path 🍇 text 🔶🌊📍🐚🔶🌊🔠🍆 ➡️ 🔶🌊🔢
  ↩️ 📂🕊🗄 text out❗️
🍉❗️ ➡️ result
🐽out❗️ ➡️ database  💭 A 🍬🔶🌊🕳
🗑🕊🔶🌊🔧 slot❗️
```

## Strings

C strings are pointers to 🔶🌊🔠 that end with a 0 byte. 🧶 converts between
them and 🔡:

- `🔡🕊🔶🌊🧶 pointer❗️` copies a NUL-terminated C string into a new 🔡, and
  `🔡🕊🔶🌊🧶 pointer count❗️` copies *count* bytes. Both are ☣️ and expect
  UTF-8.
- `🧶🕊🔶🌊🧶 string closure❗️` calls the closure with a NUL-terminated copy of
  a 🔡 and returns what the closure returns. The pointer is only valid while
  the closure runs, so C must not keep it.

```
📦 c 🌊

🕊 🏡 🍇
  🎍🌊 🐇☣️❗️ 🔍 name 🔶🌊📍🐚🔶🌊🔠🍆 ➡️ 🍬🔶🌊📍🐚🔶🌊🔠🍆 📻 🔤getenv🔤
🍉

🏁 🍇
  ☣️ 🍇
    🧶🕊🔶🌊🧶 🔤HOME🔤 🍇 name 🔶🌊📍🐚🔶🌊🔠🍆 ➡️ 🔡
      ↪️ 🔍🕊🏡 name❗️ ➡️ value 🍇
        ↩️ 🔡🕊🔶🌊🧶 value❗️
      🍉
      ↩️ 🔤HOME is not set🔤
    🍉❗️ ➡️ home
    😀 home❗️
  🍉
🍉
```

`getenv` returns `NULL` for variables that are not set, so its return type is
the optional 🍬🔶🌊📍🐚🔶🌊🔠🍆.

## C Structs

A value type attributed with 🎍🌊 is a C struct. Its instance variables are its
fields, in the order they are declared, and they are laid out in memory as C
lays out the fields of a struct. Every field must have a C type, which
includes other C structs. A C struct cannot be generic.

Apart from that, a C struct is an ordinary value type: it can have
initializers, methods and protocol conformances. Unlike other value types, it
may have no initializer at all, since C structs are often only read from
memory that C filled.

C functions can take and return C structs by value, and pointers to C structs
work like any other pointer. Consider this C file, `vectors.c`:

```c
typedef struct { double x, y; } Vector;

Vector vectorScale(Vector vector, double factor) {
    return (Vector){ vector.x * factor, vector.y * factor };
}

void vectorFlip(Vector *vector) {
    double x = vector->x;
    vector->x = vector->y;
    vector->y = x;
}
```

The following program compiles and links `vectors.c` with a
[link hint](#linking-c-code), declares `Vector` as 🧭 and calls both
functions:

```
📦 c 🌊

🔗 🔤vectors.c🔤 🔗

🎍🌊 🕊 🧭 🍇
  🖍🆕 x 💯
  🖍🆕 y 💯

  🆕 🍼 x 💯 🍼 y 💯 🍇🍉

  ❗️ 🔡 ➡️ 🔡 🍇
    ↩️ 🔤(🧲🔡 x 1❗️🧲, 🧲🔡 y 1❗️🧲)🔤
  🍉
🍉

🕊 📐 🍇
  🎍🌊 🐇☣️❗️ 📏 vector 🧭 factor 💯 ➡️ 🧭 📻 🔤vectorScale🔤
  🎍🌊 🐇☣️❗️ 🔄 vector 🔶🌊📍🐚🧭🍆 📻 🔤vectorFlip🔤
🍉

🏁 🍇
  ☣️ 🍇
    📏🕊📐 🆕🧭 1.5 -2.0❗️ 2.0❗️ ➡️ scaled
    😀 🔡scaled❗️❗️

    🍺🛒🕊🔶🌊🔧 🆕🔶🌊📏▶️🔢 ⚖️🧭❗️❗️ ➡️ memory
    📍🐚🧭🍆memory❗️ ➡️ pointer
    scaled ➡️🐽 pointer❗️
    🔄🕊📐 pointer❗️
    😀 🔡🐽pointer❗️❗️❗️
    🗑🕊🔶🌊🔧 memory❗️
  🍉
🍉
```

```
(3.0, -4.0)
(-4.0, 3.0)
```

The Emojicode names of the struct and its fields do not matter to C. Only the
types and the order of the fields must match the C declaration.

### Passing Structs by Value

How a struct is passed by value differs from platform to platform. So that it
is always passed exactly as C passes it, the compiler calls a C function that
takes or returns a struct by value through a small *trampoline* written in C.
The compiler writes the trampolines to a file ending in `_trampolines.c` next
to the object file it creates and compiles it with the C compiler (`$CC`, or
`cc` if it is not set). A C compiler must therefore be installed to compile
such calls.

Trampolines only exist for calls from Emojicode to C functions that are
declared with a symbol name. C function pointers and
[C functions written in Emojicode](#writing-c-functions-in-emojicode) cannot
take or return structs by value and must pass them through a 📍 instead.

## Callbacks

A callable type written with 🎍🌊 after its 🍇 is a C function pointer.
For example, the comparator that `qsort` takes,
`int (*compar)(const void *, const void *)`, is written
🍇🎍🌊🔶🌊🕳🔶🌊🕳➡️🔶🌊🔢🍉. The parameter and return types of a C function
pointer must be C types, structs can only be passed by 📍, and a C function
pointer cannot raise errors.

C function pointers are plain pointers: they are not reference counted and
cannot be used where an ordinary callable is expected, or vice versa. An
optional C function pointer can be `NULL`.

Calling a C function pointer, for instance one that a C function returned, is
unsafe and requires a ☣️ block:

```!
🏁 🍇
  🍇🎍🌊 value 🔢 ➡️ 🔢
    ↩️ value ➕ 1
  🍉 ➡️ callback
  ⁉️ callback 1❗️ ➡️ result  💭 Calling a C function pointer requires a ☣️ block.
🍉
```

### C Closures

A closure written with 🎍🌊 after its 🍇 is compiled to a C function, and its
value is a C function pointer to it, which can be passed to C:

```
📦 c 🌊

🕊 📚 🍇
  🎍🌊 🐇☣️❗️ 🔀 base 🔶🌊🕳 count 🔶🌊📏 size 🔶🌊📏 compare 🍇🎍🌊🔶🌊🕳🔶🌊🕳➡️🔶🌊🔢🍉 📻 🔤qsort🔤
🍉

🏁 🍇
  🍿 5 -3 42 0 17 🍆 ➡️ values
  📏values❓ ➡️ count
  ☣️ 🍇
    🍺🛒🕊🔶🌊🔧 🆕🔶🌊📏▶️🔢 count ✖️ ⚖️🔶🌊🔢❗️❗️ ➡️ memory
    📍🐚🔶🌊🔢🍆memory❗️ ➡️ numbers
    🔂 i 🆕⏩ 0 count❗️ 🍇
      🆕🔶🌊🔢▶️🔢 🐽values i❗️❗️ ➡️🐽 ⏭numbers i❗️❗️
    🍉

    🔀🕊📚 memory 🆕🔶🌊📏▶️🔢 count❗️ 🆕🔶🌊📏▶️🔢 ⚖️🔶🌊🔢❗️ 🍇🎍🌊 a 🔶🌊🕳 b 🔶🌊🕳 ➡️ 🔶🌊🔢
      🐽📍🐚🔶🌊🔢🍆a❗️❗️ ➡️ left
      🐽📍🐚🔶🌊🔢🍆b❗️❗️ ➡️ right
      ↪️ left ◀️ right 🍇 ↩️ -1 🍉
      ↪️ left ▶️ right 🍇 ↩️ 1 🍉
      ↩️ 0
    🍉❗️

    🔂 i 🆕⏩ 0 count❗️ 🍇
      😀 🔤🧲🐽⏭numbers i❗️❗️🧲🔤❗️
    🍉
    🗑🕊🔶🌊🔧 memory❗️
  🍉
🍉
```

A closure written inside a ☣️ block, like the comparator above, may use unsafe
functions without a ☣️ block of its own.

A C function has no place to store captured values, so a C closure must be
self-contained:

- It cannot capture variables or use 👇.
- It cannot use the generic type variables of the method it is written in.
- It cannot be 🎍🥡 or raise errors.

```!
🏁 🍇
  5 ➡️ offset
  🍇🎍🌊 value 🔢 ➡️ 🔢
    ↩️ value ➕ offset  💭 A closure with 🎍🌊 cannot capture variables or 👇.
  🍉 ➡️ callback
🍉
```

Instead, C APIs that take a callback usually also take a `void *` that they
pass to it, often called *user data* or *context*. That is how a callback
gets at state, as the next section shows.

### Passing Objects as User Data

🕳 can carry an Emojicode object through C:

- `🆕🔶🌊🕳▶️📤 object❗️` retains the object and returns its address.
- `👀🐚T🍆 pointer❗️` returns the object without affecting its reference
  count. Use it in callbacks, which may run many times.
- `📥🐚T🍆 pointer❗️` returns the object and releases the reference that ▶️📤
  retained. The pointer must not be used afterwards.

All three are ☣️. Balance every ▶️📤 with exactly one 📥 once C no longer
uses the pointer, or the object is leaked. T must be the class of the object
that was passed to ▶️📤.

The C function `eachSquare` in this file, `squares.c`, calls a callback with
the squares of 1 to *count*:

```c
void eachSquare(int count, void (*visit)(int square, void *userData), void *userData) {
    for (int i = 1; i <= count; i++) {
        visit(i * i, userData);
    }
}
```

This program passes a 🧺 as user data and adds the squares up in it:

```
📦 c 🌊

🔗 🔤squares.c🔤 🔗

🐇 🧺 🍇
  🖍🆕 total 🔢 ⬅️ 0

  🆕 🍇🍉

  ❗️ 🧮 value 🔢 🍇
    total ⬅️➕ value
  🍉

  ❗️ 🔢 ➡️ 🔢 🍇
    ↩️ total
  🍉
🍉

🕊 📚 🍇
  🎍🌊 🐇☣️❗️ 🔄 count 🔶🌊🔢 visit 🍇🎍🌊🔶🌊🔢🔶🌊🕳🍉 userData 🔶🌊🕳 📻 🔤eachSquare🔤
🍉

🏁 🍇
  🆕🧺❗️ ➡️ basket
  ☣️ 🍇
    🆕🔶🌊🕳▶️📤 basket❗️ ➡️ userData
    🔄🕊📚 4 🍇🎍🌊 square 🔶🌊🔢 data 🔶🌊🕳
      🧮 👀🐚🧺🍆data❗️ 🔢square❗️❗️
    🍉 userData❗️
    📥🐚🧺🍆userData❗️ ➡️ taken
  🍉
  😀 🔤The squares add up to 🧲🔢basket❗️🧲🔤❗️
🍉
```

```
The squares add up to 30
```

## Writing C Functions in Emojicode

A 🎍🌊 function whose symbol name is followed by a body is defined in
Emojicode and exported under that name with the C calling convention. C code
can call it like any other C function:

```
🎍🌊 🐇☣️❗️ 🔢 text 🔶🌊📍🐚🔶🌊🔠🍆 ➡️ 🔶🌊🔢 📻 🔤wordcount_count🔤 🍇
  💭 ...
🍉
```

This is the C function `int wordcount_count(const char *text)`. The same
rules as for declared C functions apply, and in addition:

- An exported function cannot take or return a C struct by value. Pass a 📍 to
  the struct instead.
- Each symbol can only be exported once.
- Only 🎍🌊 functions can be exported. A method with both 📻 and a body is an
  error.

Exported functions are listed in the interface of a package like other C
functions, so that packages importing it can call them too. Like all 🎍🌊
functions, they are ☣️ when called from Emojicode.

C code that is compiled into the same program, for example through a
[link hint](#linking-c-code), can call exported functions directly.

## Embedding Emojicode in a C Program

When the compiler links an Emojicode program, the run-time library provides
the C `main` function. It initializes the run time and then runs the 🏁 block.

A C or C++ program can instead provide `main` itself and call into an
Emojicode package through its exported functions. It must call `ejcInit`,
which the run-time library provides, before any Emojicode code runs:

```c
void ejcInit(int argc, char **argv);
```

The 🏁 block of the package, if it has one, is not run.

Here is a package, `wordcount.emojic`, that exports two functions:

```
📦 c 🌊

🕊 📤 🍇
  🎍🌊 🐇☣️❗️ 🔢 text 🔶🌊📍🐚🔶🌊🔠🍆 ➡️ 🔶🌊🔢 📻 🔤wordcount_count🔤 🍇
    0 ➡️ 🖍🆕 words
    🔂 word 🔫 🔡🕊🔶🌊🧶 text❗️ 🔤 🔤❗️ 🍇
      ↪️ 📐word❗️ ▶️ 0 🍇
        words ⬅️➕ 1
      🍉
    🍉
    ↩️ 🆕🔶🌊🔢▶️🔢 words❗️
  🍉

  🎍🌊 🐇☣️❗️ 📢 text 🔶🌊📍🐚🔶🌊🔠🍆 📻 🔤wordcount_shout🔤 🍇
    😀 📫 🔡🕊🔶🌊🧶 text❗️❗️❗️
  🍉
🍉
```

And here is the C program, `main.c`, that uses it:

```c
#include <stdio.h>

void ejcInit(int argc, char **argv);
int wordcount_count(const char *text);
void wordcount_shout(const char *text);

int main(int argc, char **argv) {
    ejcInit(argc, argv);
    printf("%d words\n", wordcount_count("the quick  brown fox"));
    fflush(stdout);
    wordcount_shout("hello from C");
    return 0;
}
```

Compile the package to an archive, then compile the C program and link it
with the C++ compiler, because the run-time library is written in C++. Link
the package first, then the packages it imports, then the s package and the
run-time library, which are in the directory that the compiler searches for
packages (for example `/usr/local/EmojicodePackages`):

```bash
emojicodec -p wordcount -o libwordcount.a wordcount.emojic -O
cc -c main.c -o main.o
P=/usr/local/EmojicodePackages
c++ main.o libwordcount.a $P/c/libc.a $P/s/libs.a $P/runtime/libruntime.a -lm -lpthread -o main
./main
```

```
4 words
HELLO FROM C
```

The package’s own link hints are not applied when you link this way, so pass
the libraries it needs to the linker yourself. Alternatively, `-c` produces a
single object file, which also contains the objects of the package’s native
sources and trampolines:

```bash
emojicodec -p wordcount -c wordcount.emojic -o wordcount.o -O
```

## Linking C Code

The code of a C library must be linked into the program. Link hints (🔗),
which are described in
[Specifying Shared Libraries to Link](../reference/packages.html#specifying-shared-libraries-to-link),
tell the compiler what to link. Each hint can be:

- The name of a library, like `🔤sqlite3🔤`, which is linked with `-lsqlite3`.
- Linker flags, if the hint starts with `-`, like `🔤-framework Foundation🔤`
  or `🔤-L/opt/homebrew/lib🔤`.
- A C, C++ or Objective-C source file (`.c`, `.cc`, `.cpp`, `.cxx` or `.m`)
  or an object file (`.o`), relative to the file that contains the hint. The
  compiler compiles the source and links it into the program, or adds it to
  the archive of a package.

```
🔗 🔤sqlite3🔤 🔤helpers.c🔤 🔗
```

Link hints work in programs as well as in packages. The hints of an imported
package are applied when a program that imports it is linked, except for
source and object files, which are already part of the package’s archive.

## Memory and Safety

Everything that crosses into C is outside of Emojicode’s memory management
and type checking. That is why all 🎍🌊 functions, calls through C function
pointers and most pointer operations are ☣️. Keep these rules in mind:

- 📍 and 🕳 do not retain or release what they point to. You are responsible
  for freeing memory you allocated with 🛒, and for not using memory after it
  was freed.
- A pointer made from a 🧠 (▶️🧠) or passed to the closure of 🧶 is only valid
  as long as the 🧠 or the call exists. C must not keep it.
- Every object passed to C with 🕳 ▶️📤 is retained until it is taken back with
  📥. Use 👀 in callbacks, and call 📥 exactly once.
- C structs contain only C values, so copying one copies plain memory.
- Declare pointers that can be `NULL` as optional.
- C function pointers and C closures are not reference counted. C closures
  capture nothing, so there is nothing to release.

Wrap C functions in safe types so that the rest of your code does not need
☣️ blocks.

## Example: Wrapping SQLite

The directory `examples/sqlite` of the compiler’s source code contains a
package that wraps the SQLite C library using only Emojicode, 🎍🌊 and the `c`
package, and a program that uses it. It shows most of what this guide
describes. Its build script, `build.sh`, builds the package, builds and runs
the program and compares the output with the expected output.

The package links SQLite with a link hint and declares the C functions it uses
in a value type, exactly as `sqlite3.h` declares them. The opaque `sqlite3 *`
and `sqlite3_stmt *` are 🕳:

```
📦 c 🌊

🔗 🔤sqlite3🔤 🔗

🕊 🗄 🍇
  🎍🌊 🐇☣️❗️ 📂 path 🔶🌊📍🐚🔶🌊🔠🍆 database 🔶🌊📍🐚🍬🔶🌊🕳🍆 ➡️ 🔶🌊🔢 📻 🔤sqlite3_open🔤
  🎍🌊 🐇☣️❗️ 🚪 database 🔶🌊🕳 ➡️ 🔶🌊🔢 📻 🔤sqlite3_close_v2🔤
  🎍🌊 🐇☣️❗️ 🗯 database 🔶🌊🕳 ➡️ 🔶🌊📍🐚🔶🌊🔠🍆 📻 🔤sqlite3_errmsg🔤
  🎍🌊 🐇☣️❗️ 🏃 database 🔶🌊🕳 sql 🔶🌊📍🐚🔶🌊🔠🍆
    callback 🍇🎍🌊🔶🌊🕳 🔶🌊🔢 🔶🌊📍🐚🍬🔶🌊📍🐚🔶🌊🔠🍆🍆 🔶🌊📍🐚🔶🌊📍🐚🔶🌊🔠🍆🍆➡️🔶🌊🔢🍉
    userData 🔶🌊🕳 errorMessage 🍬🔶🌊🕳 ➡️ 🔶🌊🔢 📻 🔤sqlite3_exec🔤
  💭 ...
🍉
```

The exported class 🗃 owns a database connection. Its initializer opens the
database with the [out-parameter](#out-parameters) shown earlier and raises
an error if that fails, and its deinitializer closes the connection, so users
of the package never see a pointer or a ☣️ block:

```
🌍 🐇 🗃 🍇
  🖍🆕 handle 🔶🌊🕳

  🆕 path 🔡 🚧💥 🍇
    💭 Calls sqlite3_open and raises 💥 if it fails.
  🍉

  ♻️ 🍇
    ☣️ 🍇
      🚪🕊🗄 handle❗️
    🍉
  🍉
🍉
```

To run a query with `sqlite3_exec`, whose callback is called once per row,
🗃 passes an object holding the Emojicode closure as user data. The C closure
turns the C strings of each row into a dictionary and hands it to the object
it gets back with 👀. After `sqlite3_exec` returns, 📥 releases the object:

```
🆕📬 row❗️ ➡️ mailbox
☣️ 🍇
  🆕🔶🌊🕳▶️📤 mailbox❗️ ➡️ userData
  🧶🕊🔶🌊🧶 sql 🍇 text 🔶🌊📍🐚🔶🌊🔠🍆 ➡️ 🔶🌊🔢
    ↩️ 🏃🕊🗄 handle text 🍇🎍🌊 userData 🔶🌊🕳 count 🔶🌊🔢
        values 🔶🌊📍🐚🍬🔶🌊📍🐚🔶🌊🔠🍆🍆 names 🔶🌊📍🐚🔶🌊📍🐚🔶🌊🔠🍆🍆 ➡️ 🔶🌊🔢
      🆕🍯🐚💎🍆❗️ ➡️ 🖍🆕 row
      🔂 i 🆕⏩ 0 🔢count❗️❗️ 🍇
        🔡🕊🔶🌊🧶 🐽⏭names i❗️❗️❗️ ➡️ name
        ↪️ 🐽⏭values i❗️❗️ ➡️ value 🍇
          🆕💎▶️🔡 🔡🕊🔶🌊🧶 value❗️❗️ ➡️🐽 row name❗️
        🍉
        🙅 🍇
          🆕💎▶️🈳❗️ ➡️🐽 row name❗️
        🍉
      🍉
      📨 👀🐚📬🍆userData❗️ row❗️
      ↩️ 0
    🍉 userData 🤷‍♀️❗️
  🍉❗️ ➡️ result
  📥🐚📬🍆userData❗️ ➡️ taken
  💭 Raises 💥 if result is not 0.
🍉
```

A program that imports the package works with SQLite without any unsafe
code:

```
📦 sqlite 🏠

🏁 🍇
  🆗 database 🆕🗃 🔤:memory:🔤❗️ 🍇
    🆗 🔨 database 🔤SELECT * FROM nope🔤❗️ 🍇🍉
    🙅 error 🍇
      😀 🔤SQLite error 🧲🔢error❗️🧲: 🧲💬error❗️🧲🔤❗️
    🍉
  🍉
  🙅 error 🍇
    😀 🔤Could not open the database🔤❗️
  🍉
🍉
```

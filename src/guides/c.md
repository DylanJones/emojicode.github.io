# Calling C with 🎍🌊

Emojicode can call C functions directly, pass C structs and pointers, receive
callbacks from C, and export functions for C programs to call. No C or C++
code is needed for any of this: a C library is wrapped in Emojicode alone.

>!N Everything that touches C memory is unsafe. Functions that call C must be
>!N marked ☣️, and their calls must be in a ☣️ block. Wrap C libraries in
>!N classes that offer a safe interface, like in the example at the end of
>!N this guide.

## The c Package

The types C uses are provided by the `c` package. Import it into the namespace
🌊, which is the convention:

```
📦 c 🌊
```

It provides these types, all used as `🔶🌊` followed by the name:

| Emojicode | C | Emojicode | C |
|---|---|---|---|
| 🔢 | `int` | 🔢🔸🔼 | `unsigned int` |
| 🐁 | `short` | 🐁🔸🔼 | `unsigned short` |
| 🐘 | `long` | 🐘🔸🔼 | `unsigned long` |
| 🦕 | `long long` | 🦕🔸🔼 | `unsigned long long` |
| 🔠 | `char` | 🔠🔸🔼, 🔠🔸🔽 | `unsigned char`, `signed char` |
| 📏 | `size_t` | 📐 | `ssize_t` |
| ⏰ | `time_t` | 🎈 | `float` |
| 🕳 | `void *` | 📍🐚T🍆 | `T *` |

The types of s that C also has can be used directly: 💯 is `double`, 👌 is
`bool`, 🔢 is `int64_t` and 💧 is `int8_t`.

Integer and real literals can be used as values of these types:

```
🆕🔶🌊🔢▶️🔢 7❗️ ➡️ seven  💭 converts a 🔢 to a C int
😀 🔤🧲seven ➗ 2🧲🔤❗️
🔢seven❗️ ➡️ integer  💭 converts back to 🔢
```

Arithmetic and comparisons follow C: unsigned types divide, compare and shift
as unsigned values, and conversions wrap around like C conversions.

## Declaring C Functions

A C function is declared as a type method of a value type or an enumeration,
marked 🎍🌊 and ☣️. After `📻` follows the name of the C function:

```
📦 c 🌊

🕊 🧮 🍇
  🎍🌊 🐇☣️❗️ 🏧 value 🔶🌊🔢 ➡️ 🔶🌊🔢 📻 🔤abs🔤
  🎍🌊 🐇☣️❗️ 🌱 value 💯 ➡️ 💯 📻 🔤sqrt🔤
🍉

🏁 🍇
  ☣️ 🍇
    😀 🔤🧲🏧🕊🧮 -42❗️🧲🔤❗️
  🍉
🍉
```

The function is called exactly as C would call it: no hidden arguments are
passed, and narrow integers are extended as the C calling convention requires.
Only C types can appear in the signature, and a C function cannot raise
errors or be generic.

Libraries other than the C standard library must be linked with a link hint,
like `🔗 🔤sqlite3🔤 🔗`. See
[Specifying Shared Libraries to Link](../reference/packages.html#specifying-shared-libraries-to-link).

## Pointers

📍🐚T🍆 is a C pointer to T, and 🕳 is a C `void *`. They are not managed:
Emojicode neither retains nor releases what they point to. Pointers that can be
`NULL` are optionals, 🍬📍🐚T🍆 and 🍬🕳.

Pointers have these methods:

- `🐽 pointer❗️` reads the value, `value ➡️🐽 pointer❗️` writes it.
- `⏭ pointer count❗️` is `pointer + count` in C.
- `🕳 pointer❗️` and `📍🐚T🍆 voidPointer❗️` convert between the two pointer types,
  `🎭🐚U🍆 pointer❗️` casts to a pointer to U.
- `🆕🔶🌊🕳▶️🧠 memory❗️` is the address of the bytes of a 🧠, valid while
  the 🧠 is alive. `🆕🔶🌊🕳▶️🔢 address❗️` and `🔢 pointer❗️` convert to and
  from numeric addresses.

`🛒🕊🔶🌊🔧 size❗️` (`malloc`) and `🗑🕊🔶🌊🔧 pointer❗️` (`free`) allocate C
memory, for example for out-parameters:

```
🍺🛒🕊🔶🌊🔧 🆕🔶🌊📏▶️🔢 ⚖️🔶🌊🕳❗️❗️ ➡️ slot
📍🐚🍬🔶🌊🕳🍆slot❗️ ➡️ out   💭 a sqlite3 ** for sqlite3_open
```

## Strings

C strings are pointers to 🔠 that end with a zero byte. 🔡 is not
zero-terminated, so strings are converted:

```
🧶🕊🔶🌊🧶 🔤hello🔤 🍇 string 🔶🌊📍🐚🔶🌊🔠🍆 ➡️ 🔶🌊📏
  ↩️ 📏🕊🔶🌊🔧 string❗️  💭 strlen
🍉❗️ ➡️ length
```

🧶 calls the closure with a zero-terminated copy of the string, which is only
valid while the closure runs. `🔡🕊🔶🌊🧶 pointer❗️` copies a C string into a new
🔡, and `🔡🕊🔶🌊🧶 pointer count❗️` copies *count* bytes.

## Structs

A value type marked 🎍🌊 is a C struct. Its instance variables are its fields,
in order, and must have C types:

```
🎍🌊 🕊 📌 🍇
  🖍🆕 x 🔶🌊🔢
  🖍🆕 y 🔶🌊🔢

  🆕 🍼 x 🔶🌊🔢 🍼 y 🔶🌊🔢 🍇🍉
🍉
```

Structs can be passed by pointer and by value. Functions that take or return
structs by value are called through small C functions that the compiler
generates and compiles with the C compiler (`$CC`, or `cc`), so structs are
passed exactly as C passes them.

## Callbacks

`🍇🎍🌊 … 🍉` is the type of a C function pointer. A closure written
`🍇🎍🌊 …` is a C function; it cannot capture variables or 👇:

```
🎍🌊 🐇☣️❗️ 🔀 base 🔶🌊🕳 count 🔶🌊📏 size 🔶🌊📏
  compare 🍇🎍🌊🔶🌊🕳🔶🌊🕳➡️🔶🌊🔢🍉 📻 🔤qsort🔤
```

To give a callback access to Emojicode objects, pass an object as the
callback's user data. `🆕🔶🌊🕳▶️📤 object❗️` retains the object and returns its
address. In the callback, `👀🐚T🍆 userData❗️` returns the object, and
`📥🐚T🍆 userData❗️` returns it and gives up the reference, which balances ▶️📤.

## Calling Emojicode from C

A C function with a body after its name is written in Emojicode and exported
under that name:

```
🕊 🧩 🍇
  🎍🌊 🐇☣️❗️ 🧮 a 🔶🌊🔢 b 🔶🌊🔢 ➡️ 🔶🌊🔢 📻 🔤add🔤 🍇
    ↩️ a ➕ b
  🍉
🍉
```

A C program can link the package (built with `emojicodec -p`), `s`, `c` and
the runtime library, and call `add`. It has its own `main` function and must
call `ejcInit(argc, argv)` before it calls into Emojicode.

## Example: SQLite

The Emojicode repository (this fork) contains
[a wrapper around SQLite](https://github.com/DylanJones/emojicode/tree/master/examples/sqlite)
written in Emojicode with the features of this guide, and a program that runs
SQL queries with it.

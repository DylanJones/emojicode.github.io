# Safe and Unsafe Code

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

Emojicode is designed to make your programs safe.

But what exactly does this mean? In contrast to other programming languages,
your program will never terminate without stating a clear reason and should do
so only in a few predictable and avoidable cases. These cases are called a panic.

To achieve this goal Emojicode imposes certain restrictions. For instance,
you can normally not allocate memory or perform random operations on allocated
memory.

## Panic

When an Emojicode program reaches a state in which it cannot continue execution
because an irrecoverable error has arosen, it will panic. Panicking occurs
in the following situations.

- Unwrapping an optional without a value or an error that contains an error
- Accessing an array out of bounds
- Calling 🔽❗️ on an iterator of a list or range that has no more values
- A call to the panic method 🤯
- The program runs out of memory (rare)

### Panicking Yourself

You can make your program panic by calling the type method 🤯 of 💻 with a
message that explains what went wrong. The compiler knows that 🤯 never returns.
Like ↩️, it ends a block, so a method that returns a value can end with 🤯
without returning anything afterwards, and code after it is reported as never
executed:

```
🕊 🧰 🍇
  🐇❗️ 🌡 celsius 🔢 ➡️ 🔡 🍇
    ↪️ celsius ◀️ 0 🍇
      ↩️ 🔤freezing🔤
    🍉
    🙅↪️ celsius ◀️ 100 🍇
      ↩️ 🔤liquid🔤
    🍉
    🤯🐇💻 🔤Water should not be this hot🔤❗️
  🍉
🍉

🏁 🍇
  😀 🌡🕊🧰 20❗️❗️  💭 Prints liquid
  😀 🌡🕊🧰 120❗️❗️  💭 Panics
🍉
```

## Unsafe Code

In some cases you might actually need to allocate memory yourself.
The s package provides a value type named 🧠, with which you can do exactly
that.

```
🌍 🕊 🧠 🍇
  ☣️️ 🆕 size 🔢 🍇🍉
  ☣️️ ❗️ 🐷🐚☣️️T⚪️🍆 value T offset 🔢 📻 🔤ejcBuiltIn🔤
  💭 ...
🍉
```

Nonetheless, these methods cannot be used by default as they are very dangerous
when misused. You might have noted that the method and initializer shown in the
above example use the attribute ☣️️.

The ☣️️ attribute marks these functions as unsafe. You can only use unsafe
functions within an unsafe block or within another unsafe function or you will
get a compiler error.

```syntax
$unsafe-block$-> ☣️️ $block$
```

So, for instance, to allocate a memory block of 10 bytes we can write this code:

```
☣️ 🍇
  🆕🧠 10❗
🍉
```

If we hadn’t wrapped the initialization expression into an unsafe block
we would get a compiler error.

Note that the unsafe block does not create its own variable scope. It is just
syntactic sugar and does not affect flow control. We therefore recommend to keep
all code that does not need to go into an unsafe block outside.

>!H You should take that biohazard sign quite seriously. Messing with memory
>!H can go terribly wrong.

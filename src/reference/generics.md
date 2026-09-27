# Generics

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

Generics allow you to write code in which you can use a placeholder – variable
names – instead of an actual type, which will then be substituted with real
types when you use that code later. This is a really powerful feature and is a
great way to avoid code duplication.

## Defining a Generic Type

To define a generic class or value type, provide generic parameters after
the name of the type. A generic parameter consists of a name, which can then be
used instead of a type inside the class or value type, and a type constraint.

```syntax
$generic-parameter$-> $variable$ $type$
$generic-parameter-list$-> $generic-parameter$ [$generic-parameter-list$]
$generic-parameters$-> 🐚 $generic-parameter-list$ 🍆
```

See this example for a box type that can store objects of a specified type. Note
that inside the class body `T` is used as a type.

```
🐇 🎁 🐚T🔵🍆 🍇
  🖍🆕 something T

  🆕 ✂️ 🍼 something T 🍇🍉

  ❗️ 🎉 ➡️  T 🍇
    ↩️ something
  🍉
🍉
```

The following example demonstrates how to instantiate a generic class:

```
🆕🎁🐚🔡🍆✂️ 🔤Been wishin' for you🔤❗
```

### Type Constraint

The type constraint constrains which types can be supplied as an arguments for
a generic type parameter. Type constraints are useful as they allow you to
treat values of a generic type as if they were an instance of the type
constraint.

A constraint can mention the generic parameters of the same declaration,
including the parameter it constrains. This is commonly used with generic
protocols: `T 🧩🐚T🍆` means “T must be a 🧩 of itself”. The protocol can even be
constrained to itself in its own declaration:

```
🐊 🧩🐚T 🧩🐚T🍆🍆 🍇
  ❗️ 🔗 other T ➡️ T
🍉

🕊 📜 🍇
  🐊 🧩🐚📜🍆
  🖍🆕 text 🔡

  🆕 🍼 text 🔡 🍇🍉

  ❗️ 🔗 other 📜 ➡️ 📜 🍇
    ↩️ 🆕📜 🔤🧲text🧲 🧲📝 other❗️🧲🔤❗️
  🍉

  ❗️ 📝 ➡️ 🔡 🍇
    ↩️ text
  🍉
🍉

🕊 🧰 🍇
  🐇❗️ 🔗🐚T 🧩🐚T🍆🍆 parts 🍨🐚T🍆 ➡️ T 🍇
    🐽 parts 0❗️ ➡️ 🖍🆕 result
    🔂 i 🆕⏩ 1 📏parts❓❗️ 🍇
      🔗 result 🐽 parts i❗️❗️ ➡️ 🖍result
    🍉
    ↩️ result
  🍉
🍉

🏁 🍇
  🍿 🆕📜 🔤Been🔤❗️ 🆕📜 🔤wishin'🔤❗️ 🆕📜 🔤for you🔤❗️ 🍆 ➡️ words
  😀 📝 🔗🕊🧰 words❗️❗️❗️  💭 Prints “Been wishin' for you”
🍉
```

Because of its constraint, `🔗` can call 🔗 on values of `T` and knows the result
is a `T` again. Calling it with a type that does not conform, like
`🔗🕊🧰 🍿 1 2 🍆❗️`, is a compiler error, as 🔢 is no 🧩🐚🔢🍆.

A constraint may also mention another generic parameter, as in
`🐚A ⚪ B 🍨🐚A🍆🍆`, where `B` must be a list of whatever `A` is.

The s package’s [📈](#numbers-in-generic-code) protocol is declared this way.

### Numbers in Generic Code

🔢, 💯 and 💧 conform to the s package’s 📈 protocol with themselves as
generic argument, i.e. 🔢 is a `📈🐚🔢🍆`. 📈 requires the arithmetic operators
➕, ➖, ✖️, ➗ and 🚮 and the comparisons ◀️, ▶️, ◀️🙌 and ▶️🙌. Code that
constrains a generic parameter to `📈` of itself can therefore compute with
any of these number types:

```
🕊 🧮 🍇
  🐇❗️ 📐🐚T 📈🐚T🍆🍆 a T b T ➡️ T 🍇
    ↩️ a ✖️ a ➕ b ✖️ b
  🍉

  🐇❗️ 🔝🐚T 📈🐚T🍆🍆 values 🍨🐚T🍆 ➡️ T 🍇
    🐽values 0❗️ ➡️ 🖍🆕 largest
    🔂 value values 🍇
      ↪️ value ▶️ largest 🍇
        value ➡️ 🖍largest
      🍉
    🍉
    ↩️ largest
  🍉
🍉

🏁 🍇
  😀 🔡 📐🕊🧮 3 4❗️❗️❗️  💭 25
  😀 🔡 📐🕊🧮 1.5 2.0❗️ 2❗️❗️  💭 6.25
  😀 🔡 🔝🕊🧮 🍿 3 9 4 🍆❗️❗️❗️  💭 9
🍉
```

Generic code can pass its own generic parameters on as generic arguments,
explicitly or inferred, to other generic methods and types, as long as they
satisfy the constraints there. Inside a method generic over `U 📈🐚U🍆`, for
instance, both `📐🐚U🍆🕊🧮 a b❗️` and `📐🕊🧮 a b❗️` call 📐 with `U` for `T`.

## Subclassing a Generic Class

Naturally you can subclass a generic class. Like in any other circumstance you
have to provide values for the superclass’s generic parameters. For instance:

```
🐇 ☑️ 🎁🐚🔡🍆 🍇

🍉
```

If the subclass itself takes a generic argument this argument can be used as
argument for the superclass:

```
🐇 🌟🐚A🔵🍆 🎁🐚A🍆 🍇

🍉
```

## Compatibility

Two generic types are only compatible if they were provided with exactly the
same arguments. So `🍨🐚🔡🍆` is only compatible to `🍨🐚🔡🍆` but not to
`🍨🐚⚪️🍆` as one might expect.

## Optional Generic Arguments

An optional can be a generic argument, as in `🍯🐚🍬🔢🍆`. Optionals do not nest,
though: where the generic code uses `🍬T` and `T` is `🍬🔢`, the type is simply
`🍬🔢`. For instance, 🍯’s 🐽 returns `🍬Element`, which is `🍬🔢` for the
dictionary below. Therefore it cannot tell a key without a value from a
missing key; use 🐣 to find out whether a key is present:

```
🏁 🍇
  🆕🍯🐚🍬🔢🍆❗️ ➡️ 🖍🆕 ages
  31 ➡️ 🐽ages 🔤Anna🔤❗️
  🤷‍♀️ ➡️ 🐽ages 🔤Ben🔤❗️

  ↪️ 🐽ages 🔤Ben🔤❗️ ➡️ age 🍇
    😀 🔡 age❗️❗️
  🍉
  🙅 🍇
    😀 🔤no age for Ben🔤❗️  💭 Printed, although Ben is in the dictionary
  🍉
  ↪️ 🐣ages 🔤Ben🔤❗️ 🍇
    😀 🔤Ben is in the dictionary🔤❗️
  🍉
🍉
```

## Generic Methods and Intializers

It’s also possible to define a generic method, type method or intializer. Such a
method, type method or intializer takes generic arguments which then can be used
as argument types, as return types or as types in the body.

A good example from the standard library is 🍨’s 🐰 method. It is defined like
this:

```
❗️ 🐰 🐚A⚪🍆️ callback 🍇Element➡️A🍉 ➡️ 🍨🐚A🍆 🍇
  💭 ...
🍉
```

As you can see above has one generic parameter named `A` which is restricted
to subtypes of ⚪️, that is any type. Now, if you'd wish to call this method
you can know provide the generic type arguments after the object or class on
which on which you call the method:

```
🍿🔤aa🔤 🔤12345🔤🍆 ➡️ list
🐰🐚🔡🍆 list 🍇 a 🔡 ➡️ 🔡
  ↩️ 🔤🧲a🧲!🔤
🍉❗️
```

The grammar for generic arguments is:

```syntax
$generic-argument-list$-> $type$ [$generic-argument-list$]
$generic-arguments$-> 🐚 $generic-argument-list$ 🍆
```

Emojicode is, however, capable of automatically inferring the generic
arguments for you, so we can just write:

```
🐰 list 🍇 a 🔡 ➡️ 🔡
  ↩️ 🔤🧲a🧲!🔤
🍉❗️
```

and Emojicode will automatically provide `🔡` as generic argument for `A`.

### Closures in Generic Methods

A closure inside a generic method can use the method’s generic parameters just
like the method’s body can: as types of variables, parameters and return
values, to instantiate generic types, and to call methods of their
constraints. This is true for escaping closures too, which can be called after
the method has returned:

```
🕊 🧰 🍇
  🐇❗️ 🖨🐚T ⚪🍆 value T ➡️ 🍇🔢➡️🍨🐚T🍆🍉 🍇
    ↩️ 🍇🎍🥡 count 🔢 ➡️ 🍨🐚T🍆
      🆕🍨🐚T🍆❗️ ➡️ 🖍🆕 copies
      🔂 i 🆕⏩ 0 count❗️ 🍇
        🐻 copies value❗️
      🍉
      ↩️ copies
    🍉
  🍉
🍉

🏁 🍇
  🖨🕊🧰 🔤hello🔤❗️ ➡️ printer
  😀 🔡 📏 ⁉️printer 3❗️❓❗️❗️  💭 Prints 3
🍉
```

## Generic Protocols

It’s also possible to define generic protocols. Generic protocols work
very similar to generic classes and the same compatibility rules apply.

A generic protocol which you might use is 🔂.

```
🐊 🔂🐚Element⚪🍆️ 🍇
  ❗️ 🍡 ➡️ 🍡🐚Element🍆
🍉
```

It takes one generic argument `Element` which determines the generic argument
for the iterator (🍡) the 🍡 method must return.

## Specialization

Generic code is compiled only once for all the types it is used with. To make
that possible, values of generic parameter types are stored in boxes and
methods called on them are looked up at run time, which is slower than
code written for one specific type.

To make generic code as fast as code written for a specific type, the compiler
*specializes* it: when a generic function is called with generic arguments
that are all concrete types, like `🍨🐚🔢🍆` rather than `🍨🐚T🍆`, the compiler
compiles an additional copy of the function for exactly these types. In that
copy, values are not boxed and methods called on them are called directly, so
they can be inlined. Calls in a specialized function are specialized in turn.
A 🔂 loop over a 🍨🐚🔢🍆 in a specialized function, for example, compiles to
a plain loop over the list’s elements without boxing them.

There is no syntax for this; the compiler decides by itself. It specializes:

- methods, type methods and initializers of value types and enumerations that
  are generic or belong to a generic type,
- methods of classes that cannot be overridden, i.e. 🔏 methods and methods of
  [final classes](inheritance.html#final-classes),
- generic functions of imported packages whose body is part of the package’s
  interface, like methods marked [🥯](classes-valuetypes.html#inline).

Methods of classes that can be overridden, initializers and type methods of
classes, and functions of imported packages that are not inline are always
used in their generic form.

Specialization never changes what a program does. The generic form of every
function is still compiled, and the compiler falls back to it whenever
a specialized copy could behave differently or would not compile. For instance,
which overload a call in generic code resolves to is decided once for the
generic code and stays the same in a specialization, even if a more specific
overload would match the concrete type. Warnings are only reported for the
generic code.

>!H If your generic code needs to be fast, write it as a method of a value type
>!H or a 🔏 method, and call it with concrete types.

## Disabling Generic Dynamism

The decorator 🎍🛢 can be used with a class or value type to disable generic
dynamism, like in the example below.

```
🎍🛢 🔏 🐇 🧺🐚Element ⚪🍆️ 🍇
  💭 ...
🍉
```

Every time you instantiate a generic class, the types you provided as generic
arguments will be stored in the newly created instance, which enables casting
with generics, for example. This, however, requires additional time and space.
In special cases it can thus be useful to disable this feature.

If you disable generic dynamism, casting to this type is no longer possible.
The code of the type can also no longer do anything that requires knowing at
run time which type a generic parameter stands for, which the compiler reports
as “Generic dynamism is disabled in this type”. This includes:

- instantiating another generic type with the parameter, like `🆕🍨🐚Element🍆❗️`
  or a list literal of `Element` values,
- taking the size of the parameter with `⚖️Element`,
- reading or writing values of the parameter in memory with 🧠.

Specialization does not help here, as the generic form of the code is always
compiled too. 🎍🛢 does not keep the type’s methods from being specialized.


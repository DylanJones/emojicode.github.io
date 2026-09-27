# Protocols

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

Protocols define methods for special functionality. Protocols only describe
the methods a type must offer to support this functionality. Types can conform
to protocols by implementing all methods and declaring the conformation.

Defining a protocol defines a type. All types that agree to that protocol are
compatible to this type.

## Declaration

The syntax to define a protocol is simliar to the way of defining a class:

```syntax
$protocol$-> 🐊 $type-identifier$ [$generic-parameters$] 🍇 $protocol-body$ 🍉
$protocol-body$-> $protocol-method$ | $protocol-method$ $protocol-body$
$protocol-method$-> [$documentation-comment$] [⚠️] [🖍] $identification$ [$parameters$] [$return-type$]
```

As protocol methods have no body, a ➡️ following a protocol method without a
return type is always read as that method’s return type. A protocol method
declared after a method without a return type can therefore not be an
assignment method (➡️). Declare the assignment method first instead.

For example:

```
🐊 💿 🍇
  ❗️🎶
🍉
```

Here we declared a protocol named 💿. All classes that conform to this protocol
will have to implement the method 🎶. This protocol doesn’t tell us anything
about the actual type but we do know that all types that conform to 💿 are
capable of playing music and therefore must provide the 🎶 method.

You can use the ❗️ to require instance methods inside the 🐊 body. Protocols
can also require ❓ methods, assignment methods and operators, like the s
package’s [📈](generics.html#numbers-in-generic-code). At present it is not
possible to require initializers or type methods.

## Conforming

To make a class conform to a protocol you must declare that it conforms to the
protocol using the conformance syntax:

```syntax
$protocol-conformance$-> 🐊 $type$
```

Let us declare a class that conforms to 💿.

```
🐇 📱 🍇
  🐊 💿

  🆕 🍇🍉

  ❗️ 🎶 🍇
    😀 🔤Lalalala🔤❗️
  🍉
🍉
```

The actual statement to achieve this is `🐊 protocolName`, where *protocolName*
must be the type name of the protocol, and can occur everywhere in the class
body.

[Promises](inheritance.html#promises) also apply when implementing protocol
methods. An extension can also make a class conform to a protocol.

## Calling Methods on Values of Protocol Type

Methods on protocol values are called like any other methods:

```
🖍🆕 cd_like 💿
🆕📱❗️ ➡️ 🖍 cd_like
🎶 cd_like❗️
```

## Mutating Protocol Methods

A value type conforming to a protocol can only implement a protocol method with
a [🖍 method](classes-valuetypes.html#mutability-of-value-types) if the protocol
method is declared 🖍 too:

```
🐊 🔋 🍇
  🖍❗️ 🔌 amount 🔢
  ❓ ⚡️ ➡️ 🔢
🍉
```

Like a 🖍 method of a value type, a 🖍 protocol method can only be called on a
value in a mutable variable. Classes and enumerations can implement 🖍 and
other protocol methods alike.

Values stored in a variable of protocol type keep their value semantics: If
you copy such a value and then call a 🖍 method on it, only the value you
called the method on changes.

```
🕊 🔦 🍇
  🐊 🔋
  🖍🆕 charge 🔢

  🆕 🍼 charge 🔢 🍇🍉

  🖍❗️ 🔌 amount 🔢 🍇
    charge ⬅️➕ amount
  🍉

  ❓ ⚡️ ➡️ 🔢 🍇
    ↩️ charge
  🍉
🍉

🏁 🍇
  🖍🆕 battery 🔋
  🆕🔦 20❗️ ➡️ 🖍battery
  battery ➡️ spare
  🔌 battery 50❗️
  😀 🔡 ⚡️battery❓❗️❗️  💭 Prints 70
  😀 🔡 ⚡️spare❓❗️❗️  💭 Prints 20
🍉
```

As parameters are constant, a method that wants to call a 🖍 protocol method
on a parameter must copy it into a mutable variable first.

## Multiprotocols

It might happen that you’ll need to deal with values of types that implement
several protocols. For instance, you might want to provide a method which
requires an argument that can be accessed with 🐽️ and can be compared as defined
by the 💿 protocol. This is where multiprotocols are of service.

You can use a multiprotocol type like so:

```
🍱 🐽️🐚🔡🍆 💿 🍱
```

For instance, when declaring the arguments to a method:

```
❗️ 🌈 a 🍱 🐽️🐚🔡🍆 💿 🍱 🍇
  💭 ...
🍉
```

As expected, `a` can now be used both as an instance of a type conforming to
🐽️🐚🔡🍆 and as an insatnce of a type conforming to 💿.

This means you can call the methods of all its protocols on a multiprotocol
value, and pass it wherever one of its protocols is expected:

```
🐊 🏷 🍇
  ❗️ 📛 ➡️ 🔡
🍉

🐇 📟 🍇
  🐊 💿
  🐊 🏷

  🆕 🍇🍉

  ❗️ 🎶 🍇
    😀 🔤Lalalala🔤❗️
  🍉

  ❗️ 📛 ➡️ 🔡 🍇
    ↩️ 🔤pager🔤
  🍉
🍉

🐇 🎧 🍇
  🐇❗️ 🎚 player 💿 🍇
    🎶 player❗️
  🍉

  🐇❗️ 📢 device 🍱 💿 🏷 🍱 🍇
    😀 📛 device❗️❗️
    🎚🐇🎧 device❗️  💭 device is passed as a 💿
  🍉
🍉
```

A multiprotocol value cannot be passed where a multiprotocol of only some of its
protocols is expected, though: A value of `🍱 💿 🏷 🐽️🐚🔡🍆 🍱` cannot be
passed as a `🍱 💿 🏷 🍱`.

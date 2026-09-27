# Inheritance and Overriding

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

Inheritance allows instances of a more concrete class to be treated like
instances of a less concrete class. When you subclass a class, the subclass
inherits all methods and type methods. Initializers are only inherited under
special conditions.

A class cannot inherit from itself, neither directly nor through its
superclasses. The compiler rejects this, for example:

```!
🐇 🐟 🐠 🍇 🍉
🐇 🐠 🐟 🍇 🍉
```

## Initializer Inheritance

In Emojicode, initializers are only inherited by subclasses if the subclass
meets these criteria:

- The subclass does not define any instance variables.
- The subclass does not define any initializer.

If the subclass doesn’t fullfil these criteria, the subclass doesn’t inherit
any initializers.

## Required Initializers

Sometimes it’s desired that all subclasses of a class have a specific
initializer. In this case, a required initializer can be used.

An initializer can be marked as required with the 🔑 attribute. Subclasses must
implement all initializers marked with 🔑 themselves if they aren't eligible
for initializer inheritance. These initializer again must then be marked with
🔑, thereby enforcing that all descendants of the original class provide the
required intalizer.

## Overriding Methods

A subclass can override a method defined in a superclass, that is providing
a new implementation of it.

A method is overriden by redeclaring it in the subclass with the ✒️ attribute.
As for example:

```
🐇 🌼 🍇
  ❗️ ⏰ time 🔢 🍇
    💭 Open and close the blossom according to the time...
  🍉
🍉

🐇 🌻 🌼 🍇
  ✒️ ❗️ ⏰ time 🔢 🍇
    💭 Sunflowers also rotate to face the sun....
    ⤴️⏰ time  💭 Open and close like other flowers; see below
  🍉
🍉
```

The same logic applies to type methods.

You cannot override generic methods and the compiler will never consider a
generic method the super-method of another method.

### Access Level and Overriding

If you override a method and do not specify an access level, the method inherits
the overridden method’s access level. In the example below, 🐡’s 🙋 method is 🔐
because no access level was specified:

```
🐇 🐟 🍇
  🆕 🍇🍉

  🔐 ❗️ 🙋 🍇
    😀 🔤I’m a fish.🔤❗️
  🍉
🍉

🐇 🐡 🐟 🍇
  ✒️❗️ 🙋 🍇
    😀 🔤I’m a blowfish.🔤❗️
  🍉
🍉
```

An overriding method must be at least as acessible as the method it overrides.
This means that you cannot make an overriding method 🔒, nor can you
override a 🔓 method with a 🔐 method.

## Calling Super Methods

Inside a method you can use this syntax to call the super method:

```syntax
$super$-> ⤴️ $emoji-id$ [$arguments$]
```

This simply calls the super method named *method-emoji* and returns it value.
You have already seen an example above.

## Final Classes

The attribute 🔏 marks a class as final. A final class cannot be subclassed or
an compiler error will be raised.

>!H Marking a class as final not only makes your intent clear but can also
>!H lead to performance improvements. Although the
>!H compiler tries to automatically detect final classes, it cannot do so in
>!H packages that export types.

The following example will raise a compiler error as 🐟 is attributed with 🔏.

```!
🔏 🐇 🐟 🍇
  🆕 🍇🍉
🍉

🐇 🐡 🐟 🍇

🍉
```

Likewise, a method attributed with 🔏 cannot be overridden by subclasses:

```!
🐇 🐟 🍇
  🆕 🍇🍉

  🔏 ❗️ 🙋 🍇
    😀 🔤I’m a fish.🔤❗️
  🍉
🍉

🐇 🐡 🐟 🍇
  ✒️ ❗️ 🙋 🍇
    😀 🔤I’m a blowfish.🔤❗️
  🍉
🍉
```

Methods of final classes and 🔏 methods are called directly instead of being
looked up at run time. If they are generic or belong to a generic class, they
can also be [specialized](generics.html#specialization).

## Promises

You must watch out not to break the superclass’s *promises* when overrding
methods. Promises are a set of rules that ensure that the methods and required
intializers of a class can be used the same way as the ones of the superclass –
a main characteristic of object orientation. These promises are:

- The method or initializer of the subclass must take the same number of
  parameters.
- The return type of the method or initializer of the subclass must be the
  same or a subtype of the return type of the overriden method or intializer.
- The parameters of the method or initializer of the subclass must be of the same
  type or a super type of the argument types of the overriden method or
  intializer.
- The method or initializer of the subclass must have the same accessibility
  or be more accessible.
- If the super method or initializer is not escaping, the overriding method or
  initializer may not be escaping.
- If a parameter of the overriding method or initializer is escaping, so must
  be the respective parameter of the super method or initializer.

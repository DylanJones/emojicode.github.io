# Threads

>!N **AI-edited:** Parts of this page were written or revised by AI (Claude) to document changes in this fork of
>!N Emojicode, and have not been fully reviewed by a human. If something here disagrees with the compiler, the
>!N compiler is right.

Emojicode offers concurrency, that is doing more than one thing at a time,
and allows you to create threads, which can each independently execute code.

## Creating Threads

Threads are represented by the 🧵 class of the s package. You can create a new
thread by using the 🆕 initializer which accepts a callable which will be called
on the newly created thread.

For example:

```
🆕🧵 🍇🎍🥡
  😀 🔤I execute on a different thread.🔤❗️
🍉❗️
```

The closure must be [escaping](callables.html#capturing-variables-and-context),
as the thread may still run it after the 🆕 initializer has returned.

If you wrapped the above into the 🏁 method compiled it into a program, you
would possibly not see any output. The problem is, that the thread is created
and the main thread, on which the 🏁 method was called, would continue to
execute and would finally reach the end of the program. The program would
likely terminate before the second thread had the chance to do anything.

In order to avoid this, you can wait for another thread to terminate with the
help of the 🛂 method. If the thread has already finished the 🛂 will
immediately return. The example below creates 5 threads (creating one
thread and waiting for it doesn't make much sense) and then waits for each to
finish.

```
🏁 🍇
  🆕🍨🐚🧵🍆❗️ ➡️ 🖍🆕threads

  🔂 i 🆕⏩ 0 5❗️ 🍇
    🐻 threads 🆕🧵 🍇 🎍🥡
      😀 🔤On a dark desert highway, cool wind in my hair🔤❗️
      😀 🔤Warm smell of colitas, rising up through the air🔤❗️
      😀 🔤Up ahead in the distance, I saw a shimmering light🔤❗️
    🍉❗️❗️
  🍉

  🔂 thread threads 🍇
    🛂 thread❗️
  🍉
🍉
```

The output of running the above code is similar to this:

```
On a dark desert highway, cool wind in my hairOn a dark desert highway, cool wind in my hairOn a dark desert highway, cool wind in my hairOn a dark desert highway, cool wind in my hair
On a dark desert highway, cool wind in my hair



Warm smell of colitas, rising up through the airWarm smell of colitas, rising up through the airWarm smell of colitas, rising up through the airWarm smell of colitas, rising up through the air
Warm smell of colitas, rising up through the air



Up ahead in the distance, I saw a shimmering lightUp ahead in the distance, I saw a shimmering lightUp ahead in the distance, I saw a shimmering lightUp ahead in the distance, I saw a shimmering light
Up ahead in the distance, I saw a shimmering light
```

The output is messed up because all threads are trying to execute
simultaneously. The order in which a thread will get the opportunity to actually
print something depends on your hardware as well as many other factors, like
load factor of the computer.

### Threads That Are Not Joined

You do not have to join every thread. If the last reference to a 🧵 goes away
before 🛂 was called on it, the thread keeps running in the background until it
has finished. This is what happens in the very first example of this chapter.

Such threads, however, are terminated when the program exits, even if they have
not finished their work yet. So join a thread if its work must be completed.

## Race Conditions and Mutexes

Imagine the following program:

```
🐇 🏦 🍇
  🖍🆕 account 🔢

  🆕 🍇
    300 ➡️ 🖍account
  🍉

  ❗️ 💸 sum 🔢 🍇
    account ⬅️➖ sum
  🍉

  ❗️ 💶 ➡️ 🔢 🍇
    ↩️ account
  🍉
🍉

🏁 🍇
  🆕🍨🐚🧵🍆❗️ ➡️ 🖍🆕threads

  🆕🏦❗️ ➡️ account

  🔂 i 🆕⏩ 0 10❗️ 🍇
    🐻 threads  🆕🧵 🍇 🎍🥡
      🔂 j 🆕⏩ 0 5❗️ 🍇

        ↪️ 💶 account❗️ ▶️🙌 10 🍇
          😀 🔤Money, money, money – Must be funny🔤❗️
          💸 account  10❗️
        🍉

      🍉
    🍉❗️❗️
  🍉

  🔂 thread threads 🍇
    🛂 thread❗️
  🍉

  😀 🔡 💶 account❗️  10❗️❗️ 💭 Print the balance
🍉
```

It creates a bank account with an initial credit balance of 300€. Then 10
threads are created which each try to withdraw 10€ 5 times if there is money
left. You might now expect that the balance will be 0€ in the end because the
threads only tried to get more money when available but never overdraw the
account. Let’s try.

```
...
Money, money, money – Must be funny
-70
```

Strange, let’s try again.

```
...
Money, money, money – Must be funny
-90
```

You probably already know what happened. What we experience here is called a
*race condition*. Let’s analyze this part of our code again:

```
↪️ 💶 account❗️ ▶️🙌 10 🍇  💭 There’s money left
  😀 🔤Money, money, money – Must be funny🔤❗️
  💸 account 10❗️
🍉
```

Imagine the following situation: A thread comes and sees that exactly 10€
are left. So the body of the ↪️ is entered and the thread prints
a message. In exactly this moment another thread comes along, checks the balance
and also sees there are 10€ left and enters the body of the ↪️. The former
thread now moves on to withdraw 10€ as the second one will do after it has
printed a message. So they withdrew 20€! In reality the program is even faster
and all threads execute the same piece of code at virtually the same time.

Now, how can we fix this? The solution is to use an instance of 🔐, which is also
called a *mutex*. A mutex ensures that only ever one thread can access a data
structure or run a piece of code.

We’ve reworked our example to use a mutex:

```
🏁 🍇
  🆕🍨🐚🧵🍆❗️ ➡️ 🖍🆕threads

  🆕🏦❗️ ➡️ account

  🆕🔐❗️ ➡️ mutex

  🔂 i 🆕⏩ 0 10❗️ 🍇
    🐻 threads  🆕🧵 🍇 🎍🥡
      🔂 j 🆕⏩ 0 5❗️ 🍇
        🔒 mutex❗️
        ↪️ 💶 account❗️ ▶️🙌 10 🍇
          😀 🔤Money, money, money – Must be funny🔤❗️
          💸 account  10❗️
        🍉
        🔓 mutex❗️
      🍉
    🍉❗️❗️
  🍉

  🔂 thread threads 🍇
    🛂 thread❗️
  🍉

  😀 🔡 💶 account❗️  10❗️❗️ 💭 Print the balance
🍉
```

If a thread comes to the line `🔒 mutex❗️` it will try to *lock* the mutex. If the
mutex is not already locked, that is if no other thread has already passed this
line, the thread will be able to do this and will continue. Otherwise however,
the thread will wait until it itself can lock the mutex. The thread which
grabbed the mutex *unlocks* the mutex at the end of the critical part by calling
`🔓 mutex❗️`. At this point another thread can get the chance to lock the mutex –
which thread however depends on your hardware, operating system, etc.

An operation like this which is protected from disturbances by other threads is
called *atomic*.

And now, no matter how often you try, you will always get the expected result:

```
...
Money, money, money – Must be funny
Money, money, money – Must be funny
0
```

If a thread should not wait for a mutex that is locked by another thread, it can
use 🔐 instead of 🔒. 🔐 locks the mutex and returns 👍 if it is not locked,
and returns 👎 immediately otherwise:

```
🏁 🍇
  🆕🔐❗️ ➡️ mutex
  🔒 mutex❗️

  🆕🧵 🍇🎍🥡
    ↪️ 🔐 mutex❗️ 🍇
      😀 🔤Got the mutex🔤❗️
      🔓 mutex❗️
    🍉
    🙅 🍇
      😀 🔤The mutex is locked, doing something else🔤❗️
    🍉
  🍉❗️ ➡️ thread
  🛂 thread❗️

  🔓 mutex❗️
🍉
```

As the main thread holds the mutex while the other thread runs, this prints
`The mutex is locked, doing something else`.

## Atomicity of the s Package

It’s important to note that none of the classes in the s package guarantee
atomicity. If you access an s package data structures always make sure to use
a mutex to avoid race conditions.

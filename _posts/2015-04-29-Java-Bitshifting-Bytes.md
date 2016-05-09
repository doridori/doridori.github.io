---
layout: post
title: "Java: Bitshifting bytes"
---

**TL;DR When bitshifting a `byte` use a `& 0xFF` mask**

Working at the bit level in Java can be very frustrating. This is for two reasons

1. There are no unsigned number primitives
2. Automatic type promotion is not intuitive!

Looking at type promotion, whenever you use a bitwise operator on a `byte` variable it is automatically promoted to an `int`. 

This means you have to write casting code such as

`byte b = (byte) (b ^ 8);` or `b ^= 8` which does the former under-the-hood.

This is not too bad really. But the pain starts when working with signed numbers together with `int` promotion. For example

<div data-gist-id="387bdce2ed4ad2b95efe" data-gist-file="ex1.java">ex1.java</div>

For `bByte` we would expect the result to be `0b1111_1001` as the right [Arithmetic shift](http://en.wikipedia.org/wiki/Arithmetic_shift) operator `>>` fills the left bit depending on the left most (sign) bit (which is `1` when negative ala [2's complement](http://en.wikipedia.org/wiki/Two%27s_complement)) so the result is as expected.

However for `cByte` we would expect the result to be `0b0000_1001` as the right [Logical shift](http://en.wikipedia.org/wiki/Logical_shift) operator `>>>` should fill the left most bit with `0`, but we still get `-7`! Why is this happening?

Well due to the auto int promotion (as a result of using any bitwise operator), when the `aByte` is promoted to an `int`, the left most bits of that int are all `1`s. In binary form this is

`0b1111_1111_1111_1111_1111_1111_1001_0000` before the shift

`0b0000_1111_1111_1111_1111_1111_1111_1001` after the shift

Due to [how casting works when casting down](http://stackoverflow.com/a/2458526/236743) the last 8-bits will be copied directly, hence still giving `0b1111_1001`. Not very intuitive when you see `aByte >>> 4` imho.

So how can we perform `>>>` operations and get the "intuitive" result of `0b0000_1001`?

_Note: This is useful as to be able to do as Java does not have an unsigned byte type the above kind of operation would let you use a java `byte` in place of another languages `unsigned byte` type. This can useful when porting code, for example, if you want to replicate a `ubyte >> 4` call then the below will be useful._

The answer is to mask the bitwise result before casting back down to `byte`. We can do this with `& 0xFF`. This works by persevering only the last 8 bits of the promotioned-to `int` only, and dropping all the extra `1` bits.When casting down we then get the result we are looking for. I.e. 

<div data-gist-id="387bdce2ed4ad2b95efe" data-gist-file="ex2.java">ex2.java</div>

This is a bit clunky but a valuable technique if you are encountering unexpected results from your bitshifts!

## Links

- [A Curse on Java Bitwise Operators!](http://henkelmann.eu/2011/02/24/a_curse_on_java_bitwise_operators)
- [Bitwise operators in java only for integer and long?](http://stackoverflow.com/a/20577639/236743)


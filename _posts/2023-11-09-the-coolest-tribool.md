---
layout: post
title: "The Coolest Tribool"
date: 2023-11-09
---

A common function in the GBA community is to convert the directional pad buttons into a signed integer value for each axis.

If Left is pressed, a `-1` is returned, if Right is pressed, a `+1` is returned. If both, or neither, are pressed: `0` is returned.

This is often called a "tribool", AKA a 3-state value.

When represented with `-1`, `0`, `+1`, the tribool can be directly added to a position value to move a character left or right. Very handy.

# Implementing Tribool

Assume our tribool is constructed from a bitmask, where bit 0 is `+1`, and bit 1 is `-1`.

## First version

Most programmers would probably write something like this as their first attempt:

```c
int tribool(int mask) {
    const int bit0 = mask & 1;
    const int bit1 = (mask >> 1) & 1;
    
    if (bit0 == bit1) return 0;
    return bit0 ? 1 : -1;
}
```

Compiled with GCC `-O2` the 32-bit ARM code is:

```arm
tribool:
        asr     r3, r0, #1
        and     r3, r3, #1
        and     r0, r0, #1
        cmp     r0, r3
        beq     .L3
        cmp     r0, #0
        mvneq   r0, #0
        bx      lr
.L3:
        mov     r0, #0
        bx      lr
```

## Tonclib version

The [libtonc](https://github.com/devkitPro/libtonc) implementation looks like:

```c
int tribool(int mask) {
    const int bit0 = mask & 1;
    const int bit1 = (mask >> 1) & 1;

    return bit0 - bit1;
}
```

Compiled with GCC `-Og` the 32-bit ARM code is:

```arm
tribool:
        and     r3, r0, #1
        asr     r0, r0, #1
        and     r0, r0, #1
        sub     r0, r3, r0
        bx      lr
```

This looks astonishingly optimised, however I suspected we could do better.

# Sign extending

On the GBA the key indices for each axis on the dpad, and the shoulder buttons, are next to each other.

I noticed that simply sign-extending the upper bit will get us somewhat close to a tribool implementation:

```c
int almost_tribool(int mask) {
    return (mask << 30) >> 30;
}
```

For an 8-bit number, this sign extend produces the following states:

| 2-bit Mask | 8-bit Sign Extended | "Tribool" State |
|------------|---------------------|-----------------|
| 0b00       | 0b00000000          | 0               |
| 0b01       | 0b00000001          | 1               |
| 0b10       | 0b11111110          | -2              |
| 0b11       | 0b11111111          | -1              |

Unfortunately this does not respect the exclusivity requirement of our tribool function: When both bits are equal (`0b00` or `0b11`), the function must always return `0`.

But this is close, and for original GBA hardware where it is physically impossible to hit both left and right buttons at the same time this almost works (as long as you only do sign comparisons or compare against zero).

For anyone interested the GCC `-Og` 32-bit ARM code is:

```arm
almost_tribool:
        lsls    r0, r0, #30
        asrs    r0, r0, #30
        bx      lr
```

A simple sign-extend.

## Fixing up the bits

My first instinct was to analyse the possible states, and figure out how to manually fix the `-2` and `-1` states.

I noticed that just adding `1` gets these into range, however this breaks the `0` and `1` cases:

| 2-bit Mask | 8-bit Sign Extended + `1` | "Tribool" State |
|------------|---------------------------|-----------------|
| 0b00       | 0b00000001                | 1               |
| 0b01       | 0b00000010                | 2               |
| 0b10       | 0b11111111                | -1              |
| 0b11       | 0b00000000                | 0               |

So we should only add `1` for the negative cases, conveniently the cases with a sign-bit set:

```c
int tribool(int mask) {
    mask = (mask << 30) >> 30;
    if (mask >= 0) return mask;
    return mask + 1;
}
```

Compiled with GCC `-O1` the 32-bit ARM code is perfect:

```arm
tribool:
        lsl     r0, r0, #30
        asrs    r0, r0, #30
        addmi   r0, r0, #1
        bx      lr
```

The `addmi` instruction is conditionally executed only when the `mi` CPU flag (the minus flag) is set, which the preceeding `asrs` instruction sets if the result or arithmetically shifting right is a negative/minus number.

The 16-bit thumb code with GCC `-O1` is equally as good:

```arm
tribool:
        lsls    r3, r0, #30
        asrs    r0, r3, #30
        lsrs    r3, r3, #31
        adds    r0, r0, r3
        bx      lr
```

In this code, the second `lsrs` converts the sign bit into the number `1` and adds that to the original value.

### What about `-Og`?

Notice that I showed the GCC `-O1` assembly output for the fixed implementation of sign-extended tribool.

For GDB debugging on embedded systems, especially retro consoles like the GBA, you should really be using `-Og`.

GCC `-Og` 32-bit ARM:

```arm
tribool:
        lsl     r0, r0, #30
        asrs    r0, r0, #30
        bxpl    lr
        add     r0, r0, #1
        bx      lr
```

GCC `-Og` 16-bit thumb:

```arm
tribool:
        lsls    r3, r0, #30
        asrs    r0, r3, #30
        cmp     r3, #0
        bge     .L1
        adds    r0, r0, #1
.L1:
        bx      lr
```

The ARM code isn't too bad, with `bxpl` returning if the result is positive, but the thumb code now has a full-fat branch.

We can further fix this by explicitly writing out the 16-bit thumb logic in C:

```c
int tribool(int mask) {
    mask = (mask << 30) >> 30;
    int fix = (unsigned int) mask >> 31;
    return mask + fix;
}
```

Compiled with GCC `-Og` 16-bit thumb:

```arm
tribool:
        lsls    r3, r0, #30
        asrs    r0, r3, #30
        lsrs    r3, r3, #31
        adds    r0, r0, r3
        bx      lr
```

Compiled with GCC `-Og` 32-bit ARM:

```arm
tribool:
        lsl     r0, r0, #30
        lsr     r3, r0, #31
        add     r0, r3, r0, asr #30
        bx      lr
```

This is the best we can get with 32-bit ARM, with 1 instruction fewer than Tonclib.

For 16-bit thumb, there is one more optimisation available.

# Register pressure

There's never enough registers on a CPU, so often a lot of time is spent juggling registers on the stack.

If we're inlining these functions (which for sometihng this small, you probably should be) then optimising how many registers are used becomes important.

The problem is the temporary used:

```c
int fix = (unsigned int) mask >> 31;
```

The `fix` value depends on `mask`, and our `mask + fix` return value depends on both, so two registers will be used.

If we could remove `fix` then the compiler will be able to operate solely on the `mask`.

## Analysis of the bits

Looking back at the 8-bit sign extended binary:

| 2-bit Mask | 8-bit Sign Extended | "Tribool" State |
|------------|---------------------|-----------------|
| 0b00       | 0b00000000          | 0               |
| 0b01       | 0b00000001          | 1               |
| 0b10       | 0b11111110          | -2              |
| 0b11       | 0b11111111          | -1              |

The two correct states (`0` and `1`) only differ by bit index 0 (the lowest bit), and the two incorrect states (`-2` and `-1`) differ by *the same* bit index 0.

If we were to shift right by 1 bit with sign extend then we can separate the correct and incorrect states into two values:

| 2-bit Mask | 8-bit Sign Extended | Shifted right 1  | Fix State |
|------------|---------------------|-----------------|-----------|
| 0b00       | 0b00000000          | 0b00000000      | 0         |
| 0b01       | 0b00000001          | 0b00000000      | 0         |
| 0b10       | 0b11111110          | 0b11111111      | -1        |
| 0b11       | 0b11111111          | 0b11111111      | -1        |

The correct states are now `0`, and the incorrect states are now `-1`, we can subtract by this `-1` to apply our `+ 1` fix from earlier:

| 2-bit Mask | Sign Extended | Fix State | Sign Extended - Fix State |
|------------|---------------|-----------|---------------------------|
| 0b00       | 0             | 0         | 0 - 0 = **0**             |
| 0b01       | 1             | 0         | 1 - 0 = **1**             |
| 0b10       | -2            | -1        | -2 - -1 = **-1**          |
| 0b11       | -1            | -1        | -1 - -1 = **0**           |

And now we have our desired tribool.

The C for this is:

```c
int tribool(int mask) {
    mask = (mask << 30) >> 30;
    return mask - (mask >> 1);
}
```

But wait, this still internally requires a temporary register for `mask >> 1` to be added to the original `mask`, so all we've done is hide the register in C.

## Ones' complement rabbit hole

After staring at the binary for a while I noticed that the two incorrect states are **ones' complement** of the actual state I needed them to be in.

So this whole time I was trying to figure out how to convert 2-bit ones' complement negative numbers into 2-bit twos' complement.

The Wikipedia articles for both ones' complement and two's complement has good information:

* [Ones' complement](https://en.wikipedia.org/wiki/Ones%27_complement)
* [Two's complement](https://en.wikipedia.org/wiki/Two%27s_complement)

This took me down the rabbit hole of trying to convert using a combination of bit flips (`~mask`) and two's complement negation (`-mask`), but nothing I tried was an improvement.

# The coolest tribool

I went back to the analysis of the `+ 1` fix:

| 2-bit Mask | 8-bit Sign Extended + `1` | "Tribool" State |
|------------|---------------------------|-----------------|
| 0b00       | 0b00000001                | 1               |
| 0b01       | 0b00000010                | 2               |
| 0b10       | 0b11111111                | -1              |
| 0b11       | 0b00000000                | 0               |

After spending so long staring at the bits, I noticed something: The values I was looking for were right there all along!

| 2-bit Mask | 8-bit Sign Extended + `1` |
|------------|---------------------------|
| 0b00       | 0b00000**00**1            |
| 0b01       | 0b00000**01**0            |
| 0b10       | 0b11111**11**1            |
| 0b11       | 0b00000**00**0            |

Just right shifting one more place (with sign extend) gets the results I had been looking for.

| 2-bit Mask | 8-bit Sign Extended + `1` | Shift Right | Tribool State |
|------------|---------------------------|------------|---------------|
| 0b00       | 0b00000001                | 0b00000000 | 0             |
| 0b01       | 0b00000010                | 0b00000001 | 1             |
| 0b10       | 0b11111111                | 0b11111111 | -1            |
| 0b11       | 0b00000000                | 0b00000000 | 0             |

The C for this is the obvious:

```c
int tribool(int mask) {
    mask = (mask << 30) >> 30;
    return (mask + 1) >> 1;
}
```

Compiled with GCC `-Og` 16-bit thumb:

```arm
tribool:
        lsls    r0, r0, #30
        asrs    r0, r0, #30
        adds    r0, r0, #1
        asrs    r0, r0, #1
        bx      lr
```

Compiled with GCC `-Og` 32-bit ARM:

```arm
tribool:
        lsl     r0, r0, #30
        asr     r0, r0, #30
        add     r0, r0, #1
        asr     r0, r0, #1
        bx      lr
```

Notice that the assembly for both thumb and ARM only operates on `r0`, which is the input parameter `mask`.

## Final implementation

32-bit ARM mode has more available registers than 16-bit thumb, so optimising for register pressure isn't necessarily the best choice.

My final implementation selects between optimising for code size with 32-bit ARM, and register pressure for 16-bit thumb.

```c
int tribool(int mask) {
    mask = (mask << 30) >> 30; // Sign extend high bit
#if defined(__arm__) && !defined(__thumb__)
    return mask - (mask >> 1); // Optimise for code size
#else
    return (mask + 1) >> 1; // Optimise for register pressure
#endif
}
```

Compiling for GCC `-Og` on x86_64 will select the register pressure optimisation:

```x86asm
tribool:
        mov     eax, edi
        sal     eax, 30
        sar     eax, 30
        add     eax, 1
        sar     eax
        ret
```

## Inverting the tribool

The GBA keypad register is actually bit inverted to what most programmers would expect, so a button pressed is `0`, and a button released is `1`.

Most developers invert the keypad mask after reading it, so they can test `mask & KEYLEFT` and get the result they expect.

This extra invert isn't strictly necessary (especially if you're using C++ which can wrap the raw data with accessing member functions), but the tribool implementation needs to be inverted to account for `0` being our "true" value, and `1` being our "false" value.

```c
int inv_tribool(int mask) {
    mask = (mask << 30) >> 30; // Sign extend high bit
#if defined(__arm__) && !defined(__thumb__)
    return (mask >> 1) - mask; // Optimise for code size
#else
    return (0 - mask) >> 1; // Optimise for register pressure
#endif
}
```

Notice the symmetry with the regular function.

The thumb and ARM assembly is just as sensible:

16-bit thumb:

```arm
inv_tribool:
        lsls    r0, r0, #30
        asrs    r0, r0, #30
        rsbs    r0, r0, #0
        asrs    r0, r0, #1
        bx      lr
```

32-bit ARM:

```arm
inv_tribool:
        lsl     r0, r0, #30
        asr     r3, r0, #31
        sub     r0, r3, r0, asr #30
        bx      lr
```

# Compiler Explorer link

[godbolt.org/z/z6Y6efb9h](https://godbolt.org/z/z6Y6efb9h)

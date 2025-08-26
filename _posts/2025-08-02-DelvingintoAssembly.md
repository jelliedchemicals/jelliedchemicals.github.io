---
title: Delving into Assembly
date: 2025-08-02 -0800 # Yes, write out the date, it doesnt automatically pick it up. -800 might be whats messing up the date and making it a day earlier so consider removing.
categories: [HAE]
tags: [hacking and the art of exploitation, jon erickson, hataoe, hae, projects, linux, objectives, hacking, assembly, C programming]     # TAG names should always be lowercase
author: <author_id>
description: Avoiding tutorial hell and challenging myself with assembly
toc: false
---


<h4>Reengaging</h4>
To start, I know the gap between these posts has been extensive to say the least. That's not to say that I stopped studying entirely, as I continued to read the remainder of Linux Basics for Hackers. I realized while reading through LBFH that I was learning a little bit, but overall I was doing exactly what I need to avoid. It wasn't challenging me enough, and I was spending too much time in the weeds scrounging for the new pieces of information. I knew I needed to change things up, so I stopped writing notes on what I was going through in that book. I did, however, continue to read all of it, allowing myself to skip over parts that were too familiar.

I needed to be challenged more than what that book was provided, although I did enjoy it and I think it's a great resource. That said, I picked up Hacking and the Art of Exploitation by Jon Erickson again and setup a simple VM on my laptop so I could follow along. 

Previously, I have read through the first 120 or so pages of this book, but that was some time ago and I didn't have a VM setup to follow along. I recognize what I have written out below is, at times, too closely following the book, but I hope to break away from that more and more as I learn and ask more questions. I also want to consult other resources like a local RE group I recently joined and delving into other materials. Amanda Rousseau, aka MalwareUnicorn, has RE workshops that look very interesting, so those are another source I want to get into. 

Now, lets dive into Hacking and the Art of Exploitation.

<h4>0x250 through 0x252</h4>
<h6>Arithmetic, Logic, and Symbols in C</h6>
Let's keep the arithmetic, logic, and symbols refresh quick.

**Modula reduction (%)** - takes the remainder after division
	13 % 5 = 2 with a remainder of 3, so the modula reduction is 3

Variables and shorthand for arithmetic operations:
```
i = i + 12    i+=12
i = i - 12     i-=12
i = i * 12     i*=12
i = i / 12     i/=12
```

Logic and symbols for shorthand
OR     ||
AND  &&
Ex: ((a < b) || (a < c)) 

Also ! typically means *not*
Ex: !(a > b) is equal to (a >= b)

Alright, lets move on to writing a program in C.

<h6>Writing Our First Program in C</h6>
firstprog.c

```
#include <stdio.h>     // Tells the compiler to include headers for a standard                          // input/output library named stdio (also will this                              // comment break things?)

int main()
{
	int i;
	for(i=0; i < 10; i++)     // loop 10 times
	{
		puts("Worried you're insane?\n");     // put the string to the output
	}
	return 0;     // Tell OS the program exited without errors.
}
```

GNU environment has a tool called **objdump** which can be used to examine compiled binaries
Ex: `objdump -M intel -D firstprog.c | grep -A20 main.:`
	Objdump would spit out too many lines of output, pipe it into grep to display 20 lines after the regular expression `main.:`
		Each byte is represented by a hex value
	This example uses the Intel formatting rather than AT&T
		Configure GDB to run in Intel format every time by putting it in the **.gdbinit** home directory
			`echo "set dis intel" > ~/.gdbinit`

<h4>0x253 Assembly Language</h4>
**Debuggers** are used to step through compiled programs step-by-step, examine program memory, and view processor registers. GNU uses **GDB**

```
gdb -q ./firstprog.out

(gdb) break main
(gdb) run

(gdb) info registers // debugger is told to display all processor registers and                       // their curren states
```

**EAX, ECX, EDX**, and **EBX** are the **general purpose registers**
**EAX** is the **Accumulator**
**ECX** is the **Counter**
**EDX** is the **Data**
**EBX** is the **Base**
These are mainly used as temporary variables for the CPU when it is executing machine instructions

**ESP, EBP, ESL** and **EDI** are also general purpose registers, but they are sometimes known as **pointers** and **indexes**
**ESP** is the **stack pointer**
**EBP** is the **base pointer**
**ESL** is the **source index**
**EDI** is the **destination index**

GDB in Intel syntax generally follows the following style
`operation <destination>, source`
The **destination** will be a register, a memory address, or value
The **operations** are mnemonics:
	**mov** is move
	**sub** is subtract
	**inc** is increment
	**cmp** is compare
	**jmp** is jump
	**jle** is jump if less than or equal to
The instructions below move the value from ESP to EBP and then subract 8 from ESP, storing the results in ESP
```
8048375:    89 e5         mov    ebp,esp
8048377:    83 ec 08      sub    esp, 0x8
```

When compiling, we can use the `-g flag` when using GCC to include extra debugging information, which gives GDB access to the source code

```
gcc -g firstprog.c
gdb -q ./a.out
(gdb) list    // shows source code, it seems

(gdb) disassemble main    // provides dump of assmebler code for function main():
...
End of assembler dump
(gdb) break main
(gdb) run
(gdb) i r eip    // info register abbreviation for eip

```

GDB provides a direct way to examine memory, with the command `x`
`x` can use different display formats
`o` displays in octal
`x` displays in hex
`u` displays in unsigned, standard base-10 decimal
`t` displays in binary

```
(gdb) i r eip
eip    0x8048384    0x8048384 <main+16>
(gdb) x/o 0x8048384
0x8048384 <main+16>:    077042707
(gdb) x/x $eip
0x8048384 <main+16>:    0x00fc45c7
(gdb) x/u $eip
0x8048384 <main+16>:    16532935
(gdb) x/t $eip
0x8048384 <main+16>:    00000000111111000100010111000111
```

In GDB, execute the current instruction with `nexti` which is short for next instruction. The processor will read the instructions at EIP, execute it, and advance EIP to the next instruction.

```
(gdb) x/10i $eip
0x804838b <main+23>:   cmp   DWORD PTR [ebp-4], 0x9
0x804838f <main+27>:   jle   0x8048393 <main+31>
0x8048391 <main+29>:   jmp   0x80483a6 <main+50>
0x8048393 <main+31>:   mov   DWORD PTR [esp],0x8048484
0x804839a <main+38>:   call  0x80482a0 <printf@plt>
0x804839f <main+43>:   lea   eax,[ebp-4]
0x80483a2 <main+46>:   inc   DWORD PTR [eax]
0x80483a4 <main+48>:   jump 0x804838b <main+23>
0x80483a6 <main+50>:   leave
0x80483a7 <main+51>:   ret
```

`cmp` here compares the memory used by variable `i` with the value 9. **The result from the comparison will be stored in the EFLAGS register**. Then, if the value is less than or equal to 9, it will jump the **EIP** to main+31, or the memory address 0x8048393. Otherwise it will continue to the `jmp` instruction, which is an unconditional jump to 0x80483a6. The unconditional jmp instruction points to the address for the `leave` instruction.

Prior to this, we saw the value 0 stored in the memory address being compared, so EIP should be at 0x8048393 after executing `nexti`. This leads us to the `mov` instruction that writes the 0x8048484 into **ESP**. If we run `i r esp` to find what ESP is pointing to, we see that it's address is 0xbffff800. As the book asks, why is 0x8048484 being written to 0xbffff800?

```
(gdb) x/2xw 0x8048484
0x8048484:   0x6c6c6548   0x6f57206f
(gdb) x/6xb 0x8048484
0x8048484:   0x48   0x65   0x6c   0x6c   0x6f   0x20
(gdb) x/6ub 0x8048484
0x8048484:   72   101   108   108   111   32
```

0x8048484 has the string "Hello world!" stored, with 72, 101, 108, 108, 111, and 32 referring to ASCII for H, e, l, l, o, and a space, respectively. 

We can see this more easily with the examine tool using the `c` format letter and `s` format letter. `c` lets us automatically look up a byte on the ASCII table, `s` will display an entire string of character data. 

`(gdb) x/6cb 0x8048484` lets us see each ASCII letter next to its corresponding unsigned **numbers** bytes. `x/s 0x8048484` returns the string at that location of `Hello, world!\n"`.

My own note here, but why did `x/6ub 0x8048484` above return `111` and then `32` (o followed by a space) rather than `111   44` for `o` followed by `,` since `x/s 0x8048484` returns the string with a comma before the space. The `x/6cb 0x8048484` example just before it, showing us how c returns the ASCII character for each corresponding byte, shows a space after the `o`. I'm not sure if I missed something, it will be explained shortly, or it was a mistake in the book. 

I came back and rewrote the program and ran `x/s 0x8048484` here to show I wrote it to have a comma in the string, then ran `x/8cb 0x8048484` and confirmed that the book missed a comma, unsigned decimal 44, before 32, which referred to the space in the string.
![Image showing x/s showing the memory address and it's contents of the string. x/8cb then shows the numbers and related ASCII characters, revealing that there should have been a 44 for the comma.](https://github.com/jelliedchemicals/jelliedchemicals.github.io/blob/main/assets/images/HAE/HAE1.jpg)
 
I did this quickly, so I rewrote the code and then examined the exact memory address 0x8048484 as per what the book was looking at. What I could've done better here was reviewing the steps that led us to find 0x8048484 and ran those to confirm I needed the same address.
	
Getting back to the instructions, the book discusses the `printf()` function. From here, we can move forward and look at the next two instructions, `lea` and `inc`, together. These two instructions increment the variable `i` by 1. `lea` refers to **Load Effective Address**, which will load `[ebp-4]` into the EAX register. The `inc` instruction increments the value found at this address, now stored in the EAX register, by 1. 

All of this is of course the for loop, which will continue until the variable reaches 10, at which point it will not meet the conditional jump, and instead move onto the unconditional jump which exits the program.

This seems like a good stopping point for this post. We'll pick this back up and look at various concepts in C.

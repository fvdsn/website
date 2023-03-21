# Memory Is A Big Array; An Easy Tutorial To Low Level Programming


At the very basics, a computer running a program has the following parts:

 1. Memory, also known as RAM
 2. A CPU, or processor

The Memory contains the instructions of your program, as well as all its data and
various variables.

The CPU can read and write from the memory, understand and execute the instructions. Note
that the CPU doesn't understand JavaScript or Python, but rather a specific set of low level
instructions that are usually called assembly. There are different families of CPU that understand
different sets of instructions. The most famous are ARM (Arduino, Mac, Smartphones) and x86 (Intel, AMD)

Also, the CPU doesn't work directly on the memory, but rather on registers, which are small 

Let's look at a simple (and simplified) example of a program that adds 1 plus 1. That simplified computer

The memory of the computer would look like the following:

```
 0 `



I recently realized that many people learned to program with high level
languages such as JavaScript or Python and never had a chance to learn low
level programming. If you are one of these dudes, this article is for you !

You'll learn exactly what it means for a system to be 32bit or 64bit, what is
a stack overflow, how memory is allocated, and other fun stuff. 

The main difference between high level and low level programming is probably how
memory is managed. In high level programming languages you don't really have to
think about it. Strings, maps, files, numbers, objects, they're just "there". You
ask for them, you get them. But behind the scenes those objects are actually put
somewhere. In low level programming you have to tell the program how and where to
put and find all of these things yourself, in other words you have to manage the
memory yourself. But what is the memory ?

Physically, the memory of your program corresponds to pieces of the RAM of your computer.
This is where all the variables and objects used by the program are stored and modified
while your program runs. Conceptually that memory is organized as one very big array of
bytes (8bits numbers, from 0 to 255). Just as the arrays you already know, each element is indexed by
an integer. We call these indexes 'addresses'.  In 32bit systems the addresses are  32bits integer, and so the maximum
index is 2^32-1 == 4294967295. Since each element of the array is one byte, that means you
can't have more than 2^32 bytes in the memory of your program. You might remember that in old computers,
programs could not have more than 4GB of memory, well that's where it comes from; 2^32 bytes is 4GB.

```  
   address:    0     1     2        2^32-2 2^32-1
            +-----+-----+-----+-   -+-----+-----+
   value:   |  0  |  42 | 255 | ... | 19  | 66  |
            +-----+-----+-----+-   -+-----+-----+
```

Nowadays, most systems are 64bit, meaning the memory is indexed by 64bit integers. 2^64 is a lot and
so the number of addresses is not a practical limit for the size of 

In 2023 nearly all computers are 64bit, meaning your programs can index memory with 64bit integers. 
2^64 is a lot more than 2^32 and now the index is not a practical limit anymore. 



When you program with low level languages, you are interacting directly with this 'big array'. When you
store a va

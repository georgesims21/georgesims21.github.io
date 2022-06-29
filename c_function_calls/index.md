# C function calls (32-bit)


Each CPU has the following registers (64-bit in parantheses):
* PC (IP): Instruction pointer  - Points to next instruction for the CPU to execute
* SP (SP): Stack pointer        - Points to the top of the stack
* FP (BP): Base pointer         - Points to the stack frame of the current active function
* RVR (AX): Return value      - Points to the function return value

```c
main()                  |   int sub(int x, int y)
{                       |   {
int a, b, c;            |   int u, v;
a = 1; b = 2; c = 3;    |   u = 4; v = 5;
c = sub(a, b);          |   return x+y+u+v;
printf("c=%d\n", c);    |   }
}                       |
```

When a C program is envoked, the return address (current PC) is pushed onto the stack and then the value stored in the PC register is replaced with the function we wish the CPU to execute. In the above case when we wish to run the program (call main()), the current PC is pushed and PC is replaced by the entry address of main(). In this diagram XXXX denotes stack data before we run the program:

```
(Stack grows downward)
High address      -->     Low address
-------------------------------------
            | XXXX | PC |
                        ^
                       SP
-------------------------------------
```

Every C program follows the same procedure when called (following main and sub program):
1. Push FP onto the stack to save the CPU's previous stack frame location
```
            | XXXX | PC | FP |
                             ^
                            SP
```
2. Point FP to the new FP to establish the new stack frame
```
            | XXXX | PC | FP |
                             ^
                           SP,FP
```

3. Allocate space for auto local variables, but don't initialise them
```
                               a   b   c
            | XXXX | PC | FP | ? | ? | ? |
                             ^           ^
                            FP          SP
```
4. Potentially allocate more for temporary memory (denoted by tmp)
```
                               a   b   c
            | XXXX | PC | FP | ? | ? | ? | tmp |
                             ^                 ^
                            FP                SP
```
5. The code starts executing, in our case assigning the values to the variables
   (in main) comes first
```
                               a   b   c
            | XXXX | PC | FP | 1 | 2 | 3 | tmp |
                             ^                 ^
                            FP                SP
```
Using the assumption that we know an int is 4 bytes (i.e sizeof(int) == 4), the
assignments can be done individually by simply measuring the distance between
the FP we saved and the registers next to it, thus -4(FP) == a, -8(FP) == b and
-12(FP) == c. In assembly code this would look like:
```assembly
movl $1, -4(%ebp) /* move value 1 ($1) into register located -4(FP)*/
movl $2, -8(%ebp)
movl $3, -12(%ebp)
```
6. In our case, main() calls sub() via a direct assignment to the variable c.
First the parameters given to the sub() method are pushed in reverse order, so b
and then a. We then repeat the same steps (because this is a C program) from 1.
to make the CPU execute the sub() function. So it would look something like this afterwards:
```
        Stack frame of                    Stack frame of
|---------- main() ----------|     |--------- sub() ---------|
                   a   b   c         b   a             u   v
| XXXX | PC | FP | 1 | 2 | 3 | tmp | 2 | 1 | PC | FP | 4 | 5 | tmp |
                 ^                                   ^             ^
                 |                                  FP            SP
                 +------------ points to ------------+
```
You can also see from this diagram that accessing the parameters given to a
function is positively relative to the FP rather than negative: 8(FP) == a and
12(FP) == b. This format is exactly how a stack looks to the calling function,
or more generally:
```
| -------------------- Function's stack frame ----------------------|
| params | return PC (CPU should return to) | saved FP | local vars |
```

A simple program like this:
```
crt0.o -> main() -> A(par_a) -> B(par_b) -> C(par_c)
```
Should look like this when thinking of stack frames:

```
                   SF of        SF of
         ...    |- main() -|  |- A() -| ...
| 0 |
    ^ 
    +--| FP0 |
             ^
             +--| FPmain() |
                           ^
                           +--| FPA() |
                                      ^
                                      +--| FPB() |
                                                 ^
                                                 +--| FPC() |
                                                            ^
                                                       CPU.FP
```

7. The return value of sub() is placed into the AX register and the
function returns to the callers stack frame. Before returning however, the local
variables must be deallocated:
* First copy the value of the current FP into SP, so they are pointing to the
same thing
```
                   a   b   c         b   a 
| XXXX | PC | FP | 1 | 2 | 3 | tmp | 2 | 1 | PC | FP | 
                                                     ^
                                                 FP,SP
```
* Next pop the stack into the FP pointer, giving it the FP of the caller
```
                   a   b   c         b   a 
| XXXX | PC | FP | 1 | 2 | 3 | tmp | 2 | 1 | PC |
                 ^                              ^
                FP                             SP,(CPU.PC)
```
* Finally the RET function is executed, popping the top of the stack into the PC
  register, making the CPU execute from the saved return address, belonging to
  the caller
```
                   a   b   c         b   a 
| XXXX | PC | FP | 1 | 2 | 3 | tmp | 2 | 1 |
            ^    ^                         ^
     (CPU.PC)   FP                        SP
```
8. Lastly, the caller 'catches' the value in the AX register, and cleans the
   remaining parameters from the called function by simply adding 8 to the SP
   (2* sizeof(int)) and moves on to the next instruction after main. The figure
   below shows that we are back to being inside the stack frame of main() only
```
           Stack frame of
| ------------ main() -------------|
                   a   b   c
| XXXX | PC | FP | 1 | 2 | 3 | tmp |
            ^    ^                 ^
     (CPU.PC)   FP                SP
```

### Side note: long jump
A long jump can be used to esentially skip returning to the caller, and return
elsewhere, normally earlier than the caller. If we have a program in which:
main() -> A() -> B(), but upon return B() jumps to main() instead of its caller A().
```c
/** longjump.c file: demonstrate long jump in Linux **/
#include <stdio.h>
#include <setjmp.h>
jmp_buf env; // for saving longjmp environment

int main() {
    int r, a=100;
    printf("call setjmp to save environment\n");
    if ((r=setjmp(env)) == 0){ // saves cur exec env in jmp_buf strct
        A();
        printf("normal return\n");
    } else
        printf("back to main() via long jump, r=%d a=%d\n", r, a);
}

int A() { 
    printf("enter A()\n");
    B();
    printf("exit A()\n");
}

int B() {
    printf("enter B()\n");
    printf("long jump? (y|n) ");
    if (getchar()=='y')
        longjmp(env, 1234);
    printf("exit B()\n");
}
```
The idea is that the registers containing the callers PC and FP are replaced
by the values stored in the jmp_buf struct, which in the above example are
main()'s, which means we can follow the regular steps and we will skip A()
completely. It is also possible for the longjmp() function to save the CPU's
current registers and SP so that it can be restored if needed.

```
                   Replace with saved
                          |
| XXXX | PC | FP | ... |  =>  | XXXX | SavedPC | SavedFP | ... |
                 ^     ^                                 ^     ^
                FP    SP                                FP    SP
```


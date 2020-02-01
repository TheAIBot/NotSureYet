## Why is CPU IPC improvements in the single digits?

In order to explain one of the biggest problems here a few things about programming. In order to do so we will invent
our own little programming language which runs on our own fancy CPU.


```
x = a + b
y = c + d
z = x + y
```

The above is a simple program. Programs are executed from top to bottom. This somewhat looks like what a programmer writes when writing a program.
The program loads the values from a, b, c and d into the cpu, calculates x, then y and at last z.

Converting the program to something more alike to what the cpu uses would like something like this:

```
load a
load b
add x a b
load c
load d
add y c d
add z x y
```

The above program does the exact same thing but there is now a specific order in which things are done. 
This way of reading the program is essentially the way a CPU reads a program. 
From top to bottom with a very specific order in which things should be done.

### Making our first CPU

Now lets get to the core of the issue. In the program above we have two different instructions the CPU can execute. 
First is a load instruction which loads a value from memory into the CPU, second instruction adds two numbers together and stores the result in the CPU.

These two instructions does not take the same time to execute. `load` takes 100 cycles and `add` takes 1 cycle.
If we just execute the instructions in a sequential order from top to bottom then the program takes 403 cycles.
The first release of our CPU runs in this sequential manner.
in 402 cycles we've executed 6 instructions. That's an IPC of 0.017. Absolutely horrendous performance.

### Second version of the CPU - Adding out of order execution

In the second version of our CPU we would like to increase our IPC.
We make it so the CPU is able to understand instruction dependencies and make it so the CPU is able to change the order in which
instructions are executed, as long as the instruction dependeinces are the same. 
In our program one such dependency is the `add x a b` which depends on the two instructions `load a` and `load b` having finished before the add can start.

We also add hardware to the CPU so it can now do multiple loads at once.

The CPU is now able to read the program and rearrange the instructions into the following program.

```
load a
load b
load c
load d
add x a b
add y c d
add z x y
```

Because the CPU can execute multiple loads at once, it can do the  4 loads in 100 cycles instead of 400!
A massive improvement that decreases the programs execution time to 103 cycles and increases the IPC to 0.068

### Third version of the CPU - Adding cache



The third version

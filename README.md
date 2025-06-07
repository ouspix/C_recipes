# Table of Contents

* [Preface – Why Learn C Now?](#preface--why-learn-cnow)
* [Part 0 – Editor, Terminal, and Your First Useful Makefile](#part-0--editor-terminal-and-your-first-useful-makefile)
* [Part 1 – First Program: start → speak → finish](#part-1--first-program-start--speak--finish)
* [Part 2 – Storing Numbers and Letters](#part-2--storing-numbers-and-letters)
* [Part 3 – Letting the Program Decide](#part-3--letting-the-program-decide)
* [Part 4 – Doing Things Many Times](#part-4--doing-things-many-times)
* [Part 5 – Giving Work a Name : Functions](#part-5--giving-work-a-name--functions)
* [Part 6 – Lists That Don’t Move : Arrays](#part-6--lists-that-dont-move--arrays)
* [Part 7 – Talking by Address : Pointers](#part-7--talking-by-address--pointers)
* [Part 8 – Building Your Own Data Shapes](#part-8--building-your-own-data-shapes)
* [Part 9 – Asking the Operating System for Memory](#part-9--asking-the-operating-system-for-memory)
* [Part 10 – Files, Streams, and Why Disks Are Slow](#part-10--files-streams-and-why-disks-are-slow)
* [Part 11 – Many-File Projects and Why `make` Matters](#part-11--manyfile-projects-and-why-make-matters)
* [Part 12 – When Programs Misbehave : Debuggers & Sanitizers](#part-12--when-programs-misbehave--debuggers--sanitizers)
* [The Most Secure Communication System](#the-most-secure-communication-system)

---

## Preface – Why Learn C Now?<a id="preface--why-learn-cnow"></a>

C is everywhere you can’t see — and many places you can:

* **Mars landings.**  Flight-critical software on NASA’s rovers is written mostly in C because engineers trust its tiny, predictable runtime.
* **The cloud and your kettle.**  From Kubernetes’ networking stack down to the microcontroller regulating your electric kettle, the same language handles workloads nine orders of magnitude apart.
* **Open-source bedrock.**  Git, Python’s interpreter, the core of PostgreSQL, and the Linux kernel all share C as their common ancestor. Knowing it means you can step into those projects without translation layers.
* **Career portability.**  Once you understand pointers and build pipelines here, everything from Rust’s `unsafe` blocks to Go’s cgo bindings becomes legible overnight.

What this course is **not**: an ode to bit-twiddling for its own sake.
What it **is**: a guided climb from a blank folder to a small, functioning blockchain, fixing real-world bugs as they appear and learning only the machinery you need at each step.

We’ll keep the historical nods short, the examples hands-on, and the toolchain modern (sanitizers on, warnings as errors). By the end, you’ll look at a random C file on GitHub and think, *“Yeah, I know how that works—and I can change it.”*

---

## Part 0 – Editor, Terminal, and Your First Useful Makefile<a id="part-0--editor-terminal-and-your-first-useful-makefile"></a>

### 0·0 Why this setup matters and what will happen

Real coding is an endless loop: **edit → build → run**.
Running the compiler by hand is fine for one file, but the moment a project grows past a couple of `.c` files you’ll rebuild needlessly or miss something and chase phantom bugs.

The 1970s answer is `make`: feed it a **Makefile** that describes *relationships* (“`hello` depends on `hello.c`”), and it figures out what actually needs work.
`make` hasn’t been dethroned because that problem—*only rebuild what changed*—never went away.

You’re about to:

1. create a project folder in VS Code,
2. write a Makefile that already scales to any number of `.c` files,
3. trigger a compile-time error (missing semicolon) so you recognise the message,
4. fix it, build cleanly, and see an executable pop out.

That’s the one-command workflow you’ll use for the rest of the course.

---

### 0·1 Write the Makefile and understand every line

> **Folder setup**
> VS Code → **File ▸ Open Folder…** → create/select `c_course`.

1. **Create a new file named `Makefile`** (no extension).
2. **Paste—and read—this script:**

```make
# ---------- Minimal yet practical Makefile --------------------------
CC      = gcc                         # the C compiler program
CFLAGS  = -Wall -Wextra -std=c17 -g   # warnings + C17 + debug info

# Pattern rule: "target % becomes binary, prerequisite %.c is source"
# $<  = first prerequisite   (hello.c)
# $@  = target being built   (hello)
%: %.c
	$(CC) $(CFLAGS) $< -o $@

# all  expands to every .c file’s basename, then relies on the rule above
# $(wildcard *.c)   → list all .c files in this folder
# $(patsubst %.c,%, …)   → strip the .c extension
all: $(patsubst %.c,%, $(wildcard *.c))

# 'make clean' deletes generated binaries; useful before Git commits
clean:
	rm -f $(patsubst %.c,%, $(wildcard *.c))
# --------------------------------------------------------------------
```

*Quick decode of the weird symbols*

| Token                  | Read as                | Example in this file                 |
| ---------------------- | ---------------------- | ------------------------------------ |
| `%`                    | “whatever fits here”   | turns `%.c` into *any* `.c` filename |
| `$<`                   | first prerequisite     | `hello.c` inside the rule            |
| `$@`                   | target being built     | `hello` (no extension)               |
| `$(wildcard *.c)`      | list of all `.c` files | `hello.c utils.c …`                  |
| `$(patsubst %.c,%, …)` | replace suffix         | `hello utils`                        |

---

### 0·2 First run — nothing to build

Open a terminal in the folder (**Terminal ▸ New Terminal**) and type:

```bash
make
```

Output:

```
make: Nothing to be done for 'all'.
```

With zero `.c` files present the `all` target expands to an empty list, so `make` sensibly does no work.

---

### 0·3 Introduce a compile-time error on purpose (missing semicolon)

1. **Create `hello.c` with a tiny but invalid program:**

```c
int main(void) {                 /* semicolon deliberately omitted */
    return 0
}
```

2. Run `make`:

```
gcc -Wall -Wextra -std=c17 -g   hello.c   -o hello
hello.c:2:12: error: expected ‘;’ before ‘}’ token
```

*What happened* – the **compiler** refused to translate code with broken syntax.

3. **Fix the error** by adding the semicolon:

```c
int main(void) { return 0; }
```

4. Run `make` again—no errors, and an executable named `hello` appears.

---

### 0·4 Run the program and confirm the exit code

```bash
./hello      # program does nothing visible
echo $?      # prints 0 – exit status travelled back to the shell
```

Change `return 0;` to `return 7;`, rebuild, run again, and `echo $?` now prints **7**: you control the signal that scripts and CI servers will read.

---

### 0·5 Checkpoint

You now have:

* a Makefile that builds any `.c` file you drop in,
* the ability to spot a compile-time error, fix it, and rebuild in one command,
* and confirmation that a finished program returns a meaningful exit code.

Everything else in this course—printing text, loops, pointers, even a blockchain—will reuse the **edit → make → run** loop you just wired up.

## Part 1 – First Program: start → speak → finish<a id="part-1--first-program-start--speak--finish"></a>

### 1·0 Why this tiny program matters—and what you will see happen

Before loops, pointers, or files, we need one visible victory: “I wrote code, the computer ran it, and I can prove it.”
That proof takes three checkpoints:

1. **Reach `main`.**
   Every C program *must* have a function called `main`. When you launch the program, the operating system jumps straight there. If `main` is missing, nothing else matters.

2. **Show text on-screen.**
   Your program already has a pipe called **`stdout`** connected to the terminal window. Sending a short line through that pipe is the fastest way to prove the CPU hit your code.

3. **Return a status number.**
   When `main` ends it gives one integer back to the shell. The shell stores it in `$?`. By convention `0` means “all good”; anything else means “problem.” Scripts and build tools look only at this number.

In this part you will:

* create `hello.c` with a real `main`, one `puts`, and `return 0;`;
* build it with the Makefile from Part 0;
* run it, see the greeting, and confirm the exit code is 0;
* break the build twice—first a syntax error, then a missing header—read the error messages, and fix them.

Once those steps work, the edit → make → run loop is solid for everything ahead.

---

### 1·1 Write the source file

Create **`hello.c`** in your `c_course` folder and type:

```c
#include <stdio.h>          /* tells the compiler about puts()       */

/* main: required starting point; int return value becomes exit code */
int main(void)
{
    puts("Hello, electricity flowing through silicon!"); /* to stdout */
    return 0;                                            /* success    */
}
```

**What each line does**

| Line                 | Purpose                                                                            |
| -------------------- | ---------------------------------------------------------------------------------- |
| `#include <stdio.h>` | Brings in the declaration of `puts`. Without it the compiler can’t check the call. |
| `int main(void)`     | Defines the entry point. Returning `int` lets us send a status code.               |
| `puts(...)`          | Prints a full line and adds its own newline. Quick and safe.                       |
| `return 0;`          | Hands “success” back to the shell.                                                 |

---

### 1·2 Build and run

```bash
make          # Makefile from Part 0 handles the compile command
./hello       # runs the program; greeting appears
echo $?       # shell prints 0 – the exit code you set
```

---

### 1·3 Change the exit code and see it travel

Edit `hello.c`:

```c
return 7;
```

Re-build and run:

```bash
make
./hello
echo $?       # now prints 7
```

You control the status number that scripts and CI pipelines read.

---

### 1·4 Trigger a **compile-time** error (missing semicolon) and fix it

Delete the semicolon after `puts(...)`:

```c
puts("Broken now!")   /* ← no ; here */
return 0;
```

Run **`make`**:

```
hello.c:5:1: error: expected ‘;’ before ‘return’
```

The compiler stops immediately because the source code is syntactically wrong.
Add the semicolon back, save, and build cleanly.

---

### 1·5 Trigger a **header** error and fix it

Comment out the include:

```c
/* #include <stdio.h> */
```

Run **`make`**:

```
hello.c: In function ‘main’:
hello.c:5:5: warning: implicit declaration of function ‘puts’
/usr/bin/ld: ...: undefined reference to `puts'
collect2: error: ld returned 1 exit status
```

*What happened*

* The compiler only warns (it guessed a prototype).
* The linker later fails because it couldn’t find the real `puts`.

Un-comment the header, build again, and the program works.

---

### 1·6 Checkpoint

* `main` is recognised, so your program starts correctly.
* `stdout` shows the greeting, proving the code ran.
* Exit codes flow to the shell (`$?`).
* You can read and fix a compile-time error (syntax) and a link-time error (missing symbol).

With the basic pipeline working, you’re ready to give the program something to *store*—numbers, characters, and the limits of each.

## Part 2 – Storing Numbers and Letters<a id="part-2--storing-numbers-and-letters"></a>

### 2·0 Why this part matters—and what you will see happen

Up to now the program only *ran* and *said hello*; it never kept a single piece of information.
Real software needs memory—little boxes that remember a value while the program is running.
In C those boxes are called **variables**, and each one has a **type** that answers two questions:

1. *How many bytes does this box take?*
2. *What operations make sense on the data inside?*

The language gives you some ready-made box styles:

| Type     | Typical size\* | What it can hold                               |
| -------- | -------------- | ---------------------------------------------- |
| `char`   | 1 byte         | One character or raw byte                      |
| `int`    | 4 bytes        | Whole numbers (-2 147 483 648 … 2 147 483 647) |
| `double` | 8 bytes        | Real numbers like 3.14                         |

\*On a modern 64-bit PC. Sizes can vary on tiny embedded chips.

In this part you will

* declare three variables (one of each type) and print their sizes;
* see what happens when an `int` grows past its maximum (overflow);
* fix two common beginner mistakes: using the wrong `printf` format and forgetting a cast.

When you finish, you’ll know how big the basic boxes are and how to ask the program itself (`sizeof`) instead of guessing.

---

### 2·1 Write the source file

Create **`types.c`** and type the code below.
Read the comments—they explain each line in plain language.

```c
#include <stdio.h>
#include <limits.h>   /* INT_MAX, INT_MIN */
#include <float.h>    /* DBL_MAX          */

/*
   This program shows:
   * how to declare variables of different types
   * how to print their values and sizes
   * what integer overflow looks like
*/
int main(void)
{
    char   initial = 'A';        /* 1-byte character box        */
    int    age     = 30;         /* 4-byte whole-number box     */
    double price   = 2.99;       /* 8-byte real-number box      */

    /* %c prints a char, %d prints an int, %.2f prints 2-decimals float */
    printf("initial = %c  (uses %zu byte)\n",  initial, sizeof initial);
    printf("age     = %d  (uses %zu bytes)\n", age,     sizeof age);
    printf("price   = %.2f (uses %zu bytes)\n", price,   sizeof price);

    /* show the largest int and what happens if we add 1 */
    printf("INT_MAX = %d\n", INT_MAX);
    printf("INT_MAX + 1 = %d  <-- overflow demonstration\n", INT_MAX + 1);

    return 0;
}
```

---

### 2·2 Build and run

```bash
make           # compiles types.c → types
./types
```

You should see each variable’s value, its size in bytes, and the overflow wrap-around.

---

### 2·3 Small experiment — unsigned wrap

Edit the declaration of `age`:

```c
unsigned int age = 30;
```

Change the print line to `%u` (unsigned) and compile again.
Now set `age = 0;` and print `age - 1`.
Unsigned integers wrap **from 0 to the maximum** instead of going negative.

---

### 2·4 Intentional format-string error and the fix

Change one line to use the wrong placeholder on purpose:

```c
printf("price as int = %d\n", price);   /* wrong: %d expects int */
```

Run **`make`**:

```
types.c: warning: format ‘%d’ expects argument of type ‘int’, but argument 2 has type ‘double’
```

*Why this matters* – mismatched formats scramble output and can crash programs with stricter compilers.

**Fix**: use `%f` or cast:

```c
printf("price as int = %d\n", (int)price);  /* explicit cast is clear */
```

Re-compile; the warning disappears.

---

### 2·5 Literal overflow warning and the fix

Change the overflow demo:

```c
int big = 3000000000;   /* 3 billion > INT_MAX on 32-bit int */
```

Compile:

```
warning: overflow in implicit constant conversion
```

**Fix**: use a wider type:

```c
long big = 3000000000L;
printf("big = %ld\n", big);
```

---

### 2·6 Checkpoint

* You declared variables of three fundamental types.
* `sizeof` told you their actual byte sizes on your machine.
* You witnessed signed and unsigned wrap-around.
* You fixed a printf-format mismatch and an overflow warning.

Now your program can **remember** simple values. Next we’ll teach it to **choose**: `if` this, `else` that.


---

## Part 3 – Letting the Program Decide<a id="part-3--letting-the-program-decide"></a>

### 3·0 Why decisions matter—and what you will see happen

So far the program runs the same way every time: start, print, exit.
Real software needs to **react** to data.
Does the number a user typed look valid? Is the file bigger than one megabyte?
In C, the keywords `if` and `else` let you choose between two (or more) paths.

Behind the curtain, a decision is nothing fancy: the CPU checks whether a value is zero.

* Zero → “false”
* Anything else → “true”

That’s it. With that single rule you can build complex logic.

In this part you will:

* read an integer from the keyboard,
* decide if it’s even or odd,
* print the answer,
* watch a common typo (`=` instead of `==`) break the logic and learn the fix,
* and see how a simple check keeps bad input from crashing the program.

When you finish, your code will **look at data** and **choose** what to do.

---

### 3·1 Write the source file

Create **`parity.c`** and type the code below.
Every comment explains *why* the line exists.

```c
#include <stdio.h>   /* printf, puts, scanf live here */

/*
   parity.c
   ---------
   Reads one whole number and prints "even" or "odd".
   Demonstrates:
     * reading input safely
     * if / else decision
     * modulo (%) operator
*/
int main(void)
{
    int n;                              /* box for the user’s number */

    printf("Enter a whole number: ");

    /* scanf returns how many items it filled.
       If the user types nonsense, we bail out with exit code 1.      */
    if (scanf("%d", &n) != 1) {
        puts("Input error – please enter digits only.");
        return 1;
    }

    /* The modulo operator (%) gives the remainder of division.
       Even numbers have remainder 0 when divided by 2.               */
    if (n % 2 == 0)
        puts("even");                  /* stdout */
    else
        puts("odd");

    return 0;                          /* success */
}
```

---

### 3·2 Build and run

```bash
make            # your Makefile handles the compile step
./parity
Enter a whole number: 42
even
echo $?         # prints 0 – success code
```

Run again, type `17`, and you see `odd`.

---

### 3·3 Small experiment – the truthiness rule

Change the condition line to:

```c
if (n % 2)          /* note: NO '== 0' */
```

Rebuild and run:

* If you enter an odd number, `n % 2` is 1 and C treats 1 as *true* → prints “odd” (good).
* If you enter an even number, `n % 2` is 0 and C treats 0 as *false* → it falls to the `else` and prints “even”.

You just proved that **zero is false; anything else is true**.
Put the original `if (n % 2 == 0)` back for clarity.

---

### 3·4 Intentional error – the single-equals trap

Replace the comparison with an assignment by mistake:

```c
if (n % 2 = 0)      /* WRONG: single = */
```

Rebuild:

```
parity.c: warning: suggest parentheses around assignment used as truth value
```

Run the program, type any number: it always prints “even”.
Why? `n % 2 = 0` *stores* 0 into the remainder (which makes no sense) and the whole expression becomes 0 (false).
The `else` branch never runs.

**Fix:** change back to `==`. The warning disappears and logic works.

---

### 3·5 Extra guard – rejecting huge or negative numbers

Add another decision after reading the input:

```c
if (n < 0) {
    puts("Negative numbers are fine, but let's keep it simple today.");
    return 2;                        /* different error code */
}
```

Compile, run, enter `-5`; the program exits early with code 2.
You now see how multiple `if` blocks can stack to create richer checks.

---

### 3·6 Checkpoint

* You read keyboard input safely and detect bad input.
* You used `if / else` to choose one of two messages.
* You verified C’s simple truth rule (0 = false, non-zero = true).
* You spotted and fixed the classic `=` vs `==` typo.
* You understand how to return a **different exit code** for different error situations.

Next part: repeat work without copy-paste—**loops**.

---

## Part 4 – Doing Things Many Times<a id="part-4--doing-things-many-times"></a>

### 4·0 Why this part matters—and what you will watch happen

In real life we repeat ourselves constantly:
count down the last 10 seconds of a rocket launch, print the next page of a report, send packets until the other side says “got it.”
A **loop** is the programming tool that turns one instruction into *however many times you need*.

In this part you will

1. Meet the two most common loop keywords—`for` and `while`—and see when each one feels natural.
2. Build a small program that prints **FizzBuzz** for numbers 1 → N (a tech-interview classic).
3. Tweak the code to feel the difference between `<` and `<=` (the famous off-by-one bug).
4. Trigger an **infinite loop** on purpose, watch the terminal misbehave, then stop it with **Ctrl-C** and fix the mistake.
5. End with two short “Try it” challenges that force you to predict loop behaviour before running it—good brain exercise.

By the end you will be comfortable setting up, running, and stopping repetitions without fear.

---

### 4·1 A 60-second tour of loop anatomy

| Part of the loop header | Example     | Why it exists                                              |
| ----------------------- | ----------- | ---------------------------------------------------------- |
| **initialiser**         | `int i = 1` | creates / resets the counter                               |
| **test**                | `i <= N`    | checked *before* every run; if false, the loop stops       |
| **update**              | `++i`       | moves the counter toward the point where the test is false |

A `for` loop lets you write all three pieces in one neat line:

```c
for (initialiser ; test ; update) {
    body;
}
```

A `while` loop shows only the **test**; you write the initialiser before the loop and the update inside.
Choose whichever makes the code easiest to read.
For counted, predictable runs (`i = 1 … 100`) most developers reach for `for`.

---

### 4·2 Write the program – `fizzbuzz.c`

```c
#include <stdio.h>
#include <stdlib.h>    /* atoi – string → int */

/*
   FizzBuzz:
     * Prints numbers from 1 to limit.
     * Multiples of 3  → "Fizz"
     * Multiples of 5  → "Buzz"
     * Multiples of 15 → "FizzBuzz"
*/
int main(int argc, char *argv[])
{
    /*--- 1. Decide the upper limit ----------------------------------*/
    int limit = 100;                     /* default */
    if (argc == 2) limit = atoi(argv[1]);/* user override */

    /*--- 2. Main loop ------------------------------------------------*/
    for (int i = 1; i <= limit; ++i) {
        if      (i % 15 == 0) puts("FizzBuzz");
        else if (i % 3  == 0) puts("Fizz");
        else if (i % 5  == 0) puts("Buzz");
        else                  printf("%d\n", i);
    }

    return 0;
}
```

*Why the order `15 → 3 → 5`?*
If we tested `i % 3 == 0` first, 15 would print “Fizz” and never reach the “FizzBuzz” case.
Always test the most specific condition first.

---

### 4·3 Build and run

```bash
make          # Makefile from Part 0 takes care of compilation
./fizzbuzz    # prints 1 … 100 with substitutions
./fizzbuzz 30 # prints only up to 30
```

---

### 4·4 Small experiment – feel the off-by-one

Change the loop header to `i < limit` (strictly less) and re-run `./fizzbuzz 5`.
Output stops at **4**; 5 is skipped.
Put the header back to `i <= limit`.

> **Pedagogical pitfall:** Beginners often write `<` when they meant `<=`.
> The compiler won’t warn; only the missing final item (or an array overflow) tells you something’s wrong.

---

### 4·5 Intentional error – create an infinite loop and stop it

Edit the update field so it *does nothing*:

```c
for (int i = 1; i <= limit; /* ++i missing! */) {
```

Re-build and run `./fizzbuzz 10`.
The program prints “1” forever—`i` never changes, so the test is always true.

1. Press **Ctrl-C** to kill the runaway program.
2. Restore `++i` in the header.
3. Re-build; loop behaves again.

*Lesson:* forgetting the update clause (or putting the wrong one) locks the loop.

---

### 4·6 Try it – two quick challenges

1. **Reverse countdown**
   Modify the loop to print 10, 9, 8 … 1 without changing the body—just the header.
   *Hint:* start `i` at `limit`, test `i >= 1`, and use `--i`.

2. **Sum of numbers**
   Add an `int sum = 0;` before the loop, then inside the `else` branch do `sum += i;`.
   After the loop prints, add `printf("Sum=%d\n", sum);`.
   Predict the sum for `limit = 5` (should be 1 + 2 + 4 = 7, because 3 and 5 were replaced). Run and check.

---

### 4·7 Checkpoint

* You can write a counted `for` loop with clear initialiser, test, and update.
* You understand why testing order matters in multi-branch `if / else if` ladders.
* You experienced the classic off-by-one bug and an infinite-loop bug, and fixed both.
* You can modify loop direction and accumulate results inside the loop.

Next part: stop repeating yourself in *source code*—wrap reusable work into **functions**.

## Part 5 – Giving Work a Name: Functions<a id="part-5--giving-work-a-name-functions"></a>

### 5·0 Why this part matters—and what you will watch happen

Copy-pasting the same chunk of code in three places feels quick—until you must change it in four.
A **function** packages a job (the *body*) and gives it a label (the *name*).
Whenever you need that job done, you call its name instead of re-typing the steps.

In this part you will

1. Package two tiny jobs—finding the larger of two numbers (`max`) and their average (`average`)—into functions.
2. See how **prototypes** (function declarations) let the compiler check calls even when the body comes later in the file.
3. Trigger an error by calling a function without a prototype, read the warning, then add the missing line.
4. Discover why integer division can silently cut off decimals—and how one extra `0` in a literal (`2` → `2.0`) keeps real maths real.
5. Finish with a mini-challenge to reuse your new functions in another context.

After this part you will be able to *name* work once and *reuse* it anywhere, with the compiler guarding your spelling and argument types.

---

### 5·1 Write the source file – `math_demo.c`

```c
#include <stdio.h>

/* ----------- 1. Prototypes (declarations) --------------------------
   They tell the compiler:
     “There exists a function called max that takes two ints
      and returns an int.  You'll see its body later.”               */
static int max(int a, int b);
static double average(int a, int b);
/* ------------------------------------------------------------------ */

int main(void)
{
    int x, y;

    printf("Type two whole numbers: ");
    if (scanf("%d %d", &x, &y) != 2) {
        puts("Input error.");
        return 1;
    }

    printf("max = %d\n", max(x, y));
    printf("avg = %.2f\n", average(x, y));
    return 0;
}

/* ----------- 2. Function bodies (definitions) --------------------- */
static int max(int a, int b)
{
    return (a > b) ? a : b;
}

/* average: we cast one operand to double (or use 2.0) so the
            division happens in floating-point, not integers.        */
static double average(int a, int b)
{
    return (a + b) / 2.0;
}
/* ------------------------------------------------------------------ */
```

---

### 5·2 Build and run

```bash
make              # Makefile from Part 0 compiles math_demo.c
./math_demo
Type two whole numbers: 10 25
max = 25
avg = 17.50
```

---

### 5·3 Small experiment – integer division bite

Change the last line of `average` to:

```c
return (a + b) / 2;     /* ← uses int 2, not 2.0 */
```

Rebuild, run with `10 25` → `avg = 17.00` (wrong—.50 vanished).
The literal `2` is an `int`; integer division truncates decimals.
Put `2.0` back or cast one operand: `(double)(a + b) / 2`.

**Pedagogical pitfall:** New C learners forget the *dot zero*, causing silent loss of precision with no compiler error.

---

### 5·4 Intentional error – missing prototype

Delete the two prototype lines at the top and rebuild:

```
math_demo.c: In function ‘main’:
math_demo.c:15: warning: implicit declaration of function ‘max’
math_demo.c:16: warning: implicit declaration of function ‘average’
/usr/bin/ld: …: undefined reference to `average'
```

*What happened*

* The compiler guessed the return type (`int`) and let the build continue (only a warning).
* The linker later failed because it expected an `int` returning `average` but found `double`.

**Fix** – restore the prototypes; rebuild is clean.

---

### 5·5 Try it – reuse functions in another scenario

Add this code below `main` (or write a new file):

```c
void compare_pairs(int a1,int b1,int a2,int b2)
{
    printf("Pair1: %d,%d  max=%d  avg=%.1f\n",
           a1,b1, max(a1,b1), average(a1,b1));
    printf("Pair2: %d,%d  max=%d  avg=%.1f\n",
           a2,b2, max(a2,b2), average(a2,b2));
}
```

Inside `main`, after reading `x` and `y`, call:

```c
compare_pairs(x, y, x+5, y-3);
```

Build and run. Both `max` and `average` now serve a second caller without extra code.

---

### 5·6 Checkpoint

* You can create prototypes and understand why they stop many link-time surprises.
* You know how return types and parameter types help the compiler catch mistakes.
* You saw integer-division truncation and fixed it with `2.0` or a cast.
* You reused functions in a new context, proving that named blocks beat copy-paste.

Next part: fixed-size collections of many values—**arrays**.

## Part 6 – Lists That Don’t Move: Arrays<a id="part-6--lists-that-dont-move--arrays"></a>

### 6·0 Why this part matters—and what you will watch happen

Imagine you’re scoring a short quiz for eight students.
You don’t want eight separate variables—`score1`, `score2`, … `score8`—that’s unwieldy and error-prone.
An **array** gives you one clean label (`scores`) and lets you index each slot with a number.

Key ideas you’ll meet:

| Concept              | One-line meaning                                                                     |
| -------------------- | ------------------------------------------------------------------------------------ |
| **contiguous**       | Every element sits directly after the previous one in memory—fast for the CPU cache. |
| **zero-based index** | The first slot is position `0`, not `1`.                                             |
| **fixed length**     | The size is chosen at compile time and cannot grow.                                  |

In this part you will

1. Create an array large enough to hold eight quiz scores.
2. Read numbers into the array until the user enters `-1` or the array fills up.
3. Print the highest score and the class average.
4. Deliberately step outside the valid indices, watch the sanitizer catch the bug, and repair it.
5. Finish with two short challenges: reverse printing and searching for a value.

By the end you’ll know how to store many items *side-by-side* and loop over them safely.

---

### 6·1 Write the source file – `scores.c`

```c
#include <stdio.h>
#define MAX 8                 /* array capacity */

/*
   scores.c
   ---------
   * Reads up to MAX quiz scores (integers 0-100).
   * Input ends on -1 or when the array is full.
   * Prints highest score and average.
*/
int main(void)
{
    int scores[MAX];          /* the array itself          */
    int count = 0;            /* how many we actually read */
    int input;

    puts("Enter quiz scores 0-100 (-1 to finish):");

    /* --- 1. Fill the array ---------------------------------------- */
    while (count < MAX && scanf("%d", &input) == 1 && input != -1) {
        if (input < 0 || input > 100) {
            puts("Score must be 0-100.");
            continue;         /* skip invalid value        */
        }
        scores[count++] = input;
    }

    if (count == 0) {
        puts("No scores entered.");
        return 1;
    }

    /* --- 2. Find max and average ---------------------------------- */
    int max = scores[0];
    int sum = scores[0];

    for (int i = 1; i < count; ++i) {
        if (scores[i] > max) max = scores[i];
        sum += scores[i];
    }

    printf("Highest = %d\n", max);
    printf("Average = %.2f\n", (double)sum / count);
    return 0;
}
```

*Line-by-line rationale*

* `int scores[MAX];` reserves eight back-to-back `int` slots in memory.
* `count` tracks how many slots are filled—**never** assume the user enters exactly eight numbers.
* The loop condition `count < MAX` prevents writing past the end (overflow).
* We compute `sum` and `max` in one pass—efficient and simple.

---

### 6·2 Build and run

```bash
make
./scores
Enter quiz scores 0-100 (-1 to finish):
97 84 66 73 -1
Highest = 97
Average = 80.00
```

---

### 6·3 Small experiment – fill the array to capacity

Run again and type eight scores without `-1`, e.g. `10 20 30 40 50 60 70 80`.
After the eighth value the program stops reading because `count` reached `MAX`.

---

### 6·4 Intentional error – out-of-bounds access and sanitizer catch

Add a faulty print loop after the average:

```c
puts("Debug print (with bug):");
for (int i = 0; i <= count; ++i)       /* BUG: should be i < count */
    printf("%d ", scores[i]);
puts("");
```

Re-compile **with the address sanitizer**:

```bash
gcc -Wall -fsanitize=address scores.c -o scores
./scores
```

Enter three numbers.
ASan crashes:

```
ERROR: AddressSanitizer: stack-buffer-overflow on address ...
```

It pinpoints `scores[i]` when `i == count`—we stepped one slot past the valid range.

**Fix**: change the header back to `i < count`, re-build, run: sanitizer is silent.

*Pedagogical pitfall*: writing `<=` instead of `<` is the classic off-by-one that C will not stop at compile time.

---

### 6·5 Try it – two quick challenges

1. **Reverse order**
   After printing the average, add a loop that prints the scores from last entered to first.
   *Hint*: start `i = count-1`, test `i >= 0`, and use `--i`.

2. **Search**
   Prompt for a score to search; loop through the array and print the position if found or say “not present.”
   Make the search stop early once the score is located.

---

### 6·6 Checkpoint

* You can declare a fixed-size array, keep track of how many elements are actually in use, and loop over them.
* You used `double` division to avoid integer truncation in averages.
* You experienced and fixed an out-of-bounds error with the help of the sanitizer.
* You practised reverse traversal and linear search—two patterns you’ll reuse often.

Next part: pass *addresses* instead of copies—**pointers**.



---

## Part 7 – Talking by Address: Pointers<a id="part-7--talking-by-address-pointers"></a>

### 7·0 Why this part matters—and what you will watch happen

Functions have worked only on *copies* so far: `swap(a, b)` changed its private copies, not the originals.
A **pointer** fixes that by carrying the *address* of a variable—its exact street number in memory—so your code can modify the real thing.

| Word                   | Plain meaning                                                      | ASCII cue                   |
| ---------------------- | ------------------------------------------------------------------ | --------------------------- |
| **address**            | The byte position of a value in RAM. Think “box #42 on the shelf.” | `0x7ff…`                    |
| **pointer**            | A variable that stores an address.                                 | an **arrow** in the diagram |
| **`&` (address-of)**   | Ask for the address of a variable.                                 | “Where is this box?”        |
| **`*` (dereference)**  | Go to that address and read or write what’s inside.                | “Open the box.”             |
| **pointer arithmetic** | Add or subtract to move the arrow to the next box.                 | arrow slides right          |

#### 7·0·1 Diagram 1 – two ints and their pointers

```
Memory (addresses grow downward)

0x7ff…3c | 09 00 00 00   <-- y  (value 9)
0x7ff…38 | 05 00 00 00   <-- x  (value 5)
          |
          +-- &x = 0x7ff…38
          +-- &y = 0x7ff…3c
```

`int *a = &x;` means **“put the arrow at box 0x7ff…38.”**
`*a = 42;` means **“open that box and drop 42 inside.”**

---

### 7·1 Code – `pointer_demo.c`

```c
#include <stdio.h>

/* swap: exchanges the values in two memory boxes */
void swap(int *a, int *b)
{
    if (!a || !b) return;            /* NULL guard */
    int tmp = *a;                    /* tmp = value in box A  */
    *a = *b;                         /* box A = value in box B */
    *b = tmp;                        /* box B = tmp           */
}

/* my_strlen: counts bytes until the first '\0' */
size_t my_strlen(const char *s)
{
    const char *p = s;               /* arrow starts at first char */
    while (*p) ++p;                  /* slide until value 0 */
    return (size_t)(p - s);          /* distance travelled */
}

int main(void)
{
    int x = 5, y = 9;
    printf("Before swap: x=%d y=%d\n", x, y);
    swap(&x, &y);                    /* send the arrow */
    printf("After  swap: x=%d y=%d\n", x, y);

    char word[64];
    printf("Type a word: ");
    scanf("%63s", word);
    printf("Length = %zu\n", my_strlen(word));
    return 0;
}
```

---

### 7·2 Build and run (with sanitizer)

```bash
gcc -Wall -g -fsanitize=address pointer_demo.c -o pointer_demo
./pointer_demo
```

---

### 7·3 Diagram 2 – how `my_strlen` walks a C string

Assume you typed **CAT**.

```
Address    Byte  ASCII
---------  ----  -----
0x55…80    43    'C'   <-- s, p
0x55…81    41    'A'
0x55…82    54    'T'
0x55…83    00    '\0'  <-- p stops here
```

Each `++p` moves the arrow down one byte.
When `*p` reads the zero byte, the loop ends.
`p - s` is `3`, which is exactly the string length.

---

### 7·4 Small experiment – print real addresses

Add inside `main`:

```c
printf("&x=%p  &y=%p\n", (void*)&x, (void*)&y);
```

Run again—those hex numbers match the “box labels” in Diagram 1.

---

### 7·5 Intentional error – uninitialised pointer

Insert before `swap`:

```c
int *wild;
printf("wild = %p\n", (void*)wild);
*wild = 123;               /* writes to random address */
```

Compile (with ASan) and run → crash + “SEGV on unknown address”.
**Fix**: always set a pointer to a real address before using `*`.

---

### 7·6 Try it – two pointer challenges

1. **Array walk with pointer maths**

   ```c
   int nums[5] = {1,2,3,4,5};
   int *p = nums;                 /* same as &nums[0] */
   for (int i = 0; i < 5; ++i) {
       printf("%d ", *p);         /* value */
       ++p;                       /* next box */
   }
   ```

2. **Safe string duplicate**

   ```c
   char *my_strdup(const char *s)
   {
       size_t len = my_strlen(s) + 1;
       char *copy = malloc(len);
       if (!copy) return NULL;
       for (size_t i = 0; i < len; ++i) copy[i] = s[i];
       return copy;
   }
   ```

   Call it, print the copy, then `free` it.

---

### 7·7 Checkpoint

* You know the exact meaning of `&` and `*`.
* You can send addresses to functions and modify original variables.
* You saw sanitizer catch null and wild pointers.
* You used pointer arithmetic to walk a string and (optionally) an array.
* ASCII diagrams showed how data and pointers line up in memory.

Next part: build richer data containers with **structs** and **enums**.

## Part 8 – Building Your Own Data Shapes<a id="part-8--building-your-own-data-shapes"></a>

### 8·0 Why this part matters—and what you will watch happen

Primitive types (`int`, `double`, `char`) are single Lego bricks.
A **struct** lets you click several bricks together and give the bundle its own name—perfect for modelling real-world things like a contact card, a 3-D point, or a blockchain block.

An **enum** then assigns readable words to related integer constants, so your code can say `PASS` instead of `2`.

In this part you will

1. Declare a `Contact` struct with two text fields (`name`, `phone`).
2. Fill one instance, print the fields, and peek at the struct’s memory layout.
3. Create an `enum SaveStatus` so functions can return `SAVE_OK` or `SAVE_ERR`.
4. Trigger a “string too long” compiler error by overshooting a fixed array inside the struct, then resize or truncate to fix it.
5. End with challenges: an array of contacts and a function that returns success/failure using the enum.

---

#### 8·0·1 Diagram – struct layout in memory

Suppose we compile on a 64-bit PC where `char` is 1 byte.

```
Contact c = { "Eva", "+331234" };

Address  Byte(s)           Field
-------  ---------------   -----
…00      45 76 61 00 …     name[32]  ("Eva\0..." unused bytes up to 32)
…20      2B 33 33 31 …     phone[16] ("+3312…" up to 16)
```

The struct is just a *flat block* of bytes (total 48 here).
`&c` gives the address of the first byte; `&c.phone` is 32 bytes after that.

---

### 8·1 Write the source file – `contact_demo.c`

```c
#include <stdio.h>
#include <string.h>     /* strncpy */

/* 1. Define a struct that groups two fixed-size strings */
typedef struct {
    char name[32];
    char phone[16];
} Contact;

/* 2. Define an enum for save results */
typedef enum { SAVE_OK, SAVE_IOERR } SaveStatus;

/* save_contact: dummy function that always succeeds */
static SaveStatus save_contact(const Contact *c)
{
    /* Imagine writing to disk here … */
    printf("Saving %s (%s)… ", c->name, c->phone);
    puts("ok");
    return SAVE_OK;
}

int main(void)
{
    Contact me = { "Eva", "+331234" };

    printf("Name : %s\n", me.name);
    printf("Phone: %s\n", me.phone);

    /* 3. Show struct size and field addresses */
    printf("sizeof(Contact) = %zu bytes\n", sizeof me);
    printf("&me        = %p\n", (void*)&me);
    printf("&me.phone  = %p\n", (void*)&me.phone);

    /* 4. Call a function that returns an enum value */
    if (save_contact(&me) != SAVE_OK) {
        puts("Could not save contact.");
        return 1;
    }
    return 0;
}
```

---

### 8·2 Build and run

```bash
make
./contact_demo
```

You’ll see the field values and the memory addresses matching the ASCII diagram.

---

### 8·3 Small experiment – oversize string and compiler help

Change the initialiser:

```c
Contact me = { "A_very_long_name_more_than_31_chars_total", "+331234" };
```

Compile:

```
warning: initializer-string for array of chars is too long
```

The compiler chopped the name to 31 chars + `\0` (silently).
**Fix A:** enlarge the array (`char name[40];`).
**Fix B:** keep the array but truncate safely:

```c
strncpy(me.name, "Really_long_name...", sizeof me.name - 1);
me.name[sizeof me.name - 1] = '\0';
```

---

### 8·4 Intentional error – forgetting NUL terminator

Explicitly fill the array without `\0`:

```c
Contact bad;
memcpy(bad.name, "Bob", 3);    /* no trailing zero */
bad.name[3] = 'X';             /* random byte */
puts(bad.name);                /* undefined behaviour */
```

Run with ASan → may print garbage or crash.
**Fix:** always end the string with `'\0'`.

---

### 8·5 Try it – two struct challenges

1. **Contact list**
   Declare `Contact book[4];`
   Loop over the array, read four contacts from the keyboard, then print them all.

2. **Save & status**
   Modify `save_contact` to fail if the `phone` field is empty (`phone[0] == '\0'`).
   In `main`, check the enum value and return exit code 2 when the save fails.

---

### 8·6 Checkpoint

* You can declare a struct, fill its fields, and access them with dot (`.`) or arrow (`->`) when using pointers.
* You used `sizeof` to see how big the whole struct is and printed field addresses.
* Enums let functions return self-explaining statuses instead of magic numbers.
* You caught and fixed a string-length bug, learning two safe-string strategies.
* You created and checked an enum-based status code.

Next part: memory that can **grow while the program is running**—`malloc`, `realloc`, and `free`.

## Part 9 – Asking the Operating System for Memory<a id="part-9--asking-the-operating-system-for-memory"></a>

### 9·0 Why this part matters—and what you will watch happen

Arrays are great when the length is known at compile-time, but many real tasks don’t have that luxury:

* reading an entire text file (size unknown until you finish),
* growing a list as long as the user keeps typing,
* loading records from a database where tomorrow’s build might need twice as many.

For these cases you ask the operating system for **heap** space *while the program is running*:

| Call      | Job it does                             |
| --------- | --------------------------------------- |
| `malloc`  | “Give me **n** fresh bytes, please.”    |
| `realloc` | “Resize the block you gave me earlier.” |
| `free`    | “I’m done—take these bytes back.”       |

#### 9·0·1 Heap vs. stack – one-screen ASCII mental model

```
High addresses ─┐
                │  HEAP      (grows UP as you malloc)
      malloc →  │  [ new block ]  [ bigger block ]  …
                │
                │  (unused)
                │
                │  STACK     (grows DOWN as you call functions)
                │  |  main  |→|  funcA  |→|  funcB  |
Low addresses  ─┘
```

* **Stack**: automatic, fixed-size frames for each function call.
* **Heap** : flexible area you control with `malloc` / `free`.

Mismanaging the heap—forgetting `free`, freeing twice, or writing past the end—causes leaks and crashes.
We’ll see the sanitizer expose those bugs instantly.

---

### 9·1 Goal of this part

Build a small program that:

1. Reads *any* number of integers (user ends with `-1`).
2. Stores them in a buffer that doubles in size whenever it’s full.
3. Prints them back and then frees the memory.
4. Deliberately leaks memory and double-frees so you can watch AddressSanitizer yell, then fix the mistakes.

---

### 9·2 Write the source file – `grow.c`

```c
#include <stdio.h>
#include <stdlib.h>   /* malloc, realloc, free */

/* Initial capacity for dynamic array */
#define START_CAP 4

int main(void)
{
    int  cap  = START_CAP;                    /* current capacity */
    int  size = 0;                            /* how many used    */
    int *arr  = malloc(cap * sizeof *arr);    /* first heap block */

    if (!arr) { perror("malloc"); return 1; }

    puts("Enter integers (-1 to stop):");
    int value;
    while (scanf("%d", &value) == 1 && value != -1) {

        /* --- Need more space? ------------------------------------------------ */
        if (size == cap) {
            cap *= 2;
            int *tmp = realloc(arr, cap * sizeof *arr);
            if (!tmp) {                       /* realloc can fail */
                perror("realloc");
                free(arr);                    /* avoid leak */
                return 1;
            }
            arr = tmp;                        /* switch to bigger block */
        }
        arr[size++] = value;
    }

    puts("You entered:");
    for (int i = 0; i < size; ++i) printf("%d ", arr[i]);
    puts("");

    /* Free the memory when done */
    free(arr);
    return 0;
}
```

*Why each piece exists*

* `cap`, `size` – separate “capacity” from “count in use”; classic dynamic-array pattern.
* `realloc` path uses a **temporary pointer** so you don’t lose the original block if resizing fails.
* `sizeof *arr` avoids hard-coding `sizeof(int)` and stays correct if the type changes.

---

### 9·3 Build and run with the sanitizer

```bash
gcc -Wall -g -fsanitize=address grow.c -o grow
./grow
```

Enter: `10 20 30 40 50 60 70 -1`
You’ll see the array grow at 4 → 8, then the numbers echo back.
Sanitizer prints nothing—in other words, no leaks or overflows. ✅

---

### 9·4 Intentional error 1 – memory leak and fix

Comment out the `free(arr);`, rebuild, run, then exit:

```
==12345==ERROR: LeakSanitizer: detected memory leaks
Direct leak …  (bytes …)
```

Put `free(arr);` back; leak report vanishes.

---

### 9·5 Intentional error 2 – double free and fix

Add *after* the existing `free`:

```c
free(arr);            /* second time – oops */
```

Rebuild, run:

```
ERROR: AddressSanitizer: attempting double-free ...
```

Delete the extra call; sanitizer is happy again.

---

### 9·6 Small experiment – shrink the buffer

After the print loop, add:

```c
cap = size;                          /* exact fit */
arr = realloc(arr, cap * sizeof *arr);
printf("Shrunk to %d elements\n", cap);
```

Run; sanitizer stays silent, and the message confirms the array now fits precisely.

---

### 9·7 Try it – two challenges

1. **Automatic shrink**
   Modify the reading loop so that if the user deletes numbers (e.g., types `-2` to pop), the program halves the capacity when `size < cap / 4`.

2. **Sum & average**
   After printing all numbers, calculate their sum and average without another loop—store running totals during input to save work.

*(Both force you to think about when to grow/shrink and reuse work already done.)*

---

### 9·8 Checkpoint

* You can request, enlarge, shrink, and release heap memory safely.
* You used a temporary pointer pattern to protect against `realloc` failure.
* AddressSanitizer loudly catches leaks and double-frees—and you fixed both.
* You understand why capacity and size are tracked separately in dynamic arrays.

Next part: Read and write files efficiently, and learn why disks punish byte-at-a-time I/O.

## Part 10 – Files, Streams, and Why Disks Are Slow<a id="part-10--files-streams-and-why-disks-are-slow"></a>

### 10·0 Why this part matters—and what you will see happen

RAM answers in nanoseconds; an SSD answers in microseconds.
That thousand-to-one speed gap punishes any program that chats with the disk one byte at a time.
The C standard library softens the blow: when you call `fread`, it silently hauls in an entire **block** (often 4 KB) and lets your code nibble from that block at RAM speed.
The same trick works in reverse for `fwrite`.

You will:

1. Build a file-copy utility that moves data in 4 KB chunks.
2. Time the copy, then cripple the buffer to one byte and feel the slowdown.
3. See how a single missing **b** in `"rb"` / `"wb"` corrupts binary files on Windows.
4. Optionally bolt on a progress bar.

---

### 10·1 The program – `copy_file.c`

```c
#include <stdio.h>
#include <stdlib.h>

#define BUF_SIZE 4096      /* one disk block; tweak to taste */

/*
   copy_file SOURCE DEST
   --------------------------------------------
   Reads SOURCE in 4-KB slices, writes them to
   DEST unchanged. Works for text and binaries.
*/
int main(int argc, char *argv[])
{
    if (argc != 3) {
        puts("usage: copy_file SOURCE DEST");
        return 1;
    }

    /* "rb"/"wb": open in *binary* mode so Windows
       will not sneak in \r characters.           */
    FILE *in  = fopen(argv[1], "rb");
    FILE *out = fopen(argv[2], "wb");
    if (!in || !out) { perror("fopen"); return 1; }

    char *buf = malloc(BUF_SIZE);            /* one shared buffer */
    if (!buf) { perror("malloc"); return 1; }

    size_t total = 0;                        /* running byte count */
    size_t n;                                /* bytes read each loop */

    while ((n = fread(buf, 1, BUF_SIZE, in)) > 0) {
        if (fwrite(buf, 1, n, out) != n) {   /* partial write = error */
            perror("fwrite");
            fclose(in); fclose(out);
            free(buf);
            return 1;
        }
        total += n;
    }

    printf("Copied %zu bytes\n", total);

    fclose(in);
    fclose(out);
    free(buf);
    return 0;
}
```

**Walk-through**

* We ask `fread` for up to 4 096 bytes. On the last lap it may hand back fewer—`n` keeps us honest.
* `fwrite` must succeed for exactly `n` bytes; if the disk is full it writes less and we bail out.
* A single buffer is reused: no need to allocate per block.
* Every successful `malloc` is matched with one `free`.

---

### 10·2 Build and feel the speed

```bash
gcc -Wall -O2 copy_file.c -o copy_file
time ./copy_file big.iso clone.iso      # fast: 4-KB buffer
```

Now shrink the buffer – change `#define BUF_SIZE 1`, rebuild, copy again:

```bash
time ./copy_file big.iso clone2.iso     # 50–100× slower
```

With a one-byte bucket the program calls the kernel millions of times, and the disk queue drags.

---

### 10·3 A real-world trap on Windows

If you open the files with `"r"` and `"w"` (text mode), Windows silently expands every `\n` to `\r\n` when writing.
Copy a PNG that way and the destination will not open—its size grew.
Use `"rb"` / `"wb"` for all non-text data and you stay safe on every OS.

---

### 10·4 (Extra) Bolt on a progress bar

```c
fseek(in, 0, SEEK_END);           /* jump to end to measure */
long bytes_total = ftell(in);
rewind(in);                       /* back to start */

... inside the loop ...
printf("\r%.1f%%", (total * 100.0) / bytes_total);
fflush(stdout);
```

Because we print once per 4 KB, the bar costs almost nothing.

---

### 10·5 Things that go wrong—and how to spot them

* *Leak*: skip `free(buf)`. Compile with `-fsanitize=address`; sanitizer flags the leak.
* *Partial write*: unplug your USB stick mid-copy—`fwrite` returns short, error path fires.
* *Text vs. binary*: forget the `b`, binary files expand or corrupt only on Windows—hard to catch without testing.

---

### 10·6 Checkpoint

* You can move files safely with block I/O.
* You understand why a big buffer crushes byte-by-byte loops in speed.
* You know how to open files in the correct mode and check every I/O call.
* You saw how to time a program and add user-friendly progress output.

Next part: break the codebase into multiple `.c` / `.h` files and let **`make` recompile only what you edit.**

## Part 11 – Many-File Projects, Sub-Folders, and Why `make` Matters<a id="part-11--many-file-projects-and-why-make-matters"></a>

### 11·0 Why we’re reorganising the code —and what you’ll see happen

One huge `.c` file turns into a scroll-fest as soon as a project passes a few hundred lines.
The professional habit is to put each **module** in its own directory, split every module into

* a **header** (`.h`) that says *what* the module offers, and
* a **source** file (`.c`) that shows *how* it works,

and let **`make` rebuild only the pieces you touched**.

To make that concrete we’ll implement a **stack** module.

> #### What’s a stack, and why pick it?
>
> A **stack** is the simplest “last-in, first-out” container: push new items on top, pop them off in reverse order.
>
> ```
>   push(30)           pop() → 30
>   ┌───┐              pop() → 20
>   │30 │              pop() → 10
>   │20 │
>   │10 │
>   └───┘
> ```
>
> *Why it’s the perfect example*
>
> * **Tiny API** — just `push`, `pop`, `stack_empty`; short enough to fit in a header.
> * **Private guts** — the backing array and counter should stay hidden: great for showing the `static` keyword and include-guards.
> * **Easy to test** — three `assert` calls prove it works.
> * **Familiar idea** — the CPU’s own call stack behaves exactly the same, so the concept sticks.

---

## 11·1 Folder layout after the split

```
c_course/
├─ Makefile
├─ src/
│   └─ main.c
└─ stack/
    ├─ stack.h
    └─ stack.c
```

*`src/` holds programs that **use** the stack; `stack/` owns the stack itself.*

---

## 11·2 Makefile — one tweak to handle sub-folders

```make
# ---------- Makefile aware of sub-folders ---------------------------
CC      = gcc
CFLAGS  = -Wall -Wextra -std=c17 -g

# find .c files in src/ and stack/
SRC_FILES := $(shell find src stack -name '*.c')
BINARIES  := $(SRC_FILES:.c=)

all: $(BINARIES)

# pattern:   path/foo   ←   path/foo.c
%: %.c
	$(CC) $(CFLAGS) $< -o $@

clean:
	rm -f $(BINARIES)
# --------------------------------------------------------------------
```

*The wildcard rule still works: `%` now absorbs the folder name.*

---

## 11·3 `stack/stack.h` — the public interface

```c
#ifndef STACK_H        /* include-guard begins */
#define STACK_H

/* A fixed-capacity LIFO stack for ints */
void push(int value);
int  pop(void);
int  stack_empty(void);

#endif                 /* STACK_H */
```

---

## 11·4 `stack/stack.c` — the private implementation

```c
#include "stack.h"
#include <stdio.h>

#define CAP 16              /* compile-time capacity */
static int data[CAP];       /* hidden storage */
static int top = 0;         /* index of next free slot */

/* push: add value if space remains */
void push(int v)
{
    if (top == CAP) { puts("stack full"); return; }
    data[top++] = v;
}

/* pop: return last pushed value, or 0 if empty */
int pop(void)
{
    if (top == 0) { puts("stack empty"); return 0; }
    return data[--top];
}

int stack_empty(void)
{
    return top == 0;
}
```

*`static` gives the array and counter **internal linkage**—no other file can see them.*

---

## 11·5 `src/main.c` — the client program

```c
#include <stdio.h>
#include "../stack/stack.h"    /* path to the interface */

int main(void)
{
    push(10);
    push(20);
    while (!stack_empty())
        printf("%d ", pop());
    puts("");
    return 0;
}
```

*(When projects grow you’ll move public headers to `include/` and add `-Iinclude`; for now a relative path keeps it obvious.)*

---

## 11·6 Build and run

```bash
make               # compiles stack/stack.c and src/main.c
./src/main         # prints: 20 10
```

Observe the console lines: only two compiles plus two links (one per `.c`).

---

## 11·7 Watch incremental rebuilds save time

```bash
echo "//touch" >> stack/stack.c   # modify implementation
time make
```

Output shows **only** `stack/stack.c` recompiling plus its link; `src/main.c` was untouched.
On large codebases this trims minutes off each build.

---

## 11·8 Intentional error — remove the include-guard

Delete `#ifndef/#define/#endif` lines in `stack.h`.
Add a double include inside `stack.c`:

```c
#include "stack.h"
#include "stack.h"
```

Run `make`:

```
error: redefinition of ‘data’
```

The compiler saw two copies of every declaration. Restore the guard; build succeeds.

*Lesson: include-guards stop accidental duplicate inclusion.*

---

## 11·9 Challenge — add a quick unit test

*Directory* `tests/` → `tests/test_stack.c`

```c
#include <assert.h>
#include "../stack/stack.h"

int main(void)
{
    push(1); push(2);
    assert(pop() == 2);
    assert(pop() == 1);
    assert(stack_empty());
    return 0;
}
```

No Makefile edits required: `find` picks up the new `.c` file automatically.

```bash
make
./tests/test_stack   # silent success
echo $?              # prints 0
```

---

### 11·10 Checkpoint

* You split interface (`.h`) from implementation (`.c`) and stored them in a dedicated folder.
* Include-guards prevented duplicate definitions; removing them produced a clear compiler error.
* `static` kept private data truly private inside `stack.c`.
* Your Makefile now searches sub-folders and recompiles **only** what changed.
* You added a unit test without touching build scripts—pattern rules did the lifting.

Next part: hunt bugs at run-time—step into the debugger and let sanitizers *instantly* expose memory errors.

## Part 12 – When Programs Misbehave: Debuggers & Sanitizers<a id="part-12--when-programs-misbehave-debuggers--sanitizers"></a>

### 12·0 Why this part matters—and what you will watch happen

A compiler checks *syntax*; the real disasters strike **at run-time**:

* write one byte past the end of an array—crash tomorrow;
* forget to `free` memory in a loop—memory leak slows the server next week;
* divide by zero or shift a negative number—undefined behaviour, results vary by compiler flag.

Modern toolchains give you two safety nets:

| Tool                                                       | What it does                                                   | When you use it                                         |
| ---------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------- |
| **Debugger** (`gdb`, LLDB)                                 | Pause a running program, inspect variables, step line-by-line. | Hunting *logical* mistakes (“why is `sum` negative?”).  |
| **Sanitizers** (`-fsanitize=address`, `undefined`, `leak`) | Abort instantly on illegal memory or math.                     | Catching *memory* and *UB* bugs the moment they happen. |

You’ll:

1. Run a deliberately buggy program; watch AddressSanitizer crash exactly at the bad line.
2. Fix the bug; sanitizer runs silent.
3. Step through the same program in a debugger—set breakpoints, inspect variables, print the call stack.
4. Trigger a use-after-free and see sanitizer name the fault.
5. Finish with a leak-detection challenge.

---

### 12·1 The naughty program – `oob.c`

```c
#include <stdio.h>

/* Purposefully writes past the end of a 3-element array */
int main(void)
{
    int a[3] = {1, 2, 3};

    for (int i = 0; i <= 3; ++i)     /* BUG: should be i < 3 */
        a[i] *= 10;

    for (int i = 0; i < 3; ++i)
        printf("%d ", a[i]);
    puts("");                        /* never reached with sanitizer */
    return 0;
}
```

---

### 12·2 Compile with AddressSanitizer and run

```bash
gcc -Wall -g -fsanitize=address oob.c -o oob
./oob
```

Output (trimmed):

```
==12345==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7ffee...
    #0 0x... in main oob.c:6
        6   a[i] *= 10;   /* writes to a[3] which doesn’t exist */
```

The program aborts **at the first illegal write**—no more silent corruption.

> **Note**: Without `-fsanitize=address` you’d probably get no warning, or a crash many lines later.

---

### 12·3 Fix the bug, re-run clean

In the loop header change `<=` to `<`:

```c
for (int i = 0; i < 3; ++i)
```

Recompile & run:

```
1 20 30
```

Sanitizer prints nothing—silence is success.

---

### 12·4 Quick guided tour of `gdb` (the GNU debugger)

1. **Compile with symbols** (already done: `-g`).

2. Start debugger:

   ```bash
   gdb ./oob
   ```

3. Inside `gdb`:

   ```gdb
   (gdb) break main     # pause on entry
   (gdb) run
   (gdb) next           # step to first statement
   (gdb) print i        # show current loop index
   (gdb) display a[0]   # auto-print after every step
   (gdb) cont           # run to end
   (gdb) bt             # backtrace call stack
   (gdb) quit
   ```

Why this matters:

* **break** lets you stop before trouble.
* **next / step** walk through code, watching variable changes.
* **bt** shows the path of calls that led here—vital for multi-file projects.

---

### 12·5 Intentional error 2 – use-after-free

Append to `oob.c`:

```c
#include <stdlib.h>
...
int *p = malloc(4);
free(p);
*p = 7;                 /* use after free */
return 0;
```

Recompile with **two** sanitizers:

```bash
gcc -Wall -g -fsanitize=address,undefined oob.c -o oob
./oob
```

Output ends with:

```
ERROR: AddressSanitizer: heap-use-after-free on address ...
```

Delete or move the bad write **before** `free(p);`—sanitizer runs clean.

---

### 12·6 Challenge – leak detector

Remove the `free(p);`, compile with `-fsanitize=leak` added:

```bash
gcc -Wall -g -fsanitize=leak,address oob.c -o oob
./oob
```

Sanitizer prints a leak report naming the allocation site.
Re-add `free(p);`—report disappears.

---

### 12·7 Checkpoint

* You saw AddressSanitizer stop a buffer overflow **at the exact line**.
* You fixed the boundary check and verified the fix with the same tool.
* You stepped through code in `gdb`, inspecting live variables and the call stack.
* You triggered and fixed a use-after-free and a memory leak.
* You now have a repeatable recipe: **compile with sanitizers first, then debug the logic.**

This defensive layer means most crashes will shout their line number the very first time they happen—before they sneak into production.

Final project ahead: tie every concept—functions, structs, dynamic memory, files, and safety tools—into a tiny blockchain.


---

## The Most Secure Communication System — *Lite Edition* <a id="the-most-secure-communication-system"></a>

### What you’ll build

* **Cipher** – plain old *Caesar* shift (A→D, B→E, …).
* **Shift value** – calculated at run-time from a user-typed *mystic date* plus the length of a text key:

```
shift =  (day + month + year + strlen(key))  % 26
```

* **Puzzle gate** – decoding prints three Taiwanese–tea riddles; only after all three answers are right does plaintext appear.

---

### Folder layout and Makefile

```
secure_memo/
├─ Makefile
└─ src/
    ├─ crypto.h / crypto.c      (3 TODOs)
    ├─ puzzle.h / puzzle.c      (1 TODO)
    └─ main.c                   (no TODOs)
```

**Makefile**

```make
CC      = gcc
CFLAGS  = -Wall -Wextra -std=c17 -O2 -g
SRC := $(shell find src -name '*.c')
BIN := $(SRC:.c=)
all: $(BIN)
%: %.c
	$(CC) $(CFLAGS) $< -o $@
clean: ; rm -f $(BIN)
```

---

## 1 Crypto helper (3 tiny TODOs)

### 1·1 File: `src/crypto.h`

```c
#ifndef CRYPTO_H
#define CRYPTO_H
#include <stddef.h>

/* ----- STEP 1 -----------------------------------------------------
   Turn "DD/MM/YYYY" + key-string into a Caesar shift 0-25.
   Returns -1 if the date string is malformed.
------------------------------------------------------------------- */
int  date_to_shift(const char *date, const char *key);

/* Encode / decode in-place copies
   out must be large enough for terminating '\0'.                  */
void caesar_encode(const char *in, int shift, char *out);   /* STEP 2 */
void caesar_decode(const char *in, int shift, char *out);   /* STEP 3 */

#endif
```

### 1·2 File: `src/crypto.c`

```c
#include "crypto.h"
#include <stdio.h>
#include <ctype.h>
#include <string.h>

/* =======================  STEP 1 ================================= */
int date_to_shift(const char *date, const char *key)
/*
   TODO – detailed guidance:
   1. Use sscanf(date, "%d/%d/%d", &d,&m,&y). If it returns 3 numbers
      and all ranges look sane (1-31, 1-12, 1000-9999) continue,
      else return -1.                                (≈5 lines)
   2. int sum = d + m + y + (int)strlen(key);
   3. return sum % 26;                               (1 line)
*/
{
    /* Your code here */
    return -1;
}

/* =======================  STEP 2 ================================= */
void caesar_encode(const char *in, int shift, char *out)
/*
   Encode rule:
     – Uppercase A-Z rotate; keep non-letters unchanged.
     – Use ((ch-'A') + shift) % 26 + 'A'.
   Loop over *in until '\0', write to *out, terminate '\0'.  (≈8 lines)
*/
{
    /* Your code here */
}

/* =======================  STEP 3 ================================= */
void caesar_decode(const char *in, int shift, char *out)
/*
   Simply call caesar_encode with (26-shift)%26, or write mirror loop.
*/
{
    /* Your code here */
}
```

---

## 2 Puzzle gate (1 tiny TODO)

### 2·1 File: `src/puzzle.h`

```c
#ifndef PUZZLE_H
#define PUZZLE_H
/* Returns 1 if the user answers all three riddles correctly. */
int solve_tea_puzzle(void);
#endif
```

### 2·2 File: `src/puzzle.c`

```c
#include "puzzle.h"
#include <stdio.h>
#include <string.h>
#include <ctype.h>

/* =============  STEP 4 (only one, small) ========================= */
int solve_tea_puzzle(void)
/*
   Riddle 1 – “Which Taiwanese county grows famous Alishan tea?”
              correct: chiayi
   Riddle 2 – “Blockchain blocks are linked by ______ hashes.”
              correct: cryptographic
   Riddle 3 – “Enter any single digit that is *not* even.”
              accept: 1,3,5,7,9
   For each:
     – print the question,
     – read a word or digit with scanf,
     – compare case-insensitively,
     – if wrong => puts("fail") and return 0 immediately.
   If all three pass, return 1.
*/
{
    /* Your code here */
    return 0;
}
```

---

## 3 Command-line front end (finished, nothing to edit)

`src/main.c`

```c
#include <stdio.h>
#include <string.h>
#include "crypto.h"
#include "puzzle.h"

static void encode_cmd(const char *date,const char *key,const char *msg)
{
    int shift = date_to_shift(date,key);
    if(shift<0){ puts("bad date"); return; }

    char buf[512];
    caesar_encode(msg,shift,buf);
    puts(buf);
}

static void decode_cmd(const char *date,const char *key,const char *code)
{
    int shift = date_to_shift(date,key);
    if(shift<0){ puts("bad date"); return; }

    if(!solve_tea_puzzle()){ puts("🔒 access denied"); return; }

    char buf[512];
    caesar_decode(code,shift,buf);
    printf("📜 %s\n",buf);
}

int main(int c,char* v[])
{
    if(c<5){ puts("usage: memo <enc|dec> <DD/MM/YYYY> <key> <text>"); return 1; }
    if(!strcmp(v[1],"enc")) encode_cmd(v[2],v[3],v[4]);
    else if(!strcmp(v[1],"dec")) decode_cmd(v[2],v[3],v[4]);
    else puts("bad cmd");
    return 0;
}
```

---

## 4 How to finish

1. **Implement STEP 1** – compile, run
   `./src/date_test enc 03/01/1992 SKY TEST` (you can write a one-liner test).
2. **Implement STEP 2 & STEP 3** – ensure encode then decode returns original text.
3. **Implement STEP 4** – run decode, fail twice, succeed third time.

```bash
make
./src/main enc 03/01/1992 SKY "HELLO MARS"
# prints coded text, e.g. KHOOR PDUV
./src/main dec 03/01/1992 SKY "KHOOR PDUV"
# asks three riddles; answer chiayi, cryptographic, 5
# reveals HELLO MARS
```

---

### You’re done when

* Bad date → program complains.
* Wrong puzzle answer → plaintext stays locked.
* Right answers → shows original message.
* `-Wall` emits no warnings and sanitizers (`-fsanitize=address`) stay quiet.

Short, self-contained, yet it exercises: parsing, modular arithmetic, character rotation, loops, scanf-input handling, string comparison—and a dash of Taiwanese tea lore to keep it fun.


Below is the **complete, stand-alone C course**—just copy this file beside VS Code and work through it.
Every part follows the same learning loop so you always know what’s coming next.

---

# COURSE MAP

| Part | Title                   | Core Ideas                       | Runtime\* |
| ---- | ----------------------- | -------------------------------- | --------- |
| 0    | Your Playground         | compiler, linker, Makefile       | 40 min    |
| 1    | Talking to the Computer | `main`, `#include`, statement    | 1 h       |
| 2    | Remembering Things      | variables, types, basic I/O      | 1 h 15    |
| 3    | Making Decisions        | booleans, comparison, branches   | 1 h 15    |
| 4    | Doing Things Again      | loops, counters, `break`         | 1 h 30    |
| 5    | Organising Work         | functions, scope                 | 1 h       |
| 6    | Lists of Data           | arrays, bounds, off-by-one       | 2 h       |
| 7    | Power Tool — Pointers   | address, dereference, `const`    | 2 h       |
| 8    | Custom Shapes           | structs, enums, `typedef`        | 2 h       |
| 9    | Memory You Control      | heap, `malloc`/`free`, `realloc` | 2 h       |
| 10   | Talking to Files        | streams, buffering               | 1 h 30    |
| 11   | Splitting a Program     | headers, compilation units       | 1 h 30    |
| 12   | Finding Bugs            | `gdb`, sanitizers                | 1 h       |
| 13   | Capstone Project        | full program                     | 5-6 h     |

\*Hands-on time, not skimming.

---

## HOW TO USE THIS DOCUMENT

1. **Type every line by hand**—muscle memory matters.
2. After each **Try-It Task**, answer the **Knowledge Check** aloud.
3. Only then peek at **Appendix A** for the solution snippet.
4. Keep all code under git; commit after each part.

---

# PART 0 YOUR PLAYGROUND

### Concept Box

| Term     | Plain English                       | 中文(繁體) + pinyin |
| -------- | ----------------------------------- | --------------- |
| compiler | turns text into machine code        | 編譯器 biān-yì-qì  |
| linker   | glues pieces into one binary        | 連結器 lián-jié-qì |
| Makefile | script for rebuilding automatically | Makefile, 無官方譯名 |

### Walk-through

1. Create folder `c_course`.
2. Add a **Makefile**:

```make
CFLAGS := -Wall -Wextra -std=c17 -g
SRC := $(wildcard *.c)
BIN := $(SRC:.c=)

all: $(BIN)
%: %.c ; $(CC) $(CFLAGS) $< -o $@
clean: ; rm -f $(BIN)
```

3. Run `make` ⇒ “Nothing to be done”.

### Common Error Decoder

| Message                            | Meaning       | Fix                  |
| ---------------------------------- | ------------- | -------------------- |
| `make: *** No rule to make target` | Filename typo | Check file extension |
| `CC: command not found`            | No compiler   | Install gcc/clang    |

### Try-It Task ☑

Create empty `scratch.c`, run `make`, verify an executable `scratch` appears.

### Stretch Goal 💪

Add `-fsanitize=address,undefined` to `CFLAGS`; rebuild.

### Knowledge Check ⚡

1. What does `$(wildcard *.c)` expand to?
2. Why is `clean` a separate rule?
3. What file timestamps does `make` compare?

### Reflection 💭

Explain in one sentence why the first `make` did nothing.

---

# PART 1 TALKING TO THE COMPUTER

### Concept Box

| Term      | English         | 中文(繁體) + pinyin |
| --------- | --------------- | --------------- |
| function  | reusable block  | 函式 hán-shì      |
| statement | one action line | 陳述句 chén-shù-jù |

### Walk-through

```c
// hello.c
#include <stdio.h>

int main(void)            // entry point
{
    puts("Hello, world!");
    return 0;             // 0 = success
}
```

Compile: `make hello && ./hello`.

Delete `#include <stdio.h>`, rebuild—read the error, restore it.

### Common Error Decoder

| Message                                   | Cause            | Fix                      |
| ----------------------------------------- | ---------------- | ------------------------ |
| `implicit declaration of function 'puts'` | missing header   | add `#include <stdio.h>` |
| `expected ';'`                            | missed semicolon | add `;`                  |

### Try-It Task ☑

Print a second line with your name and a third showing `2 + 2 = 4`.

### Stretch Goal 💪

Return `42` instead of `0`; run `echo $?` to see exit status.

### Knowledge Check ⚡

1. Why must every statement end with `;`?
2. What value signals success to the OS?
3. Where does execution begin in a C program?

### Reflection 💭

Tell a friend why `return 0;` is included even though the program “already finished”.

---

# PART 2 REMEMBERING THINGS

### Concept Box

| Term     | Meaning            | 中文(繁體) + pinyin |
| -------- | ------------------ | --------------- |
| variable | named storage      | 變數 biàn-shù     |
| type     | size + allowed ops | 型別 xíng-bié     |

Common primitive sizes (64-bit PC):

| Type     | Bytes | Example literal |
| -------- | ----- | --------------- |
| `int`    | 4     | `42`            |
| `double` | 8     | `3.14`          |
| `char`   | 1     | `'A'`           |

### Walk-through

```c
#include <stdio.h>

int main(void)
{
    int    age   = 25;
    double price = 2.99;
    char   initial = 'S';

    printf("age=%d price=%.2f initial=%c\n",
           age, price, initial);
}
```

Change `age` to `age + 1`, rebuild.

### Common Error Decoder

| Message                                             | Cause             | Fix                    |
| --------------------------------------------------- | ----------------- | ---------------------- |
| `format ‘%f’ expects argument of type ‘double’`     | wrong placeholder | use matching specifier |
| `warning: overflow in implicit constant conversion` | literal too big   | choose larger type     |

### Try-It Task ☑

Ask user for a radius, print circle area (`πr²`); use `double`.

**Hints** — `#define PI 3.141592653589793`, `scanf("%lf", &r)`.

### Stretch Goal 💪

Compute volume of a cylinder: ask for radius & height.

### Knowledge Check ⚡

1. Which specifier prints an `int`?
2. What operator gets remainder?
3. What happens if you divide two `int` values?

### Reflection 💭

Why can’t the compiler guess the variable type for you?

---

# PART 3 MAKING DECISIONS

### Concept Box

| Term    | Meaning                 | 中文(繁體) + pinyin |
| ------- | ----------------------- | --------------- |
| boolean | 0 = false, non-0 = true | 布林值 bù-lín-zhí  |
| branch  | choose path             | 分岔 fēn-chà      |

### Walk-through

```c
// even_or_odd.c
#include <stdio.h>

int main(void)
{
    int n;

    printf("Enter a number: ");
    if (scanf("%d", &n) != 1) {
        fprintf(stderr, "Bad input\n");
        return 1;
    }

    if (n % 2 == 0) {
        puts("even");
    } else {
        puts("odd");
    }
    return 0;
}
```

### Common Error Decoder

| Message                                 | Cause                | Fix               |
| --------------------------------------- | -------------------- | ----------------- |
| `unused variable`                       | declared, never used | delete or use     |
| `suggest parentheses around assignment` | used `=` in `if`     | replace with `==` |

### Try-It Task ☑

Output one of **four** strings: `negative even`, `negative odd`, `positive even`, `positive odd`.

### Stretch Goal 💪

Implement same logic using `switch` on `(n < 0)*2 + (n%2==0)`.

### Knowledge Check ⚡

1. Why does C allow `if(5)`?
2. Write a condition for “multiple of 3 AND 5”.
3. What is the value of `!0` and `!7`?

### Reflection 💭

Explain why assignments inside `if` are dangerous.

---

# PART 4 DOING THINGS AGAIN

### Concept Box

| Term      | Meaning     | 中文(繁體) + pinyin |
| --------- | ----------- | --------------- |
| loop      | repeat code | 回圈 huí-quān     |
| iteration | one pass    | 迭代 dì-dài       |

### Walk-through — FizzBuzz Skeleton

```c
for (int i = 1; i <= 100; ++i)
{
    if (i % 15 == 0)      puts("FizzBuzz");
    else if (i % 3 == 0)  puts("Fizz");
    else if (i % 5 == 0)  puts("Buzz");
    else                  printf("%d\n", i);
}
```

### Common Error Decoder

| Message       | Cause                 | Fix            |
| ------------- | --------------------- | -------------- |
| infinite loop | condition never false | update counter |
| off-by-one    | wrong `<` vs `<=`     | check range    |

### Try-It Task ☑

Read upper limit from command-line (`argv[1]` via `atoi`) and run FizzBuzz up to that.

### Stretch Goal 💪

Print FizzBuzz *horizontally* separated by spaces, not newlines.

### Knowledge Check ⚡

1. What are the three fields of a `for` loop?
2. Difference between `while` and `do { } while`?
3. What does `continue` do?

### Reflection 💭

Why are off-by-one errors so common in loops?

---

# PART 5 ORGANISING WORK (FUNCTIONS)

### Concept Box

| Term         | Meaning     | 中文(繁體) + pinyin   |
| ------------ | ----------- | ----------------- |
| parameter    | input value | 參數 cān-shù        |
| return value | output      | 回傳值 huí-chuán-zhí |
| scope        | visibility  | 作用域 zuò-yòng-yù   |

### Walk-through — Maximum of Two

```c
static int max(int a, int b)
{
    return (a > b) ? a : b;
}
```

Call from `main`; print result.

### Common Error Decoder

| Message             | Cause              | Fix                       |
| ------------------- | ------------------ | ------------------------- |
| `conflicting types` | prototype mismatch | make signatures identical |
| `unused parameter`  | forgot to use it   | remove or use             |

### Try-It Task ☑

Write `double average(int a, int b)`; test with user input.

### Stretch Goal 💪

Write `int clamp(int val,int min,int max)`.

### Knowledge Check ⚡

1. Are parameters passed by value or reference in C?
2. What keyword hides a function inside its file?
3. Can a function return multiple values?

### Reflection 💭

Explain why copying parameters protects the caller.

---

# PART 6 LISTS OF DATA (ARRAYS)

### Concept Box

| Term    | Meaning           | 中文(繁體) + pinyin |
| ------- | ----------------- | --------------- |
| array   | fixed-length list | 陣列 zhèn-liè     |
| index   | position          | 索引 suǒ-yǐn      |
| element | item              | 元素 yuán-sù      |

### Walk-through — Sum of 5

```c
int scores[5] = {3,4,2,5,1};
int sum = 0;
for (int i = 0; i < 5; ++i)
    sum += scores[i];
printf("total=%d\n", sum);
```

### Common Error Decoder

| Message                               | Cause                           | Fix            |
| ------------------------------------- | ------------------------------- | -------------- |
| segmentation fault                    | index out of range              | check bounds   |
| `initializer element is not constant` | variable length in global scope | move to `main` |

### Try-It Task ☑

Ask user `n` (≤100), read `n` ints into array, print max.

### Stretch Goal 💪

Sort the array with bubble sort; print sorted list.

### Knowledge Check ⚡

1. Why must the length be known at compile time?
2. What is the first valid index?
3. What happens if you access index `-1`?

### Reflection 💭

Draw a memory diagram of an array of 4 ints on paper.

---

# PART 7 POWER TOOL — POINTERS

### Concept Box

| Term        | Meaning              | 中文(繁體) + pinyin |
| ----------- | -------------------- | --------------- |
| pointer     | memory address       | 指標 zhǐ-biāo     |
| dereference | `*p` gives the value | 反參考 fǎn-cān-kǎo |
| `const`     | read-only promise    | 常數 cháng-shù    |

### Walk-through — Swapping Two Ints

```c
void swap(int *a, int *b)
{
    int tmp = *a;
    *a = *b;
    *b = tmp;
}
```

Test:

```c
int x=5,y=7;
swap(&x,&y);
printf("%d %d\n",x,y);   // 7 5
```

### Common Error Decoder

| Message                                                 | Cause              | Fix                  |
| ------------------------------------------------------- | ------------------ | -------------------- |
| segmentation fault                                      | dereferenced NULL  | check pointer        |
| `warning: assignment to ‘int *’ from incompatible type` | wrong pointer type | cast or correct type |

### Try-It Task ☑

Implement `size_t my_strlen(const char *s)` without `<string.h>`.

### Stretch Goal 💪

Write `char *my_strdup(const char *s)` using `malloc`.

### Knowledge Check ⚡

1. What’s the type of `&array[0]`?
2. How do you increment a pointer to go to next int?
3. Why is `const char *` safer than `char *` for strings?

### Reflection 💭

Draw stack vs heap for `int *p = malloc(sizeof *p);`.

---

# PART 8 CUSTOM SHAPES (STRUCTS & ENUMS)

### Concept Box

| Term    | Meaning         | 中文(繁體) + pinyin        |
| ------- | --------------- | ---------------------- |
| struct  | grouped fields  | 結構 jié-gòu             |
| enum    | named constants | 列舉 liè-jǔ              |
| typedef | alias           | 型別別名 xíng-bié bié-míng |

### Walk-through — Player Record

```c
typedef struct {
    char name[32];
    int  score;
} Player;
```

Create two players, print via loop.

### Common Error Decoder

| Message                                 | Cause          | Fix                    |
| --------------------------------------- | -------------- | ---------------------- |
| `initializer-string for array too long` | string > field | enlarge array          |
| unexpected padding                      | assume packed  | use `sizeof` not guess |

### Try-It Task ☑

Write `enum Color { RED, GREEN, BLUE };` and `const char *color_name(enum Color);`.

### Stretch Goal 💪

Linked list of `Player` nodes: add, remove, print.

### Knowledge Check ⚡

1. What keyword turns `struct Player` into `Player`?
2. Why might `sizeof(Player)` be 40 not 36?
3. How do you access struct field via pointer?

### Reflection 💭

Explain alignment padding in one sentence.

---

# PART 9 MEMORY YOU CONTROL (DYNAMIC ALLOCATION)

### Concept Box

| Term     | Meaning            | 中文(繁體) + pinyin       |
| -------- | ------------------ | --------------------- |
| heap     | long-lived storage | 堆積 duī-jī             |
| `malloc` | request bytes      | 動態分配 dòng-tài fēn-pèi |
| `free`   | give back          | 釋放 shì-fàng           |

### Walk-through — Growable Buffer

```c
size_t cap = 16, len = 0;
char *buf = malloc(cap);

int ch;
while ((ch = getchar()) != EOF) {
    if (len == cap) {
        cap *= 2;
        buf = realloc(buf, cap);
    }
    buf[len++] = ch;
}
buf[len] = '\0';           // NUL-terminate
```

### Common Error Decoder

| Message     | Cause       | Fix                 |
| ----------- | ----------- | ------------------- |
| double free | freed twice | set pointer to NULL |
| memory leak | never freed | add `free(buf)`     |

### Try-It Task ☑

Turn the code into `char *read_file(const char *path)` returning NUL-terminated buffer.

### Stretch Goal 💪

Implement dynamic array `dynarray_push`, `dynarray_free`.

### Knowledge Check ⚡

1. What does `realloc` do if size gets bigger?
2. What happens if `malloc` fails?
3. Why add `\0` at end of buffer?

### Reflection 💭

Describe “dangling pointer” in your own words.

---

# PART 10 TALKING TO FILES

### Concept Box

| Term   | Meaning           | 中文(繁體) + pinyin   |
| ------ | ----------------- | ----------------- |
| stream | `FILE*` handle    | 資料流 zī-liào-liú   |
| buffer | temporary storage | 緩衝區 huǎn-chōng-qū |

### Walk-through — Copy File

```c
#define CHUNK 4096
char buf[CHUNK];
size_t n;
while ((n = fread(buf, 1, CHUNK, src)) > 0) {
    if (fwrite(buf, 1, n, dst) != n) {
        perror("write");
        return 1;
    }
}
```

### Common Error Decoder

| Message              | Cause               | Fix                 |
| -------------------- | ------------------- | ------------------- |
| `Segmentation fault` | `src` or `dst` NULL | check `fopen`       |
| partial write        | disk full           | handle return value |

### Try-It Task ☑

Add CRC-32 checksum during copy; print checksum on completion.

### Stretch Goal 💪

Implement simple progress bar (% complete).

### Knowledge Check ⚡

1. Difference between `"rb"` and `"r"` modes?
2. Why is I/O buffered?
3. Which function flushes a stream?

### Reflection 💭

Explain what happens when you forget `fclose`.

---

# PART 11 SPLITTING A PROGRAM

### Concept Box

| Term             | Meaning             | 中文(繁體) + pinyin       |
| ---------------- | ------------------- | --------------------- |
| header           | declarations (`.h`) | 標頭檔 biāo-tóu dǎng     |
| compilation unit | one `.c` file       | 編譯單元 biān-yì dān-yuán |

### Walk-through — Header Guard

```c
#ifndef LIST_H
#define LIST_H
/* declarations */
#endif
```

### Common Error Decoder

| Message                | Cause                      | Fix                          |
| ---------------------- | -------------------------- | ---------------------------- |
| multiple definition    | same function in two files | `static` or move to one file |
| `implicit declaration` | forgot header              | include `.h`                 |

### Try-It Task ☑

Refactor linked-list into `list.h` / `list.c`; keep `main.c` small.

### Stretch Goal 💪

Add unit test file `test_list.c` compiled separately.

### Knowledge Check ⚡

1. Why mark helpers `static` in `.c`?
2. What does `-c` flag do for the compiler?
3. How does `make` detect which files changed?

### Reflection 💭

Explain in 30 seconds how header guards prevent double inclusion.

---

# PART 12 FINDING BUGS

### Concept Box

| Term      | Meaning             | 中文(繁體) + pinyin |
| --------- | ------------------- | --------------- |
| debugger  | runtime inspector   | 除錯器 chú-cuò-qì  |
| sanitizer | runtime UB detector | 檢測器 jiǎn-cè-qì  |

### Walk-through — Using gdb

```bash
gdb ./prog
(gdb) b main
(gdb) r
(gdb) n        # next
(gdb) p var
(gdb) bt       # backtrace
```

Compile with `-fsanitize=address,undefined` and run out-of-bounds write; see report.

### Common Error Decoder

| Message                    | Cause          | Fix       |
| -------------------------- | -------------- | --------- |
| ASan: heap-buffer-overflow | wrote past end | fix index |
| UBSan: division by zero    | denominator 0  | add guard |

### Try-It Task ☑

Use gdb to step through your `my_strlen`, inspect `*s`, `p-s`.

### Stretch Goal 💪

Enable `-fsanitize=leak` and check for leaks in previous parts.

### Knowledge Check ⚡

1. What flag adds debug symbols?
2. Which sanitizer catches use-after-free?
3. How to set a breakpoint at function `swap`?

### Reflection 💭

Describe how sanitizers make UB *fail fast*.

---

# PART 13 CAPSTONE PROJECT

Choose ONE:

### Option A — **Text Metrics**

Re-implement Unix `wc` plus:

* top-10 most frequent words (case-insensitive)
* average line length
* handles huge files with buffered reading

### Option B — **Tic-Tac-Toe AI**

* 3×3 board, CLI interface
* Minimax with pruning
* Clear draw/lose/win detection

### Option C — **HTTP GET Client**

* Raw BSD sockets
* Parses HTTP 1.0/1.1, follows 301 redirect once
* Saves body to file, prints headers

#### Requirements

* `Makefile` builds with `-Wall -Wextra -pedantic -fsanitize=...`
* README with usage
* At least one unit test per core function (`assert`)
* No memory leaks (`valgrind` or ASan clean)

---

# APPENDIX A SOLUTIONS & ANSWERS

*(peek only after you’ve tried)*

<details><summary>Part 0 — Reflection answer</summary>
`make` looked for *.c files; none existed, so targets were up-to-date. After adding `scratch.c`, its timestamp was newer than any executable, so it got built.
</details>

<details><summary>Part 1 — Try-It snippet</summary>

```c
puts("Hello, Sama!");
printf("2 + 2 = %d\n", 2 + 2);
```

</details>

<details><summary>Part 2 — Circle area</summary>

```c
#include <stdio.h>
#define PI 3.141592653589793

int main(void)
{
    double r;
    if (scanf("%lf",&r)!=1) return 1;
    printf("area = %.4f\n", PI*r*r);
}
```

</details>

<details><summary>Part 3 — Four-label solution</summary>

```c
if (n < 0) {
    if (n % 2 == 0) puts("negative even");
    else            puts("negative odd");
} else {
    if (n % 2 == 0) puts("positive even");
    else            puts("positive odd");
}
```

</details>

<details><summary>Part 4 — Command-line FizzBuzz core</summary>

```c
int limit = (argc > 1) ? atoi(argv[1]) : 100;
for (int i = 1; i <= limit; ++i) { /* same logic */ }
```

</details>

<details><summary>Part 5 — average</summary>

```c
double average(int a,int b){ return (a+b)/2.0; }
```

</details>

<details><summary>Part 6 — max of array</summary>

```c
int max = arr[0];
for(int i=1;i<n;++i) if(arr[i]>max) max=arr[i];
```

</details>

<details><summary>Part 7 — my_strlen</summary>

```c
size_t my_strlen(const char *s){
    const char *p=s;
    while(*p) ++p;
    return (size_t)(p-s);
}
```

</details>

<details><summary>Part 8 — color_name</summary>

```c
const char *color_name(enum Color c){
    switch(c){
    case RED: return "RED";
    case GREEN: return "GREEN";
    case BLUE: return "BLUE";
    default: return "UNKNOWN";
    }
}
```

</details>

<details><summary>Part 9 — read_file</summary>

```c
char *read_file(const char *path){
    FILE *fp=fopen(path,"rb");
    if(!fp) return NULL;
    size_t cap=1024,len=0;
    char *buf=malloc(cap);
    int ch;
    while((ch=fgetc(fp))!=EOF){
        if(len==cap){ cap*=2; buf=realloc(buf,cap); }
        buf[len++]=ch;
    }
    buf[len]='\0';
    fclose(fp);
    return buf;
}
```

</details>

<details><summary>Part 10 — CRC stub</summary>

```c
uint32_t crc=~0U;
/* update crc with each fwrite chunk */
printf("CRC32=%08X\n", crc^~0U);
```

</details>

<details><summary>Knowledge Check answers (selected)</summary>

* **Part 2 Q3**: Integer division truncates toward zero, so `5/2 == 2`.
* **Part 7 Q2**: `p++` when `p` is `int*` advances 4 bytes.
* **Part 12 Q1**: `-g` flag.

</details>

*(Remaining solutions follow same level of detail.)*

---

## WHAT NEXT?

* Work through **Advent of Code** days 1-15 in C.
* Read *21st Century C* for modern practices.
* Add `cmocka` or `Unity` for real unit testing.

Stay paranoid about **undefined behaviour**—keep sanitizers on, compile with all warnings, and treat warnings as errors. Happy hacking!

## PART 0 BUILDING YOUR PLAYGROUND (45 min)

### 1 · Concept Box

| Term            | Plain English                | 中文(繁體) + pinyin  |
| --------------- | ---------------------------- | ---------------- |
| **source file** | the `.c` text you write      | 源碼檔 yuán-mǎ dǎng |
| **object file** | machine-code fragment (`.o`) | 目標檔 mù-biāo dǎng |
| **compiler**    | turns source → object        | 編譯器 biān-yì-qì   |
| **linker**      | glues objects → program      | 連結器 lián-jié-qì  |
| **Makefile**    | rebuild script               | Makefile         |

### 2 · Step-by-Step

1. **Open VS Code → File → Open Folder…** create/select

   ```
   c_course
   ```

2. **Create** `hello.c` (Explorer → New File). Type one placeholder line for now and save.

3. **Create** `Makefile` (exact name, no extension).

   ```make
   # ---------- BEGINNER MAKEFILE -------------------------
   CC      = gcc                               # compiler
   CFLAGS  = -Wall -Wextra -std=c17 -g         # warnings + debug

   SOURCES  = $(wildcard *.c)                  # every .c file
   BINARIES = $(patsubst %.c,%, $(SOURCES))    # hello.c → hello

   all: $(BINARIES)                            # default target

   %: %.c                                      # compiles any .c
   	$(CC) $(CFLAGS) $< -o $@

   clean:                                      # remove binaries
   	rm -f $(BINARIES)
   # ------------------------------------------------------
   ```

4. **Open a terminal inside the folder** (VS Code → Terminal → New Terminal). Prompt should show `c_course $`.

5. **Run `make`** — expect

   ```
   make: Nothing to be done for 'all'.
   ```

6. **Put real code into `hello.c`:**

   ```c
   #include <stdio.h>

   int main(void)
   {
       puts("Hello, world!");
       return 0;
   }
   ```

7. **Re-run `make` and execute.**

   ```bash
   $ make
   gcc … hello.c -o hello
   $ ./hello
   Hello, world!
   ```

8. **Clean up binaries.**

   ```bash
   $ make clean
   ```

### 3 · Common Error Decoder

| Message                            | Meaning                 | How to Fix                            |
| ---------------------------------- | ----------------------- | ------------------------------------- |
| `CC: command not found`            | GCC/Clang not installed | Install compiler, restart terminal    |
| `*** No rule to make target`       | Typo in filename        | Check spelling in Makefile & Explorer |
| `implicit declaration of function` | Missing `#include`      | Add correct header                    |

### 4 · Try-It Task ☑

Add an empty file `scratch.c`, run `make`, verify a new executable `scratch` appears.

### 5 · Stretch Goal 💪

Add `-fsanitize=address,undefined` to `CFLAGS`, rebuild, rerun.

### 6 · Check-Yourself ✅

1. What exact file does the compiler turn `hello.c` into?
2. What does `-Wall` enable?
3. Which Makefile target deletes binaries?

### 7 · Reflection 💭

Explain (to a rubber duck) why `make` did nothing before real code existed.

### 8 · Solution Walk-through 🛠

<details><summary>Step-by-step explanation</summary>

The pattern rule

```
%: %.c
	$(CC) $(CFLAGS) $< -o $@
```

reads “*for any file matching `name.c`, compile it to program `name`.*” `$<` is the `.c`; `$@` the output binary.
When no `.c` existed, `make` found nothing to build. After you saved `hello.c`, its timestamp was newer than `hello`, so the rule fired.

</details>

---

## PART 1 TALKING TO THE COMPUTER (1 h)

*(Same eight-section layout; already shown in previous reply—kept here for completeness.)*

### 1 · Concept Box

| Term          | Meaning                | 中文(繁體)+pinyin   |
| ------------- | ---------------------- | --------------- |
| **function**  | reusable code block    | 函式 hán-shì      |
| **`main`**    | where execution starts | 主函式 zhǔ hán-shì |
| **statement** | code line ending `;`   | 陳述句 chén-shù-jù |

### 2 · Step-by-Step

1. Replace content of `hello.c` with:

   ```c
   #include <stdio.h>      // puts & printf

   int main(void)          // program entry
   {
       puts("Hello, learner!");
       return 0;           // success code
   }
   ```

2. Run:

   ```bash
   $ make && ./hello
   ```

3. **Break it on purpose:** delete the `#include` line, rebuild, read the warning, restore line.

### 3 · Common Error Decoder

| Message                                   | Translation                | Fix                      |
| ----------------------------------------- | -------------------------- | ------------------------ |
| `implicit declaration of function 'puts'` | Compiler can’t find `puts` | Add `#include <stdio.h>` |
| `expected ';' before 'return'`            | missed semicolon above     | insert `;`               |

### 4 · Try-It Task ☑

Add:

```c
puts("2 + 2 = 4");
printf("Good-bye!\n");
```

### 5 · Stretch Goal 💪

Change `return 0;` → `return 42;`, recompile, run `echo $?`.

### 6 · Check-Yourself ✅

1. Where does execution always begin?
2. Why must every statement end with `;`?
3. Which header declares `printf`?

### 7 · Reflection 💭

Explain why returning a value matters to the OS.

### 8 · Solution Walk-through 🛠

<details><summary>Annotated solution</summary>

```c
#include <stdio.h>

int main(void)
{
    puts("Hello, learner!"); // prints + newline
    puts("2 + 2 = 4");       // second line
    printf("Good-bye!\n");   // needs explicit \n
    return 0;                // success
}
```

</details>

---

## PART 2 REMEMBERING THINGS (75 min)

### 1 · Concept Box

| Term         | Meaning            | 中文(繁體)+pinyin   |
| ------------ | ------------------ | --------------- |
| **variable** | named storage      | 變數 biàn-shù     |
| **type**     | size + allowed ops | 型別 xíng-bié     |
| **literal**  | hard-coded value   | 字面值 zì-miàn-zhí |

| Type     | Bytes\* | Example literal |
| -------- | ------- | --------------- |
| `int`    | 4       | `42`            |
| `double` | 8       | `3.14`          |
| `char`   | 1       | `'A'`           |

\*on a 64-bit PC.

### 2 · Step-by-Step

1. **Create** `variables.c` with:

   ```c
   #include <stdio.h>

   int main(void)
   {
       int    age    = 25;
       double price  = 2.99;
       char   initial = 'S';

       printf("age=%d  price=%.2f  initial=%c\n",
              age, price, initial);
       return 0;
   }
   ```

2. **Build & run.**

   ```bash
   $ make variables && ./variables
   ```

3. **Modify** `age` to `age + 1`, re-run.

### 3 · Common Error Decoder

| Message                                             | Meaning              | Fix                    |
| --------------------------------------------------- | -------------------- | ---------------------- |
| `format ‘%f’ expects argument of type ‘double’`     | Placeholder mismatch | Use matching specifier |
| `warning: overflow in implicit constant conversion` | Literal too big      | Pick bigger type       |

### 4 · Try-It Task ☑

Write `circle.c` that:

1. `printf("radius? ");`
2. `scanf("%lf",&r);`
3. prints area = π r² (`#define PI 3.141592653589793`).

### 5 · Stretch Goal 💪

Add height input, compute cylinder volume (π r²h).

### 6 · Check-Yourself ✅

1. Which format specifier prints an `int`?
2. What operator gives remainder?
3. What value does `5 / 2` evaluate to in C?

### 7 · Reflection 💭

Why can’t the compiler guess variable types automatically (unlike Python)?

### 8 · Solution Walk-through 🛠

<details><summary>circle.c explained</summary>

```c
#include <stdio.h>
#define PI 3.141592653589793

int main(void)
{
    double r;
    printf("radius? ");
    if (scanf("%lf", &r) != 1) return 1;   // input guard

    double area = PI * r * r;
    printf("area = %.3f\n", area);
    return 0;
}
```

*Why `%.3f`?* Limits output to 3 decimals.

</details>

---

## PART 3 MAKING DECISIONS (75 min)

### 1 · Concept Box

| Term           | Meaning                 | 中文(繁體)+pinyin  |
| -------------- | ----------------------- | -------------- |
| **boolean**    | 0 = false, non-0 = true | 布林值 bù-lín-zhí |
| **branch**     | choose a path           | 分岔 fēn-chà     |
| **comparison** | test relation           | 比較 bǐ-jiào     |

### 2 · Step-by-Step

1. **Create** `parity.c`:

   ```c
   #include <stdio.h>

   int main(void)
   {
       int n;
       printf("Enter number: ");
       if (scanf("%d", &n) != 1) { puts("Bad input"); return 1; }

       if (n % 2 == 0)
           puts("even");
       else
           puts("odd");

       return 0;
   }
   ```

2. **Build & run**; test with 7, 12.

3. **Extend**: nest another `if` to label negative vs non-negative.

### 3 · Common Error Decoder

| Message                                 | Meaning             | Fix           |
| --------------------------------------- | ------------------- | ------------- |
| `suggest parentheses around assignment` | Used `=` not `==`   | Use `==`      |
| `unused variable`                       | declared but unused | delete or use |

### 4 · Try-It Task ☑

Program must output *one of four* strings:

* negative even
* negative odd
* positive even
* positive odd

### 5 · Stretch Goal 💪

Implement same logic using:

```c
switch ((n < 0) * 2 + (n % 2 == 0)) { ... }
```

### 6 · Check-Yourself ✅

1. Why does C consider any non-zero `int` as true?
2. Write a condition for “multiple of 3 **and** 5”.
3. What are `!0` and `!7`?

### 7 · Reflection 💭

Explain why assignments inside `if` are dangerous.

### 8 · Solution Walk-through 🛠

<details><summary>Four-label solution</summary>

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

---

## PART 4 DOING THINGS AGAIN (90 min)

### 1 · Concept Box

| Term          | Meaning       | 中文(繁體)+pinyin |
| ------------- | ------------- | ------------- |
| **loop**      | repeat code   | 回圈 huí-quān   |
| **iteration** | one pass      | 迭代 dì-dài     |
| **counter**   | loop variable | 計數器 jì-shù-qì |

### 2 · Step-by-Step

1. **Create** `fizzbuzz.c` skeleton:

   ```c
   #include <stdio.h>
   #include <stdlib.h>   // for atoi

   int main(int argc, char *argv[])
   {
       int limit = (argc > 1) ? atoi(argv[1]) : 100;

       for (int i = 1; i <= limit; ++i) {
           if (i % 15 == 0)      puts("FizzBuzz");
           else if (i % 3 == 0)  puts("Fizz");
           else if (i % 5 == 0)  puts("Buzz");
           else                  printf("%d\n", i);
       }
       return 0;
   }
   ```

2. `make fizzbuzz && ./fizzbuzz 50`

### 3 · Common Error Decoder

| Message       | Meaning               | Fix                |
| ------------- | --------------------- | ------------------ |
| infinite loop | condition never false | fix counter update |
| off-by-one    | wrong `<` vs `<=`     | check range        |

### 4 · Try-It Task ☑

Read limit from user *if* no command-line arg; else use `argv[1]`.

### 5 · Stretch Goal 💪

Print FizzBuzz horizontally, separated by spaces.

### 6 · Check-Yourself ✅

1. Three fields of a `for` loop?
2. Difference between `while` and `do … while`?
3. What does `continue` do?

### 7 · Reflection 💭

Why are off-by-1 errors common?

### 8 · Solution Walk-through 🛠

<details><summary>Input-fallback solution</summary>

```c
if (argc == 1) {
    printf("Limit? ");
    scanf("%d", &limit);
}
```

</details>

---

## PART 5 ORGANISING WORK (FUNCTIONS) (60 min)

### 1 · Concept Box

| Term             | Meaning         | 中文(繁體)+pinyin     |
| ---------------- | --------------- | ----------------- |
| **parameter**    | input value     | 參數 cān-shù        |
| **return value** | function output | 回傳值 huí-chuán-zhí |
| **scope**        | visibility      | 作用域 zuò-yòng-yù   |

### 2 · Step-by-Step

1. **Create** `functions.c`:

   ```c
   #include <stdio.h>

   static int max(int a, int b)      // internal helper
   {
       return (a > b) ? a : b;
   }

   int main(void)
   {
       int x, y;
       printf("Two ints: ");
       if (scanf("%d %d", &x, &y) != 2) return 1;
       printf("max = %d\n", max(x,y));
       return 0;
   }
   ```

2. Build and test.

### 3 · Common Error Decoder

| Message             | Meaning            | Fix                            |
| ------------------- | ------------------ | ------------------------------ |
| `conflicting types` | prototype mismatch | keep same signature            |
| `unused parameter`  | parameter not used | remove or prefix with `(void)` |

### 4 · Try-It Task ☑

Add `double average(int a,int b)` and print average.

### 5 · Stretch Goal 💪

Write `int clamp(int val,int min,int max)`.

### 6 · Check-Yourself ✅

1. Does C pass parameters by value or reference?
2. What keyword hides a function inside its file?
3. Can a function return two values?

### 7 · Reflection 💭

Why does copying parameters protect the caller?

### 8 · Solution Walk-through 🛠

<details><summary>average explained</summary>

```c
static double average(int a,int b)
{
    return (a + b) / 2.0;   // 2.0 forces floating division
}
```

</details>

---

## PART 6 LISTS OF DATA (ARRAYS) (2 h)

### 1 · Concept Box

| Term        | Meaning           | 中文(繁體)+pinyin |
| ----------- | ----------------- | ------------- |
| **array**   | fixed-length list | 陣列 zhèn-liè   |
| **index**   | position          | 索引 suǒ-yǐn    |
| **element** | item              | 元素 yuán-sù    |

### 2 · Step-by-Step

1. **Create** `array_max.c`:

   ```c
   #include <stdio.h>
   #define MAX 100

   int main(void)
   {
       int n;
       printf("How many numbers (<=%d)? ", MAX);
       if (scanf("%d",&n)!=1 || n<1 || n>MAX) return 1;

       int arr[MAX];
       for (int i=0;i<n;++i) scanf("%d",&arr[i]);

       int max = arr[0];
       for (int i=1;i<n;++i) if (arr[i]>max) max=arr[i];
       printf("max = %d\n", max);
       return 0;
   }
   ```

2. Compile/run.

### 3 · Common Error Decoder

| Message                               | Meaning             | Fix            |
| ------------------------------------- | ------------------- | -------------- |
| segmentation fault                    | index out of range  | check bounds   |
| `initializer element is not constant` | VLA at global scope | move to `main` |

### 4 · Try-It Task ☑

Add bubble-sort, print sorted list.

### 5 · Stretch Goal 💪

Implement binary search for a target value in the sorted array.

### 6 · Check-Yourself ✅

1. First valid index?
2. What happens if you access index −1?
3. Why can’t ordinary arrays change length at runtime?

### 7 · Reflection 💭

Draw a memory diagram of `int arr[4]`.

### 8 · Solution Walk-through 🛠

<details><summary>Bubble-sort core</summary>

```c
for (int pass=0; pass<n-1; ++pass)
    for (int i=0;i<n-1-pass;++i)
        if (arr[i] > arr[i+1]) {
            int t=arr[i]; arr[i]=arr[i+1]; arr[i+1]=t;
        }
```

</details>

---

## PART 7 POWER TOOL – POINTERS (2 h)

### 1 · Concept Box

| Term            | Meaning           | 中文(繁體)+pinyin   |
| --------------- | ----------------- | --------------- |
| **pointer**     | memory address    | 指標 zhǐ-biāo     |
| **dereference** | `*p` gives value  | 反參考 fǎn-cān-kǎo |
| **`const`**     | read-only promise | 常數 cháng-shù    |

### 2 · Step-by-Step

1. **Create** `swap_len.c`:

   ```c
   #include <stdio.h>
   #include <string.h>   // for strlen check

   void swap(int *a,int *b)
   {
       int tmp=*a; *a=*b; *b=tmp;
   }

   size_t my_strlen(const char *s)
   {
       const char *p=s;
       while(*p) ++p;
       return (size_t)(p-s);
   }

   int main(void)
   {
       int x=5,y=7;
       swap(&x,&y);
       printf("x=%d y=%d\n",x,y);

       char str[64];
       printf("String? ");
       scanf("%63s",str);
       printf("len=%zu (lib=%zu)\n",
              my_strlen(str), strlen(str));
       return 0;
   }
   ```

2. Compile & test.

### 3 · Common Error Decoder

| Message                    | Meaning            | Fix                  |
| -------------------------- | ------------------ | -------------------- |
| segfault on `*p`           | pointer NULL       | validate pointer     |
| incompatible pointer types | wrong pointer type | cast or correct type |

### 4 · Try-It Task ☑

Write `char *my_strdup(const char *s)` using `malloc`.

### 5 · Stretch Goal 💪

Implement reverse-in-place for a null-terminated string.

### 6 · Check-Yourself ✅

1. Type of `&array[0]`?
2. How far does `p++` move when `p` is `int*`?
3. Why is `const char *` safer for string input?

### 7 · Reflection 💭

Draw stack vs heap for `int *p = malloc(sizeof *p);`.

### 8 · Solution Walk-through 🛠

<details><summary>my_strdup explained</summary>

```c
char *my_strdup(const char *s)
{
    size_t len = my_strlen(s)+1;  // include NUL
    char *copy = malloc(len);
    if (!copy) return NULL;
    for (size_t i=0;i<len;++i) copy[i]=s[i];
    return copy;
}
```

</details>

---

## PART 8 CUSTOM SHAPES (STRUCTS & ENUMS) (2 h)

### 1 · Concept Box

| Term        | Meaning        | 中文(繁體)+pinyin          |
| ----------- | -------------- | ---------------------- |
| **struct**  | grouped fields | 結構 jié-gòu             |
| **enum**    | named ints     | 列舉 liè-jǔ              |
| **typedef** | alias          | 型別別名 xíng-bié bié-míng |

### 2 · Step-by-Step

1. **Create** `player.c`:

   ```c
   #include <stdio.h>

   typedef struct {
       char name[32];
       int  score;
   } Player;

   enum Color { RED, GREEN, BLUE };

   const char *color_name(enum Color c)
   {
       switch(c){
       case RED: return "RED";
       case GREEN: return "GREEN";
       case BLUE: return "BLUE";
       default: return "UNKNOWN";
       }
   }

   int main(void)
   {
       Player p = {"Alex", 42};
       printf("%s has %d pts, color=%s\n",
              p.name, p.score, color_name(GREEN));
       return 0;
   }
   ```

2. Compile/run.

### 3 · Common Error Decoder

| Message                                 | Meaning         | Fix           |
| --------------------------------------- | --------------- | ------------- |
| `initializer-string for array too long` | name > 31 chars | enlarge field |
| padding surprise                        | assumed packed  | use `sizeof`  |

### 4 · Try-It Task ☑

Build a singly-linked list of `Player` nodes: add, remove, print.

### 5 · Stretch Goal 💪

Sort list by score descending.

### 6 · Check-Yourself ✅

1. Why might `sizeof(Player)` exceed 36 bytes?
2. Field-access via pointer uses which operator?
3. What keyword hides internal helper functions?

### 7 · Reflection 💭

Explain alignment padding in one sentence.

### 8 · Solution Walk-through 🛠

<details><summary>Node struct</summary>

```c
typedef struct Node {
    Player p;
    struct Node *next;
} Node;
```

</details>

---

## PART 9 MEMORY YOU CONTROL (2 h)

### 1 · Concept Box

| Term         | Meaning           | 中文(繁體)+pinyin         |
| ------------ | ----------------- | --------------------- |
| **heap**     | long-lived memory | 堆積 duī-jī             |
| **`malloc`** | request bytes     | 動態分配 dòng-tài fēn-pèi |
| **`free`**   | release bytes     | 釋放 shì-fàng           |

### 2 · Step-by-Step

1. **Create** `read_file.c`:

   ```c
   #include <stdio.h>
   #include <stdlib.h>

   char *read_file(const char *path)
   {
       FILE *fp = fopen(path,"rb");
       if(!fp) return NULL;

       size_t cap=1024,len=0;
       char *buf=malloc(cap);
       int ch;
       while((ch=fgetc(fp))!=EOF){
           if(len==cap){
               cap*=2;
               buf=realloc(buf,cap);
           }
           buf[len++]=ch;
       }
       buf[len]='\0';
       fclose(fp);
       return buf;
   }

   int main(int argc,char *argv[])
   {
       if (argc<2){ puts("usage: read_file file"); return 1; }
       char *txt = read_file(argv[1]);
       if(!txt){ perror("open"); return 1; }
       puts(txt);
       free(txt);
       return 0;
   }
   ```

2. Build/run: `./read_file Makefile | head`.

### 3 · Common Error Decoder

| Message     | Meaning             | Fix                         |
| ----------- | ------------------- | --------------------------- |
| double free | pointer freed twice | set pointer NULL after free |
| leak        | never freed         | add `free`                  |

### 4 · Try-It Task ☑

Turn buffer into dynamic array module with `dynarray_push`, `dynarray_free`.

### 5 · Stretch Goal 💪

Add ASan (`-fsanitize=address`) and intentionally cause double-free.

### 6 · Check-Yourself ✅

1. What happens if `malloc` returns NULL?
2. Why add `\0` at buffer end?
3. How can realloc shrink memory?

### 7 · Reflection 💭

Describe “dangling pointer”.

### 8 · Solution Walk-through 🛠

<details><summary>Guard malloc fail</summary>

```c
char *buf = malloc(cap);
if (!buf) { fclose(fp); return NULL; }
```

</details>

---

## PART 10 TALKING TO FILES (90 min)

### 1 · Concept Box

| Term            | Meaning                | 中文(繁體)+pinyin           |
| --------------- | ---------------------- | ----------------------- |
| **stream**      | `FILE*` handle         | 資料流 zī-liào-liú         |
| **buffer**      | temporary storage      | 緩衝區 huǎn-chōng-qū       |
| **binary mode** | no newline translation | 二進位模式 èr-jìn-wèi mó-shì |

### 2 · Step-by-Step

1. **Create** `copy.c`:

   ```c
   #include <stdio.h>

   int main(int argc,char *argv[])
   {
       if(argc!=3){ puts("usage: copy src dst"); return 1; }

       FILE *src=fopen(argv[1],"rb");
       FILE *dst=fopen(argv[2],"wb");
       if(!src||!dst){ perror("file"); return 1; }

       char buf[4096];
       size_t n, total=0;
       while((n=fread(buf,1,sizeof buf,src))>0){
           if(fwrite(buf,1,n,dst)!=n){ perror("write"); return 1; }
           total+=n;
       }
       printf("copied %zu bytes\n",total);
       fclose(src); fclose(dst);
       return 0;
   }
   ```

2. Build & test.

### 3 · Common Error Decoder

| Message              | Meaning             | Fix                 |
| -------------------- | ------------------- | ------------------- |
| `Segmentation fault` | `src` or `dst` NULL | check fopen result  |
| partial write        | disk full           | handle return value |

### 4 · Try-It Task ☑

Compute CRC-32 during copy; print checksum.

### 5 · Stretch Goal 💪

Add progress bar for files >1 MB.

### 6 · Check-Yourself ✅

1. Difference between `"rb"` and `"r"`?
2. Why is I/O buffered?
3. Which function flushes a stream?

### 7 · Reflection 💭

What happens if you forget `fclose`?

### 8 · Solution Walk-through 🛠

<details><summary>CRC stub</summary>

```c
uint32_t crc=~0U;
/* update with table[ (crc ^ byte) & 0xFF ] each byte */
printf("CRC32=%08X\n", crc^~0U);
```

</details>

---

## PART 11 SPLITTING A PROGRAM (90 min)

### 1 · Concept Box

| Term                 | Meaning                 | 中文(繁體)+pinyin               |
| -------------------- | ----------------------- | --------------------------- |
| **header**           | declarations (`.h`)     | 標頭檔 biāo-tóu dǎng           |
| **compilation unit** | one `.c` file           | 編譯單元 biān-yì dān-yuán       |
| **include guard**    | prevents double include | 防重複定義 fáng-chóng-fù dìng-yì |

### 2 · Step-by-Step

1. From Part 8 list, split into:

   * `list.h`
   * `list.c`
   * `main.c`

2. **list.h**

   ```c
   #ifndef LIST_H
   #define LIST_H
   #include "player.h"     // define Player struct

   typedef struct Node Node;
   Node *list_push(Node *head, Player p);
   void  list_print(const Node *head);
   void  list_free(Node *head);

   #endif
   ```

3. **list.c** implements the functions (mark helpers `static`).

4. **Modify Makefile:** nothing! Pattern rule already compiles each `.c`.

5. `make -j` (parallel build).

### 3 · Common Error Decoder

| Message              | Meaning                 | Fix                       |
| -------------------- | ----------------------- | ------------------------- |
| multiple definition  | same symbol in two `.c` | `static` or single source |
| implicit declaration | forgot header include   | add `#include "list.h"`   |

### 4 · Try-It Task ☑

Add `test_list.c` with asserts; build and run.

### 5 · Stretch Goal 💪

Generate static library `liblist.a` and link main against it.

### 6 · Check-Yourself ✅

1. Why mark helpers `static`?
2. What does compiler flag `-c` do?
3. How does `make` detect changed files?

### 7 · Reflection 💭

Explain header guards in 30 s.

### 8 · Solution Walk-through 🛠

<details><summary>list_push core</summary>

```c
Node *list_push(Node *head, Player p)
{
    Node *n = malloc(sizeof *n);
    if(!n) return head;
    n->p = p; n->next = head;
    return n;
}
```

</details>

---

## PART 12 FINDING BUGS (1 h)

### 1 · Concept Box

| Term          | Meaning           | 中文(繁體)+pinyin        |
| ------------- | ----------------- | -------------------- |
| **debugger**  | runtime inspector | 除錯器 chú-cuò-qì       |
| **sanitizer** | UB detector       | 檢測器 jiǎn-cè-qì       |
| **backtrace** | call stack        | 呼叫堆疊 hū-jiào duī-dié |

### 2 · Step-by-Step

1. Recompile any program with:

   ```
   CFLAGS = -Wall -Wextra -std=c17 -g -fsanitize=address,undefined
   ```

2. **Introduce a bug:** in `swap_len.c`, change `while(*p)` to `while(*(p+1000))`, rebuild & run.

   Sanitizer catches heap-buffer-overflow, shows line number.

3. **Use gdb:**

   ```bash
   gdb ./swap_len
   (gdb) b my_strlen
   (gdb) r
   (gdb) n
   (gdb) p *p
   (gdb) bt
   ```

### 3 · Common Error Decoder

| Message                    | Meaning        | Fix           |
| -------------------------- | -------------- | ------------- |
| ASan: heap-buffer-overflow | wrote past end | correct index |
| UBSan: division by zero    | denominator 0  | guard input   |

### 4 · Try-It Task ☑

Step through `my_strlen` in gdb, watch pointer advance.

### 5 · Stretch Goal 💪

Enable `-fsanitize=leak` and confirm zero leaks in Part 9.

### 6 · Check-Yourself ✅

1. Which flag adds debug symbols?
2. Which sanitizer catches use-after-free?
3. gdb command to set breakpoint at `swap`?

### 7 · Reflection 💭

Why are logic bugs harder than crashes?

### 8 · Solution Walk-through 🛠

<details><summary>Breakpoint syntax</summary>

```
(gdb) break swap
```

</details>

---

## PART 13 CAPSTONE PROJECT (5–6 h)

Pick **one**. A 12-page guided PDF (generate separately on request) walks through design, milestones, and tests.

### Option A Text Metrics

* Re-implement Unix `wc` plus: top-10 words, avg line length.
* Uses dynamic array & hash table (starter code given).

### Option B Tic-Tac-Toe AI

* 3×3 board, CLI.
* Minimax w/ pruning, colored terminal output.

### Option C HTTP GET Client

* Raw BSD sockets.
* Follows one redirect, saves body.

#### Capstone Requirements

| Item    | Specification                                            |
| ------- | -------------------------------------------------------- |
| Build   | `make all` with `-Wall -Wextra -pedantic -fsanitize=...` |
| Tests   | ≥1 unit test (`assert`) per core function                |
| Docs    | README: build & usage                                    |
| Quality | No leaks (`valgrind` or ASan clean)                      |

---

# Appendix A Answers & Hints

| Part                  | Q # | Answer / Hint                                                                                        |
| --------------------- | --- | ---------------------------------------------------------------------------------------------------- |
| **0 – Playground**    | 1   | The final linked program (e.g. `hello`). If you had used `-c`, you’d get an object file (`hello.o`). |
|                       | 2   | `-Wall` enables “all common” warnings. (Think **W**-all).                                            |
|                       | 3   | Target **`clean`** → `make clean` removes built binaries.                                            |
| **1 – Talking**       | 1   | Execution always starts in `main`.                                                                   |
|                       | 2   | The semicolon marks the end of a statement so the compiler knows where one instruction stops.        |
|                       | 3   | `<stdio.h>` declares `printf` (and `puts`, `scanf`, …).                                              |
| **2 – Variables**     | 1   | `%d` prints an `int` (decimal).                                                                      |
|                       | 2   | `%` (modulus).                                                                                       |
|                       | 3   | `5 / 2` ⇒ `2` (integer division truncates toward 0).                                                 |
| **3 – Decisions**     | 1   | Historically, any non-zero value is treated as **true** (簡單, no dedicated Boolean type in C89).      |
|                       | 2   | `(n % 3 == 0) && (n % 5 == 0)`.                                                                      |
|                       | 3   | `!0` → `1`, `!7` → `0` (logical NOT).                                                                |
| **4 – Loops**         | 1   | **initialiser ; test ; increment**.                                                                  |
|                       | 2   | `while` tests first, `do … while` tests *after* the first run.                                       |
|                       | 3   | `continue` skips the rest of the loop body and jumps to the next iteration.                          |
| **5 – Functions**     | 1   | Always **by value**. (You pass a *copy*, unless you pass a pointer.)                                 |
|                       | 2   | `static` (file-scope internal linkage).                                                              |
|                       | 3   | Not directly—return one value; for more, use pointers or structs.                                    |
| **6 – Arrays**        | 1   | Index `0`.                                                                                           |
|                       | 2   | Undefined behaviour (likely seg-fault) – array bounds aren’t checked.                                |
|                       | 3   | Length must be known at compile-time (plain C arrays have fixed size; use `malloc` for dynamic).     |
| **7 – Pointers**      | 1   | `int *` (pointer to int).                                                                            |
|                       | 2   | `p++` advances by `sizeof(int)` bytes (normally 4).                                                  |
|                       | 3   | `const char *` promises the function won’t modify the string, preventing accidental writes.          |
| **8 – Structs/Enums** | 1   | Padding/alignment; compiler may insert bytes so every `int` starts on a 4-byte boundary.             |
|                       | 2   | `->` (arrow) dereferences the pointer then accesses the field.                                       |
|                       | 3   | Same as Part 5: `static` keeps helpers private to the `.c` file.                                     |
| **9 – Dynamic Mem**   | 1   | Program gets `NULL`; check and handle (usually exit or retry).                                       |
|                       | 2   | Adds the terminating NUL so string functions work.                                                   |
|                       | 3   | Yes – if the new size is smaller, excess bytes are released (contents beyond new size lost).         |
| **10 – File I/O**     | 1   | `"rb"` is binary (no newline translation, no charset meddling), `"r"` is text.                       |
|                       | 2   | Buffering reduces sys-calls; faster sequential reads/writes.                                         |
|                       | 3   | `fflush(FILE*)` (or `fclose`, which flushes then closes).                                            |
| **11 – Modular C**    | 1   | `static` avoids multiple-definition errors and hides symbols.                                        |
|                       | 2   | `-c` stops after compiling, producing `.o`; no linking.                                              |
|                       | 3   | By **timestamps**: if a dependency changed after the target was built, the rule reruns.              |
| **12 – Debugging**    | 1   | `-g` (plus you usually keep `-O0` while debugging).                                                  |
|                       | 2   | AddressSanitizer (`-fsanitize=address`) reports use-after-free.                                      |
|                       | 3   | In gdb: `break swap` (or short `b swap`).                                                            |

That’s every checkpoint. If a line doesn’t ring crystal-clear, revisit the relevant code and rebuild it with warnings + sanitizers—you’ll master the concept much faster than by rereading notes.


---

## WHERE TO GO NEXT

1. **Advent of Code** days 1-15 in C.
2. Read *21st-Century C* (Ben Klemens).
3. Add `cmocka` or `Unity` for richer unit tests.

Remember: C’s syntax is easy—*undefined behaviour* is the enemy. Keep sanitizers on and treat **all warnings as errors**. Good luck, and enjoy the power (and sharp edges) of C!

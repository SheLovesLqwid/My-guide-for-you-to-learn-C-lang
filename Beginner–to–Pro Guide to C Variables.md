# 📚 Beginner–to–Pro Guide to **C Variables** (Deep Dive)

Variables are the foundation of C. This guide covers **everything you need to know**—from the basics to advanced, standard-accurate details (C99/C11/C17). Lots of examples, memory diagrams, and gotchas.

---

## 1) What is a variable?

A **variable** is a named object that occupies **storage** (memory) and holds a **value** of a specific **type**.

```
name ──→ [ memory at some address ] ──→ value (interpreted according to its type)
```

A variable’s behavior is determined by:

* **Type** (what kind of values it holds and how many bytes)
* **Storage duration** (when it exists in time)
* **Scope** (where its name is visible)
* **Linkage** (whether the name refers to the same entity across translation units)
* **Qualifiers** (const/volatile/restrict/\_Atomic that constrain or describe access)

---

## 2) Core object types (sizes are typical on 32/64-bit systems)

> Exact sizes are implementation-defined; use `sizeof(T)` to be sure.

* **`int`** – whole numbers; usually 4 bytes.
* **`float`** – single-precision floating point; 4 bytes (\~6–7 decimal digits).
* **`double`** – double-precision; 8 bytes (\~15–16 digits).
* **`char`** – 1 byte; stores an integer representing a character (e.g., ASCII).

  * `char` may be **signed** or **unsigned** (implementation-defined).
  * Use `signed char` / `unsigned char` if you need a specific signedness.
* **`_Bool` / `bool`** – boolean (include `<stdbool.h>` to use `bool` alias). Stored as 0 or 1; typically 1 byte.
* **`char[]`** – arrays of characters (strings end with `'\0'`).

### Fixed-width & utility types (`<stdint.h>`, `<stddef.h>`)

* Exact-width: `int8_t`, `uint16_t`, `int32_t`, `uint64_t`, …
* At-least-width: `int_least16_t`, …
* Fast types: `int_fast32_t`, …
* Pointer- and size-related: `size_t`, `ptrdiff_t`, `uintptr_t`, `intptr_t`.

---

## 3) Declaring vs Defining; Initialization

* **Declaration**: tells the compiler a name and type exist.
* **Definition**: allocates storage.
* At **file scope**, a **tentative definition** (`int x;`) becomes a real definition with **zero initialization** if no other definition appears.

```c
// a.c
int x;          // tentative definition -> becomes definition with 0
extern int y;   // declaration only (no storage)

// b.c
int y = 42;     // definition + initialization
```

### Initialization rules

* **Static storage duration** (file-scope vars, or `static` anywhere):
  → **zero-initialized** before program start if not explicitly initialized.
* **Automatic storage duration** (typical local variables):
  → **indeterminate** if not initialized (reading them is **undefined behavior**).

```c
static int gs;      // zero-initialized globally
void f(void) {
    int a;         // indeterminate
    static int b;  // zero-initialized
}
```

### Designated initializers (arrays/structs)

```c
int a[5] = {[2] = 10, [4] = 7}; // others zero
struct P { int x, y; };
struct P p = {.y = 3, .x = 1};
```

---

## 4) Storage duration (lifetime of the object)

* **Automatic**: block-scope locals (default). Created on **block entry**, destroyed on **block exit**. VLAs are automatic.
* **Static**: at file scope or declared `static`. Exists **for entire program**.
* **Thread**: C11 `_Thread_local` (optionally with `static`/`extern`). One instance **per thread**.
* **Allocated**: via dynamic memory (`malloc`/`calloc`/`realloc`); lifetime is controlled by **`free`**.

```c
_Thread_local int per_thread_counter; // each thread has its own copy
```

---

## 5) Scope (where the **name** is visible)

* **Block scope**: from declaration to end of enclosing block `{ ... }`.
* **File scope**: from declaration to end of translation unit.
* **Function scope**: labels (for `goto`) are visible throughout their function.
* **Function prototype scope**: parameter names in prototypes.

```c
int g;           // file scope

void f(void) {
    int x = 1;   // block scope
    { int x = 2; /* inner x shadows outer x */ }
}
```

---

## 6) Linkage (is the name the “same” across files?)

* **External linkage**: same name refers to the **same entity** across translation units.
* **Internal linkage**: name refers to the **same entity** within one translation unit only.
* **No linkage**: each occurrence is a distinct entity (e.g., block-scope variables).

How to control it:

```c
// External linkage (default for file-scope objects without 'static'):
int g1 = 0;

// Internal linkage:
static int g2 = 0;

// Use in other file:
extern int g1;    // refers to the 'g1' defined elsewhere
```

> In C (unlike C++), `const` **does not** change linkage. Use `static` for internal linkage.

---

## 7) Storage-class specifiers

* `auto` – default for block-scope variables (rarely written explicitly).
* `register` – historical hint to put variable in a CPU register; you **cannot** take its address (`&`). Modern compilers usually ignore this hint.
* `static` – static storage duration (for locals), or internal linkage (at file scope).
* `extern` – refers to a variable defined elsewhere (no storage allocated here).
* `_Thread_local` – thread storage duration (C11).

```c
static int counter;     // file-scope, internal linkage
extern int shared;      // declaration only
void f(void) {
    static int calls;   // static storage duration, retains value between calls
}
```

---

## 8) Type qualifiers & what they **really** mean

### `const` – read-only **through this name**

* The object might still be modifiable through another non-const alias.
* At file scope, `const` alone does **not** give internal linkage; add `static` if you want it private to the file.

```c
const int a = 10; // read-only via 'a'
int *p = (int*)&a; // casting away const and writing is UB if object is truly const (e.g., placed in read-only memory)
```

**Pointers + const** (most common confusion):

* `const int *p` → “pointer to **const int**” (you can change p, not \*p)
* `int * const p` → “**const pointer** to int” (you can change \*p, not p)
* `const int * const p` → const pointer to const int

```c
int x = 1, y = 2;
const int *p = &x; // *p read-only
p = &y;            // ok: pointer itself not const
//*p = 3;          // error

int * const q = &x; // q is const
*q = 5;             // ok
//q = &y;           // error
```

### `volatile` – every access is an observable action

* Used for **memory-mapped I/O**, **signal handlers**, or data changed by **hardware**/other execution contexts.
* **Not** a concurrency primitive; it does **not** make code thread-safe.

```c
volatile int *MMIO = (int*)0x40000000; // read from device register
int v = *MMIO; // compiler must perform the read
```

### `restrict` – pointer aliasing promise (C99)

* Tells the compiler the object accessed through this pointer is **not aliased** by any other pointer in the same scope, enabling optimizations.
* **Only for pointers**, and you must uphold the promise.

```c
void add(size_t n, int * restrict dst, const int * restrict a, const int * restrict b) {
    for (size_t i = 0; i < n; ++i) dst[i] = a[i] + b[i];
}
```

### `_Atomic` – atomic types (C11)

* Use `<stdatomic.h>`. Provides atomic operations and memory ordering.

```c
#include <stdatomic.h>
_Atomic int flag = 0;
```

---

## 9) Arrays, pointers, and strings

### Arrays

* Name often **decays** to pointer to first element (except in `sizeof`, unary `&`, or as a string literal initializer).
* Not assignable as whole objects (but can be **initialized** or **copied with `memcpy`**).

```c
int a[3] = {1,2,3};
int *p = a;           // decay to &a[0]
size_t n = sizeof a;  // sizeof array (12 if 4-byte int)
```

### Variable Length Arrays (VLAs) – C99 (optional in C11+)

* Size is runtime-known, automatic storage only, lifetime ends at block exit.

```c
void f(size_t n) {
    int vla[n];       // n must be > 0 (implementation-dependent if 0 allowed)
}
```

### Flexible array member (struct “tail array”)

* Last member can be `T name[];` and is **not** counted in `sizeof`.
* Allocate extra bytes to store trailing elements.

```c
struct Packet {
    size_t len;
    unsigned char data[]; // flexible member at end
};

struct Packet *p = malloc(sizeof *p + payload_size);
p->len = payload_size;
// fill p->data[0..len-1]
```

### Strings

* C strings are **`char[]`** ending with **`'\0'`**.
* String **literals** have **static storage duration** and type `array of char` (modifying them is **UB**).

```c
char s[] = "hi";  // {'h','i','\0'}
char *p  = "hi";  // points to read-only storage; *p = 'H' is UB
```

---

## 10) Structs, unions, enums, and bit-fields

### `struct`

* Groups named members; may insert **padding** for alignment.
* Order matters for layout.

```c
struct Point { int x, y; };
struct Point p = {.y=2, .x=1};
```

### `union`

* All members share the **same storage**; size = max of member sizes.
* Reading a member different from the one most recently written is largely **implementation-defined** / **UB** (with limited exception for common initial sequences in structs).

```c
union U { int i; float f; };
union U u = {.i = 0x3f800000}; // representation trick: risky/implementation-defined
```

### `enum`

* Defines **named integer constants**; variables of enum type are integer types (underlying type is implementation-defined).

```c
enum Color { RED = 1, GREEN, BLUE };
enum Color c = GREEN; // 2
```

### Bit-fields

* Pack multiple small integer members into machine words.

```c
struct Flags {
    unsigned ready:1;
    unsigned error:1;
    unsigned code:6;
};
```

**Notes:** address-of a bit-field is illegal; layout/packing is implementation-defined.

---

## 11) Alignment and address

* Each type has an **alignment requirement**. The compiler may add **padding** inside structs.
* C11 adds `_Alignas` and `_Alignof`.

```c
#include <stdalign.h>
printf("alignof(double)=%zu\n", alignof(double));
_Alignas(16) int x; // request 16-byte alignment
```

---

## 12) Constants & immutability

* **`const`** makes an object read-only **through that name**.
* For compile-time integer constants, prefer `#define` macros or `enum` constants.
* For typed constants with storage, use `static const` at file scope to keep them private.

```c
static const double PI = 3.141592653589793;
```

---

## 13) The integer model (promotions & conversions)

* **Integer promotions**: types narrower than `int` (like `char`, `short`) are promoted to `int` (or `unsigned int`) in expressions.
* **Usual arithmetic conversions**: when mixing signed/unsigned or different ranks, both operands are converted to a common type. Be careful with signed/unsigned comparisons to avoid surprises.

```c
unsigned int u = 1;
int          i = -1;
printf("%d\n", i < u); // often prints 0, because 'i' converts to a large unsigned
```

---

## 14) `sizeof` and `_Alignof` are compile-time (except VLAs)

```c
int a[10];
printf("%zu\n", sizeof a);        // bytes in entire array
printf("%zu\n", sizeof a[0]);     // bytes in one element
```

* On VLAs, `sizeof` is **runtime**.

---

## 15) `typedef` vs variables vs macros

* `typedef` creates a **type alias**, not a variable.

```c
typedef unsigned long ulong;
ulong x; // x is variable of type unsigned long
```

* Macros (`#define`) are **textual substitution**; no type checking.

---

## 16) Thread safety & shared state

* **File-scope** or `static` variables are **shared** across threads unless declared `_Thread_local`.
* Use **atomics** (`_Atomic` + `<stdatomic.h>`) or **mutexes** (from `<threads.h>` or POSIX) to synchronize.
* `volatile` ≠ synchronization.

---

## 17) Lifetime pitfalls & UB hall of fame

* **Reading uninitialized automatic vars** → UB.
* **Returning address of a local** → dangling pointer.

```c
int *bad(void) {
    int x = 42;
    return &x; // dangling
}
```

* **Using pointer after `free`** → use-after-free.
* **Writing through `const`** (truly const object) → UB.
* **Modifying string literals** → UB.
* **Out-of-bounds** array access → UB.
* **Aliasing violations** with `restrict` or strict aliasing → UB.

---

## 18) `extern` and multi-file organization (idiomatic pattern)

**header.h**

```c
#ifndef HEADER_H
#define HEADER_H
extern int g_count;             // declaration (no storage)
void inc(void);
#endif
```

**lib.c**

```c
#include "header.h"
int g_count = 0;                // single definition (storage)
void inc(void) { ++g_count; }
```

**main.c**

```c
#include <stdio.h>
#include "header.h"
int main(void) {
    inc();
    printf("%d\n", g_count);
}
```

> One **definition** per object across the entire program; many **extern declarations** allowed.

---

## 19) Function parameters and arrays

* Array parameters like `int a[]` are **adjusted** to `int *a`.
  Width qualifiers in parameters (`int a[static 10]`) can express minimum sizes to the compiler.

```c
void sum(int n, int a[static 10]); // caller must pass pointer to at least 10 ints if n>=10
```

* Qualifiers on parameter **pointers** (`const`, `restrict`) affect how the function treats the pointed-to data, but the pointer itself is **passed by value**.

---

## 20) `register` and the address-of rule

* `register int x;` forbids `&x`. Modern compilers optimize without it; consider it legacy.

---

## 21) Debugging variables

* Print **addresses** and **sizes**:

```c
#include <stdio.h>
int main(void){
    int x = 42;
    printf("&x = %p, sizeof(x) = %zu\n", (void*)&x, sizeof x);
}
```

* Initialize to **known values** (e.g., 0 or a pattern) to catch logic errors.
* Use sanitizers (`-fsanitize=address,undefined`) and warnings (`-Wall -Wextra -Wpedantic`).

---

## 22) Practical patterns

### Persistent function-local state

```c
int next_id(void) {
    static int id = 0; // retains value across calls
    return ++id;
}
```

### Read-only tables

```c
static const char *const months[] = {
    "Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"
};
```

### Compound literals (C99)

```c
#include <stdio.h>
struct P { int x, y; };
void printp(struct P p){ printf("(%d,%d)\n", p.x, p.y); }
int main(void){
    printp((struct P){ .x=1, .y=2 }); // temporary unnamed object
}
```

---

## 23) Memory diagrams (text)

```
int x = 5;            // automatic
&x ──> [05 00 00 00]  // little-endian, 4 bytes

static int s;         // static, zero-initialized in .bss
&s ──> [00 00 00 00]

char name[] = "Ada";  // automatic array: ['A','d','a','\0']
name ──> [41 64 61 00]
```

> Endianness affects byte order in memory, not the abstract value.

---

## 24) Checklist & best practices

* ✅ Initialize variables before use (especially locals).
* ✅ Prefer **narrowest correct type**; use `<stdint.h>` for fixed widths.
* ✅ Minimize variable scope (declare where used).
* ✅ Use `const` to document intent and enable optimizations.
* ✅ Use `static` for internal linkage or persistent locals, deliberately.
* ✅ Be precise with pointer qualifiers: `const`, `restrict`, `_Atomic`.
* ✅ Use `extern` in headers; put **exactly one** definition in a `.c` file.
* ✅ For shared state across threads, use **atomics** or **locks**—not `volatile`.

---

## 25) Mini exercises

1. **Why is this UB? Fix it.**

```c
int *f(void){
    int x = 7;
    return &x;
}
```

**Fix:** allocate or make it static:

```c
int *f_heap(void){ int *p = malloc(sizeof *p); *p = 7; return p; } // remember free
int *f_static(void){ static int x = 7; return &x; }                 // shared
```

2. **What prints? Why?**

```c
unsigned u = 1; int i = -1;
printf("%d\n", i < u);
```

Likely `0` because `i` converts to a large unsigned. Avoid mixing signed/unsigned.

3. **Qualifiers puzzle**

```c
void g(const int *p, int * const q);
```

* `p` → pointer to **const int** (g must not modify `*p`).
* `q` → **const pointer** to int (g can modify `*q`, not reassign `q`).


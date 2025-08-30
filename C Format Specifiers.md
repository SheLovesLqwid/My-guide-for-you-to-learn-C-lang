# 📘 C Format Specifiers – Beginner’s Guide

In C, **format specifiers** are special instructions you give to functions like `printf()` and `scanf()` that tell them how to interpret and display data.

They always begin with a **`%` symbol** and are followed by one or more characters that define:

* **Type** of data (integer, float, char, string, etc.)
* **Width** (minimum number of spaces to print)
* **Precision** (number of digits after decimal point, or characters to display)
* **Flags** (extra options like alignment, padding, or showing signs)

---

## 🔑 Basic Syntax

```
%[flags][width][.precision][length]specifier
```

Let’s break that down:

* `%` → Always starts with a percent sign
* **specifier** → Defines the type (e.g., `d` for integers, `f` for floats, `s` for strings)
* **flags** → Optional, modifies behavior (e.g., `-`, `+`, `0`)
* **width** → Optional, minimum number of characters to print
* **.precision** → Optional, digits after decimal or max string length
* **length** → Optional, modifies size (`l`, `h`, etc.)

---

## 🧩 Common Format Specifiers

| Specifier   | Meaning                   | Example Input | Example Output |
| ----------- | ------------------------- | ------------- | -------------- |
| `%d` / `%i` | Signed integer (base 10)  | `42`          | `42`           |
| `%u`        | Unsigned integer          | `42`          | `42`           |
| `%f`        | Floating-point (decimal)  | `3.14`        | `3.140000`     |
| `%s`        | String (text)             | `"Hello"`     | `Hello`        |
| `%c`        | Single character          | `'A'`         | `A`            |
| `%x` / `%X` | Hexadecimal (lower/upper) | `255`         | `ff` / `FF`    |
| `%o`        | Octal (base 8)            | `64`          | `100`          |
| `%p`        | Pointer (memory address)  | `&x`          | `0x7ffee4`     |
| `%%`        | Prints a `%` symbol       | –             | `%`            |

---

## 🟦 Example 1: Printing Integers (`%d`)

### Without Width

```c
#include <stdio.h>

int main() {
    int num1 = 1, num2 = 10, num3 = 100;

    printf("%d\n", num1);
    printf("%d\n", num2);
    printf("%d\n", num3);

    return 0;
}
```

✅ Output:

```
1
10
100
```

Here, the numbers take **only as much space as needed**.

---

### With Width (`%3d`)

```c
#include <stdio.h>

int main() {
    int num1 = 1, num2 = 10, num3 = 100;

    printf("%3d\n", num1);
    printf("%3d\n", num2);
    printf("%3d\n", num3);

    return 0;
}
```

✅ Output:

```
  1
 10
100
```

Explanation:

* `%3d` means **print the integer in at least 3 spaces**.
* If the number is shorter, it’s **padded with spaces on the left**.
* `1` gets two spaces before it, `10` gets one, `100` gets none.

---

### Left-Justified (`%-3d`)

```c
#include <stdio.h>

int main() {
    int num1 = 1, num2 = 10, num3 = 100;

    printf("%-3d<\n", num1);
    printf("%-3d<\n", num2);
    printf("%-3d<\n", num3);

    return 0;
}
```

✅ Output (notice `<` marks the boundary):

```
1  <
10 <
100<
```

Explanation:

* Adding `-` left-aligns the number.
* Spaces now go **after** the number.

---

### Leading Zeros (`%03d`)

```c
#include <stdio.h>

int main() {
    int num1 = 1, num2 = 10, num3 = 100;

    printf("%03d\n", num1);
    printf("%03d\n", num2);
    printf("%03d\n", num3);

    return 0;
}
```

✅ Output:

```
001
010
100
```

Explanation:

* `0` flag means **pad with zeros instead of spaces**.

---

### Always Show Sign (`%+d`)

```c
#include <stdio.h>

int main() {
    int num1 = 1, num2 = 10, num3 = -100;

    printf("%+d\n", num1);
    printf("%+d\n", num2);
    printf("%+d\n", num3);

    return 0;
}
```

✅ Output:

```
+1
+10
-100
```

---

## 🟩 Example 2: Floating-Point Numbers (`%f`)

By default, `%f` shows **6 digits after the decimal**.

```c
#include <stdio.h>

int main() {
    float price1 = 19.99, price2 = 1.5, price3 = -100.0;

    printf("%f\n", price1);
    printf("%f\n", price2);
    printf("%f\n", price3);

    return 0;
}
```

✅ Output:

```
19.990000
1.500000
-100.000000
```

---

### Setting Precision (`%.2f`)

```c
#include <stdio.h>

int main() {
    float price1 = 19.99, price2 = 1.5, price3 = -100.0;

    printf("%.2f\n", price1);
    printf("%.2f\n", price2);
    printf("%.2f\n", price3);

    return 0;
}
```

✅ Output:

```
19.99
1.50
-100.00
```

Explanation:

* `.2` → Show **2 digits after the decimal**.
* Great for money values!

---

### Rounding Example

```c
#include <stdio.h>

int main() {
    float price = 19.99;
    printf("%.1f\n", price);
    return 0;
}
```

✅ Output:

```
20.0
```

Notice how it **rounded up**.

---

### Width + Precision (`%7.2f`)

```c
#include <stdio.h>

int main() {
    float price1 = 19.99, price2 = 1.5, price3 = -100.0;

    printf("%7.2f\n", price1);
    printf("%7.2f\n", price2);
    printf("%7.2f\n", price3);

    return 0;
}
```

✅ Output:

```
  19.99
   1.50
-100.00
```

Explanation:

* `7.2` → At least **7 spaces wide**, with **2 decimals**.
* Numbers align neatly in a column.

---

### Adding a Sign (`%+7.2f`)

```c
#include <stdio.h>

int main() {
    float price1 = 19.99, price2 = 1.5, price3 = -100.0;

    printf("%+7.2f\n", price1);
    printf("%+7.2f\n", price2);
    printf("%+7.2f\n", price3);

    return 0;
}
```

✅ Output:

```
  +19.99
   +1.50
-100.00
```

---

## 🟨 Quick Reference – Flags

| Flag    | Meaning                   | Example     | Output  |
| ------- | ------------------------- | ----------- | ------- |
| `-`     | Left-align                | `%-5d` (12) | `12   ` |
| `+`     | Show sign                 | `%+d` (12)  | `+12`   |
| `0`     | Pad with zeros            | `%04d` (12) | `0012`  |
| (space) | Add space before positive | `% d` (12)  | ` 12`   |

---

## 📝 Summary

* `%d`, `%f`, `%s`, `%c` are the most common.
* Use **width** for spacing/alignment.
* Use **precision** for decimals/rounding.
* Use **flags** (`+`, `-`, `0`) for sign, alignment, or padding.
* Always match the correct specifier to the correct data type.


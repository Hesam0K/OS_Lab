# جلسه ۸: کامپایل و دیباگ برنامه‌های C در لینوکس

## مقدمه: شروع بخش جدید

تا امروز یاد گرفتیم چگونه با ابزارهای موجود سیستم‌عامل کار کنیم و آنها را با اسکریپت به هم متصل کنیم. از امروز یاد می‌گیریم چگونه با استفاده از زبان **C**، ابزارهای جدیدی بسازیم که مستقیماً با هسته سیستم‌عامل صحبت می‌کنند.

---

## بخش اول: مقدمه و مباحث تئوری

### ۱. چرا برنامه‌نویسی C؟

#### الف) زبان سیستمی:
زبان C به عنوان زبان اصلی توسعه سیستم‌عامل‌های یونیکس و لینوکس شناخته می‌شود. هسته لینوکس، درایورهای سخت‌افزاری و بسیاری از ابزارهای سیستمی به زبان C نوشته شده‌اند.

#### ب) سرعت و کارایی:
C یک زبان **کامپایلری** است و کد آن مستقیماً به زبان ماشین تبدیل می‌شود که باعث اجرای بسیار سریع آن می‌شود. برخلاف زبان‌های تفسیری (مانند Python)، که در زمان اجرا تفسیر می‌شوند.

#### ج) دسترسی سطح پایین:
C به ما اجازه می‌دهد که به صورت مستقیم با مفاهیم سیستمی مانند حافظه، فرآیندها و فایل‌ها کار کنیم. این همان کاری است که در جلسات آینده با فراخوانی‌های سیستمی (System Calls) انجام خواهیم داد.

---

### ۲. فرآیند کامپایل یک برنامه C

وقتی ما دستور ساده `gcc program.c` را اجرا می‌کنیم، در پشت صحنه **چهار مرحله اصلی** اتفاق می‌افتد. درک این مراحل بسیار مهم است.

```
hello.c (source code)
   ↓
[۱. Preprocessing] → hello.i (preprocessed source)
   ↓
[۲. Compilation] → hello.s (assembly code)
   ↓
[۳. Assembly] → hello.o (object file)
   ↓
[۴. Linking] → hello (executable)
```

#### مرحله ۱: پیشپردازش (Preprocessing)

```bash
gcc -E hello.c    # Only preprocessing, don't compile
```

**کاری که انجام می‌دهد:**
- پیشپردازنده (`cpp`) فایل کد را می‌خواند
- دستورات `#include` را با محتوای فایل‌های هِدِر جایگزین می‌کند
- دستورات `#define` (ماکروها) را جایگزین می‌کند
- دستورات شرطی پیشپردازش (`#if`, `#ifdef`) را پردازش می‌کند

**خروجی:** یک فایل کد C است (که معمولاً با پسوند `.i` ذخیره می‌شود)

**مثال:**
```c
// Original code: hello.c
#include <stdio.h>
#define NAME "World"

int main() {
    printf("Hello %s\n", NAME);
}
```

بعد از preprocessing:
```c
// Thousands of lines from stdio.h will be inserted here
// ... (contents of stdio.h)

int main() {
    printf("Hello %s\n", "World");  // NAME replaced with "World"
}
```

#### مرحله ۲: کامپایل (Compilation)

```bash
gcc -S hello.c    # Only compile, don't assemble
```

**کاری که انجام می‌دهد:**
- کامپایلر (gcc در این مرحله) کد C را به کد **اسمبلی** تبدیل می‌کند
- کد اسمبلی یک زبان سطح پایین و قابل فهم برای انسان است
- هر دستور اسمبلی به یک یا چند دستور CPU نقشه می‌شود

**خروجی:** فایلی با پسوند `.s` (Assembly source)

**مثال کد اسمبلی:**
```asm
.section .data
    fmt: .asciz "Hello World\n"

.section .text
    .global main
main:
    push rbp
    mov rsp, rbp
    lea rdi, [rel fmt]
    mov al, 0
    call printf
    mov eax, 0
    pop rbp
    ret
```

#### مرحله ۳: اسمبلی (Assembly)

```bash
gcc -c hello.c    # Compile and assemble, but don't link
```

**کاری که انجام می‌دهد:**
- اسمبلر (`as`) کد اسمبلی را به **کد ماشین** یا **آبجکت کد** تبدیل می‌کند
- این کد دیگر برای انسان قابل خواندن نیست
- به زبان صفر و یک پردازنده است

**خروجی:** فایلی با پسوند `.o` (Object file)

**نکته:** این فایل هنوز قابل اجرا نیست! (اگر `file hello.o` اجرا کنید، می‌گوید "ELF 64-bit LSB relocatable")

#### مرحله ۴: پیوند (Linking)

```bash
gcc hello.o -o hello    # Link object file to create executable
```

**کاری که انجام می‌دهد:**
- لینکر (`ld`) آبجکت فایل ما (`.o`) را با کدهای کتابخانه‌ای که از آنها استفاده کردیم ترکیب می‌کند
- مثلاً تابع `printf` در کتابخانه استاندارد C (`libc`) قرار دارد
- لینکر این تابع را پیدا کرده و به برنامه ما وصل می‌کند
- تمام **reference**های نامشخص را برطرف می‌کند

**خروجی:** یک فایل **اجرایی** کامل!

**نکته:** وقتی `gcc hello.c -o hello` اجرا می‌کنیم، gcc خودکار این ۴ مرحله را انجام می‌دهد.

---

### ۳. فایل‌های میانی و آنها را حفظ کردن

```bash
# Keep preprocessed file
gcc -save-temps hello.c

# This creates: hello.i, hello.s, hello.o, and hello
ls -la hello.*
```

---

## بخش دوم: کار عملی با ابزارهای برنامه‌نویسی

### ۱. کامپایلر GCC (GNU Compiler Collection)

#### گام ۱: نوشتن اولین برنامه C

```bash
vim hello.c
```

```c
#include <stdio.h>

int main() {
    printf("Hello from the C world!\n");
    return 0;
}
```

**توضیح:**
- `#include <stdio.h>`: شامل کتابخانه ورودی/خروجی استاندارد
- `int main()`: تابع main - نقطه شروع برنامه
- `printf()`: تابع برای چاپ متن
- `return 0`: نشان‌دهنده موفقیت برنامه

#### گام ۲: کامپایل کردن

**روش سادگی (توصیه نمی‌شود):**
```bash
gcc hello.c
# Creates: a.out
```

**روش بهتر (با مشخص کردن نام فایل خروجی):**
```bash
gcc -o hello hello.c
# Creates: hello
```

#### گام ۳: اجرای برنامه

```bash
./hello
# Output: Hello from the C world!
```

#### گام ۴: آپشن‌های مهم GCC

| آپشن | توضیح |
|------|------|
| `-o <name>` | نام فایل خروجی را مشخص کن |
| `-Wall` | تمام هشدارهای رایج (Warnings) را فعال کن |
| `-Wextra` | هشدارهای بیشتری را فعال کن |
| `-g` | اطلاعات دیباگ را به فایل اجرایی اضافه کن (برای GDB) |
| `-O0` | بدون optimization (برای دیباگ) |
| `-O2` | optimization متوسط (برای production) |
| `-c` | فقط compile کن تا فایل `.o` (بدون linking) |
| `-E` | فقط preprocessing (بدون compilation) |
| `-S` | فقط تا اسمبلی (بدون assembly) |
| `-I<dir>` | path برای جستجو کردن header files |
| `-L<dir>` | path برای جستجو کردن libraries |
| `-l<lib>` | link with specified library |

#### مثال: کامپایل کردن با تمام آپشن‌های مهم

```bash
gcc -Wall -g -o hello hello.c
```

---

### ۲. دیباگر GDB (GNU Debugger)

**چرا دیباگ کردن لازم است؟**
- پیدا کردن و درک باگ‌ها
- مشاهده مقدار متغیرها در هر نقطه
- مشاهده فلو برنامه
- شناسایی خطاهای منطقی

#### گام ۱: نوشتن برنامه با باگ

```bash
vim buggy.c
```

```c
#include <stdio.h>

int main() {
    int a = 5;
    int b = 10;
    int sum = a + b;  // THIS IS A BUG! Should be a * b
    
    printf("The product of %d and %d is %d\n", a, b, sum);
    return 0;
}
```

#### گام ۲: کامپایل کردن با فلگ دیباگ

**نکته مهم:** فلگ `-g` را باید داشته باشیم!

```bash
gcc -g -Wall -o buggy buggy.c
```

#### گام ۳: شروع GDB

```bash
gdb ./buggy
# یا
gdb
(gdb) file ./buggy
```

#### گام ۴: دستورات اصلی GDB

| دستور | کوتاه | توضیح |
|-------|-------|-------|
| `break main` | `b main` | breakpoint در ابتدای main |
| `break 5` | `b 5` | breakpoint در خط ۵ |
| `run` | `r` | برنامه را شروع کن تا breakpoint |
| `next` | `n` | به خط بعدی برو (وارد توابع نمی‌شود) |
| `step` | `s` | به خط بعدی برو (وارد توابع می‌شود) |
| `continue` | `c` | ادامه تا breakpoint بعدی یا پایان |
| `print var` | `p var` | مقدار متغیر را چاپ کن |
| `print &var` | `p &var` | آدرس متغیر را چاپ کن |
| `watch var` | - | زمانی که `var` تغییر کند توقف کن |
| `info breakpoints` | `info b` | تمام breakpoint‌ها را نمایش بده |
| `delete 1` | - | breakpoint ۱ را حذف کن |
| `backtrace` | `bt` | فلو فراخوانی توابع را نمایش بده |
| `quit` | `q` | از GDB خارج شو |

#### مثال استفاده از GDB

```bash
$ gdb ./buggy

(gdb) break main
Breakpoint 1 at 0x1129: file buggy.c, line 4.

(gdb) run
Starting program: ./buggy 

Breakpoint 1, main () at buggy.c:4
4	    int a = 5;

(gdb) next
5	    int b = 10;

(gdb) next
6	    int sum = a + b;

(gdb) print a
$1 = 5

(gdb) print b
$2 = 10

(gdb) next
8	    printf("The product of %d and %d is %d\n", a, b, sum);

(gdb) print sum
$3 = 15    # WRONG! Should be 50

(gdb) continue
The product of 5 and 10 is 15
[Inferior 1 (process 12345) exited normally]

(gdb) quit
```

#### تفاوت `next` و `step`

```c
#include <stdio.h>

void print_value(int x) {
    printf("Value: %d\n", x);
}

int main() {
    int a = 5;
    print_value(a);  // <-- اگر breakpoint اینجا باشد
    return 0;
}
```

**اگر `next` اجرا کنیم:**
- تابع `print_value` کاملاً اجرا می‌شود
- cursor به خط `return 0` می‌رود

**اگر `step` اجرا کنیم:**
- وارد تابع `print_value` می‌شویم
- cursor به خط اول تابع می‌رود

**تخلاصه:**
- `next` = روی تابع‌ها "عبور کن"
- `step` = داخل تابع‌ها "وارد شو"

---

### ۳. ابزار ساخت خودکار: Make و Makefile

#### مشکل: کامپایل دستی

در پروژه‌های بزرگ با دهها فایل و گزینه‌های متنوع:
- کامپایل دستی خسته‌کننده است
- مستعد اشتباهات است
- اگر یک فایل تغییر کند، باید دوباره همه‌ چیز کامپایل کنیم؟

#### راه‌حل: Make و Makefile

`make` ابزاری است که با خواندن یک فایل به نام `Makefile`، فرآیند کامپایل را خودکار می‌کند.

#### نوشتن یک Makefile ساده

```bash
vim Makefile
```

```makefile
# Variables
CC = gcc
CFLAGS = -g -Wall -Wextra
TARGET = hello

# The default rule is the first one
all: $(TARGET)

# Rule to build the executable
hello: hello.c
	$(CC) $(CFLAGS) -o hello hello.c

# Clean rule
clean:
	rm -f $(TARGET)

# Phony targets (not actual files)
.PHONY: all clean
```

**توضیح:**

1. **متغیرها:**
   - `CC = gcc`: نام کامپایلر
   - `CFLAGS = ...`: فلگ‌های کامپایلر
   - `TARGET = hello`: نام فایل خروجی

2. **Rules (قانون‌ها):**
   ```
   target: dependencies
   	command
   ```

3. **مثال:**
   ```makefile
   hello: hello.c
   	gcc -g -Wall -o hello hello.c
   ```
   - `hello`: نام فایل هدف (target)
   - `hello.c`: وابستگی (dependency)
   - دستور gcc: دستوری که برای ساختن `hello` اجرا می‌شود

4. **متغیرها در rules:**
   - `$(CC)`: توسع به `gcc`
   - `$(CFLAGS)`: توسع به `-g -Wall -Wextra`

**نکته مهم:** خطوط دستور (مثل `gcc ...`) باید با یک کاراکتر **Tab** شروع شوند، نه فاصله! این یکی از رایج‌ترین اشتباهات در Makefile است.

#### استفاده از Makefile

```bash
# Build the program (runs "all" rule)
make

# Or explicitly:
make all

# Clean up
make clean

# Rebuild from scratch
make clean && make
```

#### Makefile پیشرفته‌تر

```makefile
CC = gcc
CFLAGS = -g -Wall -Wextra -O2
LDFLAGS =
LIBS = -lm

# Source files
SOURCES = main.c helper.c utils.c
OBJECTS = $(SOURCES:.c=.o)
TARGET = myprogram

# Default rule
all: $(TARGET)

# Link objects to create executable
$(TARGET): $(OBJECTS)
	$(CC) $(LDFLAGS) -o $@ $^ $(LIBS)

# Compile each .c file to .o file
%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

# Clean up
clean:
	rm -f $(OBJECTS) $(TARGET)

# Phony targets
.PHONY: all clean
```

**توضیح ویژگی‌های پیشرفته:**

1. **Pattern Rules:**
   ```makefile
   %.o: %.c
   	$(CC) $(CFLAGS) -c -o $@ $<
   ```
   - `%.o`: هر فایلی با پسوند `.o`
   - `%.c`: فایل `.c` متناظر
   - `$@`: نام target
   - `$<`: نام اول dependency

2. **Variable Substitution:**
   ```makefile
   OBJECTS = $(SOURCES:.c=.o)
   ```
   - تمام `.c` را با `.o` جایگزین کن

3. **Phony Targets:**
   ```makefile
   .PHONY: all clean
   ```
   - این targets فایل‌های واقعی نیستند

---

## خلاصه مفاهیم

| مفهوم | توضیح |
|------|-------|
| **Preprocessing** | جایگزینی `#include` و ماکروها |
| **Compilation** | تبدیل C به Assembly |
| **Assembly** | تبدیل Assembly به Machine Code (`.o`) |
| **Linking** | ترکیب `.o` فایل‌ها و کتابخانه‌ها |
| **gcc** | کامپایلر - اجرای تمام ۴ مرحله |
| **gcc -c** | تا مرحله Assembly (فایل `.o`) |
| **gcc -S** | تا مرحله Compilation (فایل `.s`) |
| **gcc -E** | تا مرحله Preprocessing (فایل `.i`) |
| **gcc -g** | اضافه کردن اطلاعات دیباگ |
| **gdb** | دیباگر برای مشاهده و کنترل برنامه |
| **make** | ابزار خودکارسازی کامپایل |

---

## نکات مهم

1. **همیشه از `-Wall` استفاده کنید:** هشدارها می‌توانند باگ‌های احتمالی را نشان بدهند
2. **برای دیباگ کردن، `-g` را فراموش نکنید:** بدون آن، GDB نمی‌تواند اطلاعات line number را نشان بدهد
3. **Tab vs Space در Makefile:** اگر خطا بگیرید، حتماً Tab استفاده کردید؟
4. **فایل‌های اجرایی را به version control نکنید:** فقط source code را save کنید

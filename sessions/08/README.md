# تمرین‌های جلسه ۸: کامپایل و دیباگ برنامه‌های C در لینوکس

---

## تمرین عملی در کالس

### تمرین اصلی: حلقه with GDB Debugging

**مسئله:**

یک برنامه C بنویسید که:
- یک حلقه `for` از ۱ تا ۱۰ دارد
- در هر بار تکرار، شماره حلقه را چاپ می‌کند
- یک `Makefile` برای آن بنویسید
- با `gdb` یک breakpoint داخل حلقه قرار دهید و مقدار متغیر شمارنده را در چند مرحله چاپ کنید

---

### جواب:

#### بخش ۱: نوشتن برنامه

```bash
vim loop.c
```

```c
#include <stdio.h>

int main() {
    for (int i = 1; i <= 10; i++) {
        printf("Iteration %d\n", i);
    }
    return 0;
}
```

#### بخش ۲: نوشتن Makefile

```bash
vim Makefile
```

```makefile
CC = gcc
CFLAGS = -g -Wall -Wextra
TARGET = loop

all: $(TARGET)

$(TARGET): loop.c
	$(CC) $(CFLAGS) -o $@ $<

clean:
	rm -f $(TARGET)

.PHONY: all clean
```

**نکات مهم:**
- فلگ `-g` برای اضافه کردن اطلاعات دیباگ
- خط دستور باید با **Tab** شروع شود
- `$@` = target (`loop`)
- `$<` = اولین dependency (`loop.c`)

#### بخش ۳: کامپایل کردن

```bash
make
# یا
make all

# خروجی: gcc -g -Wall -Wextra -o loop loop.c
```

#### بخش ۴: اجرای معمولی

```bash
./loop
# Output:
# Iteration 1
# Iteration 2
# ...
# Iteration 10
```

#### بخش ۵: استفاده از GDB

```bash
gdb ./loop
```

**جلسه GDB:**

```
(gdb) break loop.c:5
Breakpoint 1 at 0x1159: file loop.c, line 5.

# Line 5 است جایی که printf داخل حلقه است

(gdb) run
Starting program: ./loop 

Breakpoint 1, main () at loop.c:5
5	        printf("Iteration %d\n", i);

# اولین بار که به حلقه رسیدیم
(gdb) print i
$1 = 1

(gdb) next
Iteration 1

# اجرای printf
(gdb) next
4	    for (int i = 1; i <= 10; i++) {

# بازگشت به خط for

(gdb) next
5	        printf("Iteration %d\n", i);

# دوبار دوم در حلقه
Breakpoint 1, main () at loop.c:5

(gdb) print i
$2 = 2

(gdb) continue
Iteration 2
Iteration 3
Iteration 4
Iteration 5
Iteration 6
Iteration 7
Iteration 8
Iteration 9
Iteration 10

# به آخر برنامه رسیدیم

(gdb) quit
```

**خلاصه دستورات استفاده شده:**
- `break loop.c:5`: breakpoint در خط ۵
- `run`: برنامه را اجرا کن
- `print i`: مقدار متغیر `i` را نمایش بده
- `next`: به خط بعدی برو
- `continue`: ادامه تا breakpoint بعدی

---

## تمرین‌های تکلیف هفته آینده

### سوال ۱: آپشن `-c` در GCC چه کاری انجام می‌دهد؟

**مسئله:**

آپشن `-c` در دستور `gcc` چه کاری انجام می‌دهد؟ خروجی آن چه فایلی است و چه کاربردی دارد؟

---

### جواب:

#### توضیح `-c` فلگ:

آپشن `-c` دستور GCC را می‌خواهد که برنامه را **تا مرحله Assembly** کامپایل کند، اما **linking را انجام ندهد**.

```bash
gcc -c hello.c
# Output: hello.o (Object file)
```

#### خروجی: فایل Object (`.o`)

**فایل `.o` چیست؟**
- کد ماشین است (Binary code)
- برای انسان قابل خواندن نیست
- هنوز **قابل اجرا نیست!**
- به عنوان واسطه بین کد C و فایل اجرایی نهایی عمل می‌کند

#### تفاوت `-c` و بدون آن:

| دستور | مراحل | خروجی | قابل اجرا؟ |
|-------|-------|-------|----------|
| `gcc hello.c` | ۱۲۳۴ | `a.out` | ✓ بله |
| `gcc -c hello.c` | ۱۲۳ | `hello.o` | ✗ خیر |
| `gcc -S hello.c` | ۱۲ | `hello.s` | ✗ خیر |
| `gcc -E hello.c` | ۱ | `a.out` (stdout) | ✗ خیر |

#### کاربرد `-c`:

**۱. پروژه‌های چند فایله:**

فرض کنید پروژه ما دو فایل دارد:

```bash
# File 1: main.c
int add(int a, int b);

int main() {
    int result = add(5, 3);
    return 0;
}

# File 2: utils.c
int add(int a, int b) {
    return a + b;
}
```

اگر بخواهیم به صورت کلاسیک کامپایل کنیم:

```bash
gcc main.c utils.c -o myprogram
```

اما اگر فقط `utils.c` تغییر کند، باید دوباره همه چیز کامپایل کنیم!

**بهتر است:**

```bash
gcc -c main.c      # → main.o
gcc -c utils.c     # → utils.o
gcc main.o utils.o -o myprogram  # Link
```

حالا اگر فقط `utils.c` تغییر کند:
```bash
gcc -c utils.c     # فقط این
gcc main.o utils.o -o myprogram
```

`main.o` دوباره ساخته نمی‌شود!

**۲. استفاده از Makefile:**

```makefile
CC = gcc
CFLAGS = -g -Wall

main.o: main.c
	$(CC) $(CFLAGS) -c main.c

utils.o: utils.c
	$(CC) $(CFLAGS) -c utils.c

myprogram: main.o utils.o
	$(CC) -o myprogram main.o utils.o

clean:
	rm -f *.o myprogram
```

**۳. تقسیم بندی کار:**

- توسعه‌دهنده A تا `module1.o` را تکمیل می‌کند
- توسعه‌دهنده B تا `module2.o` را تکمیل می‌کند
- سپس یک نفر آنها را به هم می‌لینک می‌کند

#### نمایش عملی:

```bash
# ۱. کامپایل کردن (بدون linking)
gcc -c hello.c
ls -la hello.*

# Output:
# -rw-r--r--  1 user  group   1234 Jun 26 10:00 hello.c
# -rw-r--r--  1 user  group    456 Jun 26 10:01 hello.o

# ۲. بررسی نوع فایل
file hello.o
# Output: hello.o: ELF 64-bit LSB relocatable x86-64 version 1 (SYSV), not stripped

# "relocatable" = هنوز linked نشده
# "not stripped" = اطلاعات دیباگ دارد

# ۳. سعی کردن اجرا کردن (شکست!)
./hello.o
# Output: bash: ./hello.o: Permission denied
# (حتی اگر permission داشت، فایل executable نیست)

# ۴. Linking
gcc hello.o -o hello
./hello
# Output: Hello from the C world!
```

#### خلاصه:

| موضوع | پاسخ |
|-------|------|
| **کاری که انجام می‌دهد** | Preprocessing + Compilation + Assembly (بدون Linking) |
| **فایل خروجی** | `.o` (Object file) |
| **قابل اجرا؟** | خیر |
| **کاربرد اصلی** | کامپایل کردن فایل‌های چند فایله به صورت جداگانه |
| **فائده** | سرعت: فقط فایل‌های تغییر یافته را کامپایل کنیم |

---

### سوال ۲: تفاوت `next` و `step` در GDB

**مسئله:**

در `gdb`، تفاوت اصلی بین دستور `next` و `step` چیست؟ چه زمانی از هر کدام استفاده می‌کنیم؟

---

### جواب:

#### تفاوت اساسی:

| دستور | رفتار | استفاده برای |
|-------|-------|------------|
| `next` | خط بعدی برو (توابع را Skip کن) | جلوتر رفتن بدون دخالت در جزئیات توابع |
| `step` | خط بعدی برو (وارد توابع شو) | بررسی تفصیلی داخل توابع |

#### مثال عملی:

```c
#include <stdio.h>

int multiply(int a, int b) {
    int result = a * b;
    printf("Inside multiply: %d * %d = %d\n", a, b, result);
    return result;
}

int main() {
    int x = 5;
    int y = 10;
    int z = multiply(x, y);      // <-- LINE 11
    printf("Result: %d\n", z);
    return 0;
}
```

**سناریو ۱: استفاده از `next`**

```
(gdb) break main
(gdb) run
...
(gdb) next
5       int y = 10;

(gdb) next
6       int z = multiply(x, y);      # ← اینجا هستیم

(gdb) next
7       printf("Result: %d\n", z);   # ← دیگر وارد multiply نشدیم!
```

**خروجی:**
```
Inside multiply: 5 * 10 = 50
```

تابع `multiply` کاملاً اجرا شد و سپس به خط بعدی رفتیم.

**سناریو ۲: استفاده از `step`**

```
(gdb) break main
(gdb) run
...
(gdb) next
...

(gdb) step
multiply (a=5, b=10) at multiply.c, line 2
2       int result = a * b;         # ← وارد تابع multiply شدیم!

(gdb) step
3       printf("Inside multiply: %d * %d = %d\n", a, b, result);

(gdb) step
Inside multiply: 5 * 10 = 50
4       return result;

(gdb) step
5       int z = multiply(x, y);
```

#### متنی تر: وقتی از `step` استفاده کنیم

**حالت ۱: دیباگ کردن خود تابع**

```c
void important_function(int data) {
    // اگر این تابع اشتباه است، از step استفاده کنیم
    // تا وارد آن شویم و خط به خط مشاهده کنیم
}
```

```gdb
(gdb) break main
(gdb) run
...
(gdb) step  # وارد important_function
```

**حالت ۲: استفاده از `next` برای توابع معتبر**

```c
void valid_function(int data) {
    // این تابع معتبر است، بر اساس نام و کاری که انجام می‌دهد اعتماد داریم
}
```

```gdb
(gdb) break main
(gdb) run
...
(gdb) next  # این تابع اجرا شود و skip کنیم
```

#### مثال پیشرفته: حلقه داخل تابع

```c
void print_numbers() {
    for (int i = 1; i <= 3; i++) {
        printf("Number: %d\n", i);
    }
}

int main() {
    print_numbers();    // <- LINE 7
    printf("Done\n");
    return 0;
}
```

**اگر `next` استفاده کنیم:**
```
# Line 7 اجرا
(gdb) next
Done   # تمام حلقه اجرا شد
```

**اگر `step` استفاده کنیم:**
```
# Line 7 اجرا
(gdb) step
print_numbers () at test.c:2
2       for (int i = 1; i <= 3; i++) {

(gdb) step
4           printf("Number: %d\n", i);

(gdb) step
Number: 1
4           printf("Number: %d\n", i);

# و می‌توانیم بقیه حلقه را مرحله به مرحله ببینیم
```

#### خلاصه: کی کدام را استفاده کنیم

**استفاده از `next`:**
- ✓ اگر اطمینان داریم تابع درست کار می‌کند
- ✓ اگر می‌خواهیم سریع پیش بریم
- ✓ اگر فقط نتیجه تابع مهم است نه جزئیات

**استفاده از `step`:**
- ✓ اگر تابع خودمان نوشته‌ایم و شک داریم
- ✓ اگر می‌خواهیم ببینیم تابع داخل چه می‌کند
- ✓ اگر باگ احتمالاً داخل تابع است
- ✓ اگر یاد می‌گیریم چجوری کد کار می‌کند

#### نکات بیشتر

**۱. میانبر:**
```gdb
n    # مختصر next
s    # مختصر step
```

**۲. فشار دادن Enter:**
```
# اگر دوباره Enter را فشار دهید، آخرین دستور تکرار می‌شود
(gdb) next
5   int x = 10;

(gdb)    # فقط Enter فشار داده ایم
6   int y = 20;

# اپ "next" دوباره اجرا شد
```

**۳. combined استفاده:**
```
(gdb) next
...
(gdb) step      # وارد تابعی شدیم که شک داریم
...
(gdb) next      # اطراف حلقه move کنیم
(gdb) next
(gdb) step      # دوباره وارد تابع دیگری
```

---

### سوال ۳: پروژه چند فایله با Makefile

**مسئله:**

یک پروژه کوچک C با دو فایل ایجاد کنید:
- `main.c`: شامل تابع main
- `helper.c`: شامل توابع کمکی

تابع main توابعی را از `helper.c` فراخوانی می‌کند.

یک `Makefile` بنویسید که:
- هر دو فایل را به آبجکت فایل (`.o`) کامپایل کند
- سپس آنها را به یکدیگر لینک کند تا فایل اجرایی نهایی ساخته شود

---

### جواب:

#### بخش ۱: نوشتن `helper.h` (Header file)

```bash
vim helper.h
```

```c
#ifndef HELPER_H
#define HELPER_H

// Function declarations
int add(int a, int b);
int subtract(int a, int b);
int multiply(int a, int b);

#endif
```

**توضیح:**
- `#ifndef HELPER_H`: جلوگیری از double inclusion
- Function declarations: تابع‌ها را فقط اعلان می‌کنیم، implementation نه

#### بخش ۲: نوشتن `helper.c` (Implementation)

```bash
vim helper.c
```

```c
#include "helper.h"
#include <stdio.h>

int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}

int multiply(int a, int b) {
    return a * b;
}
```

**توضیح:**
- `#include "helper.h"`: شامل header file (با نقل‌قول)
- توابع را پیاده‌سازی می‌کنیم

#### بخش ۳: نوشتن `main.c`

```bash
vim main.c
```

```c
#include <stdio.h>
#include "helper.h"

int main() {
    int a = 10;
    int b = 5;
    
    printf("a = %d, b = %d\n", a, b);
    printf("add(%d, %d) = %d\n", a, b, add(a, b));
    printf("subtract(%d, %d) = %d\n", a, b, subtract(a, b));
    printf("multiply(%d, %d) = %d\n", a, b, multiply(a, b));
    
    return 0;
}
```

#### بخش ۴: نوشتن `Makefile`

```bash
vim Makefile
```

```makefile
# Variables
CC = gcc
CFLAGS = -g -Wall -Wextra
TARGET = calculator

# Object files
OBJECTS = main.o helper.o

# Default rule
all: $(TARGET)

# Link rule: دستور نهایی برای ایجاد executable
$(TARGET): $(OBJECTS)
	$(CC) $(CFLAGS) -o $@ $^

# Compile main.c to main.o
main.o: main.c helper.h
	$(CC) $(CFLAGS) -c main.c

# Compile helper.c to helper.o
helper.o: helper.c helper.h
	$(CC) $(CFLAGS) -c helper.c

# Clean rule
clean:
	rm -f $(OBJECTS) $(TARGET)

# Phony targets
.PHONY: all clean
```

**توضیح Makefile:**

1. **متغیرها:**
   - `CC = gcc`: کامپایلر
   - `CFLAGS`: فلگ‌ها (include `-g` برای دیباگ)
   - `TARGET`: نام نهایی
   - `OBJECTS`: فایل‌های object

2. **Rules:**

   ```makefile
   all: $(TARGET)
   ```
   - Rule پیشفرض (اگر صرفاً `make` بنویسیم)

   ```makefile
   $(TARGET): $(OBJECTS)
       $(CC) $(CFLAGS) -o $@ $^
   ```
   - `calculator` به `main.o` و `helper.o` وابسته است
   - `$@` = `calculator` (target)
   - `$^` = `main.o helper.o` (تمام dependencies)

   ```makefile
   main.o: main.c helper.h
       $(CC) $(CFLAGS) -c main.c
   ```
   - `main.o` به `main.c` و `helper.h` وابسته است

   ```makefile
   helper.o: helper.c helper.h
       $(CC) $(CFLAGS) -c helper.c
   ```

3. **Clean rule:**
   - تمام تولیدات را پاک می‌کند

#### نسخه بهتر‌تر: Pattern Rules

```makefile
CC = gcc
CFLAGS = -g -Wall -Wextra
TARGET = calculator

SOURCES = main.c helper.c
OBJECTS = $(SOURCES:.c=.o)

all: $(TARGET)

$(TARGET): $(OBJECTS)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

clean:
	rm -f $(OBJECTS) $(TARGET)

.PHONY: all clean
```

**توضیح Pattern Rule:**

```makefile
%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<
```

- `%.o`: هر فایل با پسوند `.o`
- `%.c`: فایل `.c` متناظر
- `$@` = نام target (مثلاً `main.o`)
- `$<` = اولین dependency (مثلاً `main.c`)

#### استفاده:

```bash
# Step 1: ایجاد
make

# خروجی:
# gcc -g -Wall -Wextra -c -o main.o main.c
# gcc -g -Wall -Wextra -c -o helper.o helper.c
# gcc -g -Wall -Wextra -o calculator main.o helper.o

# Step 2: اجرا
./calculator

# خروجی:
# a = 10, b = 5
# add(10, 5) = 15
# subtract(10, 5) = 5
# multiply(10, 5) = 50

# Step 3: تغییر و بازسازی
# اگر فقط helper.c تغییر کند:
vim helper.c

make

# خروجی:
# gcc -g -Wall -Wextra -c -o helper.o helper.c
# gcc -g -Wall -Wextra -o calculator main.o helper.o
# (main.o دوباره compile نشد!)

# Step 4: پاکیزه کردن
make clean

# خروجی:
# rm -f main.o helper.o calculator
```

#### بررسی فایل‌ها:

```bash
# بعد از make:
ls -la

# خروجی:
# -rw-r--r--  Makefile
# -rw-r--r--  main.c
# -rw-r--r--  main.o        ← object file
# -rw-r--r--  helper.c
# -rw-r--r--  helper.o      ← object file
# -rw-r--r--  helper.h
# -rwxr-xr-x  calculator    ← executable
```

#### استفاده از GDB برای این پروژه:

```bash
# با فلگ -g کامپایل شده (توسط Makefile)، می‌توانیم دیباگ کنیم:

gdb ./calculator

(gdb) break main
(gdb) run
(gdb) next
(gdb) step
# وارد توابع helper می‌شویم
(gdb) print a
$1 = 10
(gdb) continue
...
```

#### نکات مهم:

1. **Header Guards:** `#ifndef ... #define ... #endif` جلوگیری از مشکل double inclusion

2. **Local includes:** `#include "helper.h"` (نقل‌قول) برای فایل‌های محلی

3. **Compilation شامل:**
   - `main.c` به `main.o`
   - `helper.c` به `helper.o`
   - Linking `main.o` و `helper.o` → `calculator`

4. **Makefile Dependencies:** اگر `helper.h` تغییر کند، هر دو فایل دوباره compile می‌شود

5. **Incremental Compilation:** اگر فقط `helper.c` تغییر کند، فقط آن compile می‌شود (سرعت!)

---

## خلاصه جلسه ۸

| مفهوم | نکات کلیدی |
|------|-----------|
| **۴ مرحله compile** | Preprocessing → Compilation → Assembly → Linking |
| **gcc -c** | تا مرحله assembly (فایل `.o`) |
| **gcc -g** | برای دیباگ با GDB |
| **GDB: next** | خط بعدی (توابع skip) |
| **GDB: step** | خط بعدی (داخل توابع) |
| **Makefile** | خودکار‌سازی کامپایل پروژه‌های چند فایله |
| **Pattern Rules** | `%.o: %.c` برای تعدادی فایل |
| **Tab vs Space** | دستورات Makefile باید با Tab شروع شوند |

---

**توجه:** تمام فایل‌های C را کامپایل کنید و نتیجه را تست کنید!

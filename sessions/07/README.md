# تمرین‌های جلسه ۷: اسکریپتنویسی پیشرفته (ساختارهای کنترلی)

---

## تمرین عملی در کالس

### تمرین اصلی: تشخیص زوج یا فرد

**مسئله:**
اسکریپتی بنویسید که یک عدد به عنوان آرگومان دریافت کند و تشخیص دهد که آن عدد زوج است یا فرد.

**راهنمایی:** از عملگر باقیمانده `%` در محاسبات استفاده کنید.

---

### جواب:

```bash
#!/bin/bash
# Script to check if a number is even or odd

if [[ $# -ne 1 ]]; then
    echo "Error: Please provide exactly one number as argument."
    echo "Usage: $0 <number>"
    exit 1
fi

NUM=$1

# Check if the input is a valid number
if ! [[ "$NUM" =~ ^[0-9]+$ ]]; then
    echo "Error: '$NUM' is not a valid number."
    exit 1
fi

# Check if the number is even or odd
if [[ $((NUM % 2)) -eq 0 ]]; then
    echo "$NUM is an EVEN number."
else
    echo "$NUM is an ODD number."
fi
```

**توضیح:**

1. **`if [[ $# -ne 1 ]]; then`**: بررسی می‌کند که دقیقاً یک آرگومان داده شده است
   - `$#` = تعداد آرگومان‌ها
   - `-ne` = نابرابر (not equal)
   - اگر نه، پیغام خطا نشان می‌دهد

2. **`NUM=$1`**: عدد ورودی را در متغیر `NUM` ذخیره می‌کند

3. **`[[ "$NUM" =~ ^[0-9]+$ ]]`**: بررسی می‌کند که ورودی یک عدد معتبر است
   - `=~` = عملگر regex matching
   - `^[0-9]+$` = تنها اعداد مثبت

4. **`if [[ $((NUM % 2)) -eq 0 ]]; then`**: بررسی می‌کند که آیا عدد زوج است
   - `$((NUM % 2))` = محاسبه باقیمانده (Modulo)
   - اگر باقیمانده 0 باشد = **زوج**
   - اگر باقیمانده 1 باشد = **فرد**

**تست کردن:**

```bash
chmod +x even_odd.sh

./even_odd.sh 10          # Output: 10 is an EVEN number.
./even_odd.sh 7           # Output: 7 is an ODD number.
./even_odd.sh             # Error: Please provide exactly one number
./even_odd.sh abc         # Error: 'abc' is not a valid number
./even_odd.sh 5 6         # Error: Please provide exactly one number
```

---

## تمرین‌های تکلیف هفته آینده

### سوال ۱: اسکریپت بررسی دقیق آرگومان و فایل

**مسئله:**

اسکریپتی بنویسید که:
- دقیقاً یک آرگومان (نام یک فایل) را بپذیرد
- اگر تعداد آرگومان‌ها صفر یا بیشتر از یک باشد، پیغام خطا چاپ کند و با کد خروج `1` از اسکریپت خارج شود
- اگر آرگومان درست باشد، بررسی کند که آیا آن فایل وجود دارد یا نه

---

### جواب:

```bash
#!/bin/bash
# Script to check if a file exists - with strict argument validation

# Check if exactly one argument is provided
if [[ $# -ne 1 ]]; then
    echo "Error: This script requires exactly one argument (filename)."
    echo "Usage: $0 <filename>"
    exit 1
fi

# Get the filename from argument
FILENAME="$1"

# Check if the file exists
if [[ -e "$FILENAME" ]]; then
    # Further check if it's a regular file, directory, or something else
    if [[ -f "$FILENAME" ]]; then
        echo "✓ '$FILENAME' is a regular file and exists."
    elif [[ -d "$FILENAME" ]]; then
        echo "✓ '$FILENAME' is a directory and exists."
    else
        echo "✓ '$FILENAME' exists but is neither a regular file nor a directory (might be a symlink, device, etc.)"
    fi
else
    echo "✗ Error: '$FILENAME' does not exist."
    exit 2  # Different exit code to indicate file not found
fi
```

**توضیح تفصیلی:**

1. **اول: بررسی تعداد آرگومان‌ها**
   ```bash
   if [[ $# -ne 1 ]]; then
   ```
   - اگر تعداد آرگومان‌ها برابر با ۱ نباشد، خطا رخ می‌دهد
   - `exit 1` = خروج با کد خطا (طبق سوال)

2. **دوم: ذخیره آرگومان در متغیر**
   ```bash
   FILENAME="$1"
   ```
   - از Double Quote استفاده می‌کنیم (اگر فایل نام فاصله دار داشته باشد)

3. **سوم: بررسی وجود فایل**
   ```bash
   if [[ -e "$FILENAME" ]]; then
   ```
   - `-e` = چک می‌کند که آیا فایل یا دایرکتوری وجود دارد

4. **چهارم: تفسیر نوع موجودیت**
   - `-f` = فایل معمولی
   - `-d` = دایرکتوری
   - دیگری = سایر انواع

**تست کردن:**

```bash
chmod +x file_checker.sh

# تست ۱: بدون آرگومان (خطا)
./file_checker.sh
# Output: Error: This script requires exactly one argument
# Exit code: 1

# تست ۲: بیش از یک آرگومان (خطا)
./file_checker.sh file1.txt file2.txt
# Output: Error: This script requires exactly one argument
# Exit code: 1

# تست ۳: فایل موجود
touch test_file.txt
./file_checker.sh test_file.txt
# Output: ✓ 'test_file.txt' is a regular file and exists.
# Exit code: 0

# تست ۴: دایرکتوری موجود
./file_checker.sh /etc
# Output: ✓ '/etc' is a directory and exists.
# Exit code: 0

# تست ۵: فایل غیر موجود (خطا)
./file_checker.sh nonexistent.txt
# Output: ✗ Error: 'nonexistent.txt' does not exist.
# Exit code: 2
```

**نکات مهم:**

- **کد خروج مختلف برای شرایط مختلف**: کد `1` برای مشکل آرگومان، کد `2` برای فایل نیافته
- این مفید است زمانی که اسکریپت را از اسکریپت دیگری فراخوانی کنیم و بخواهیم بدانیم چی اشتباه شد
- از `"$FILENAME"` استفاده کردیم (نه `$FILENAME`) برای محافظت از فاصله‌ها

---

### سوال ۲: فهم اهمیت Double Quotes و متغیرهای خالی

**مسئله:**

چرا در شرط `[ "$VAR" = "$value" ]` استفاده از دابل کوتیشن دور متغیر (`$VAR`) یک عادت خوب و ضروری است؟
اگر متغیر `$VAR` خالی یا حاوی فاصله باشد و کوتیشن نگذاریم چه اتفاقی می‌افتد؟

---

### جواب:

#### مسئله ۱: متغیر خالی بدون Double Quotes

```bash
#!/bin/bash

# بدی: بدون Double Quotes
VAR=""

# این خطا می‌دهد!
if [ $VAR = "hello" ]; then
    echo "VAR is hello"
fi

# خروجی خطا:
# test: =: unary operator expected
```

**دلیل:** وقتی `$VAR` خالی است، bash این را می‌بیند:
```bash
if [ = "hello" ]; then
```

باش یک عملگر بدون operand‌های کافی و خطا می‌دهد.

#### حل: استفاده از Double Quotes

```bash
#!/bin/bash

VAR=""

# درست: با Double Quotes
if [[ "$VAR" = "hello" ]]; then
    echo "VAR is hello"
else
    echo "VAR is not hello"
fi

# خروجی: VAR is not hello
```

#### مسئله ۲: متغیر حاوی فاصله بدون Double Quotes

```bash
#!/bin/bash

# بدی: بدون Double Quotes
VAR="hello world"

# این خطا می‌دهد!
if [ $VAR = "hello world" ]; then
    echo "Match"
fi

# خروجی خطا:
# test: world: command not found
# test: too many arguments
```

**دلیل:** bash تفسیر می‌کند:
```bash
if [ hello world = "hello world" ]; then
```

و `world` را یک آرگومان اضافی تلقی می‌کند.

#### حل: Double Quotes

```bash
#!/bin/bash

VAR="hello world"

# درست: با Double Quotes
if [[ "$VAR" = "hello world" ]]; then
    echo "Match!"
else
    echo "No match"
fi

# خروجی: Match!
```

---

### مثال جامع: مقایسه غلط vs درست

```bash
#!/bin/bash

# Demonstration: why double quotes matter

echo "=== Test 1: Empty Variable ==="

EMPTY=""

echo "❌ Without quotes (will fail):"
if [ $EMPTY = "test" ] 2>&1; then
    echo "EMPTY equals test"
else
    echo "Cannot compare (syntax error)"
fi

echo ""
echo "✓ With quotes (works):"
if [[ "$EMPTY" = "test" ]]; then
    echo "EMPTY equals test"
else
    echo "EMPTY does not equal test"
fi

echo ""
echo "=== Test 2: Variable with Spaces ==="

SPACED="hello world"

echo "❌ Without quotes (will fail):"
if [ $SPACED = "hello world" ] 2>&1; then
    echo "Match"
else
    echo "No match"
fi

echo ""
echo "✓ With quotes (works):"
if [[ "$SPACED" = "hello world" ]]; then
    echo "Match!"
else
    echo "No match"
fi

echo ""
echo "=== Test 3: Variable with Special Characters ==="

SPECIAL="test*file"

echo "❌ Without quotes (dangerous - globbing):"
echo "Value of SPECIAL: $SPECIAL"
if [ $SPECIAL = "test*file" ] 2>&1; then
    echo "Match"
else
    echo "No match (might expand globs!)"
fi

echo ""
echo "✓ With quotes (safe):"
if [[ "$SPECIAL" = "test*file" ]]; then
    echo "Match!"
else
    echo "No match"
fi
```

---

### خلاصه نقاط کلیدی:

| وضعیت | بدون Quotes | با Quotes |
|------|-----------|----------|
| **متغیر خالی** | ❌ خطا (syntax error) | ✓ کار می‌کند |
| **متغیر با فاصله** | ❌ خطا (too many arguments) | ✓ کار می‌کند |
| **متغیر با `*` یا `?`** | ❌ خطر (globbing می‌شود) | ✓ کار می‌کند |
| **متغیر عادی** | ⚠ کار می‌کند (اما نه امن) | ✓ همیشه امن |

**نتیجه‌گیری:** استفاده از Double Quotes یک عادت دفاعی ضروری است!

---

### سوال ۳: لیست کردن تمام فایل‌ها و دایرکتوری‌ها با حلقه `for`

**مسئله:**

با استفاده از حلقه `for`، اسکریپتی بنویسید که:
- تمام فایل‌های موجود در دایرکتوری فعلی را لیست کند
- برای هر کدام مشخص کند که فایل است یا دایرکتوری

---

### جواب:

```bash
#!/bin/bash
# Script to list all files and directories in current directory

echo "=== Files and Directories in $(pwd) ==="
echo ""

# Count files and directories
FILE_COUNT=0
DIR_COUNT=0

# Loop through all items in current directory
for ITEM in *
do
    if [[ -f "$ITEM" ]]; then
        echo "📄 FILE:      $ITEM"
        ((FILE_COUNT++))
    elif [[ -d "$ITEM" ]]; then
        echo "📁 DIRECTORY: $ITEM/"
        ((DIR_COUNT++))
    else
        echo "❓ OTHER:     $ITEM"
    fi
done

echo ""
echo "=== Summary ==="
echo "Total Files:       $FILE_COUNT"
echo "Total Directories: $DIR_COUNT"
echo "Total Items:       $((FILE_COUNT + DIR_COUNT))"
```

**توضیح:**

1. **`for ITEM in *`**: حلقه روی تمام آیتم‌های دایرکتوری فعلی
   - `*` = وحش wildcard (تمام فایل‌ها)

2. **`if [[ -f "$ITEM" ]]`**: بررسی می‌کند که آیا یک فایل معمولی است

3. **`elif [[ -d "$ITEM" ]]`**: بررسی می‌کند که آیا یک دایرکتوری است

4. **شمارش با `((FILE_COUNT++))`**: افزایش شمارنده (Bash arithmetic)

5. **نمایش نتایج**: در انتهای حلقه آمار کل را چاپ می‌کند

**تست کردن:**

```bash
# ابتدا فایل‌ها و دایرکتوری‌های تست بسازید:
mkdir test_dir1 test_dir2
touch file1.txt file2.sh file3.md
ln -s file1.txt symlink.txt

chmod +x list_items.sh

# اسکریپت را اجرا کنید:
./list_items.sh

# خروجی مثال:
# === Files and Directories in /home/user/testdir ===
# 
# 📄 FILE:      file1.txt
# 📄 FILE:      file2.sh
# 📄 FILE:      file3.md
# 📄 FILE:      list_items.sh
# 📄 FILE:      symlink.txt
# 📁 DIRECTORY: test_dir1/
# 📁 DIRECTORY: test_dir2/
# 
# === Summary ===
# Total Files:       5
# Total Directories: 2
# Total Items:       7
```

---

### نسخه بهتر: با نمایش اطلاعات بیشتر

اگر بخواهیم اطلاعات بیشتری نمایش دهیم (مثل اندازه فایل):

```bash
#!/bin/bash
# Advanced version: Show more details about each item

echo "=== Detailed List of $(pwd) ==="
echo ""
printf "%-30s %-10s %s\n" "NAME" "TYPE" "SIZE"
printf "%-30s %-10s %s\n" "----" "----" "----"

for ITEM in *
do
    if [[ -f "$ITEM" ]]; then
        SIZE=$(du -h "$ITEM" | cut -f1)
        printf "%-30s %-10s %s\n" "$ITEM" "File" "$SIZE"
    elif [[ -d "$ITEM" ]]; then
        SIZE=$(du -sh "$ITEM" | cut -f1)
        printf "%-30s %-10s %s\n" "$ITEM/" "Directory" "$SIZE"
    fi
done
```

**مثال خروجی:**
```
=== Detailed List of /home/user/project ===

NAME                           TYPE       SIZE
----                           ----       ----
README.md                      File       4.5K
main.sh                        File       2.3K
config.txt                     File       1.2K
data/                          Directory  128M
scripts/                       Directory  45M
```

---

### نکات مهم:

1. **استفاده از Double Quotes**: `"$ITEM"` برای جلوگیری از مشکلات فاصله‌ها
2. **بررسی نوع**: از `-f` و `-d` برای تمایز فایل و دایرکتوری
3. **شمارش**: استفاده از arithmetic `((...))` برای افزایش شمارنده
4. **نمایش بهتر**: استفاده از emoji و formatting برای خوانایی بیشتر

---

## خلاصه مفاهیم جلسه ۷

| موضوع | نکات کلیدی |
|------|-----------|
| **آرگومان‌ها** | `$#`, `$1`, `$@` را برای کنترل ورودی استفاده کنید |
| **کد خروج** | `$?` برای چک کردن موفقیت دستور قبلی |
| **شرطی (`if`)** | از `[[ ]]` استفاده کنید (برای bash امن‌تر) |
| **حلقه (`for`)** | برای تکرار بر روی لیست‌ها |
| **Double Quotes** | محافظت از متغیرهای خالی و فاصله‌دار |
| **خروج صحیح** | از کد‌های خروج معنادار استفاده کنید |

---

**توجه:** تمام این اسکریپت‌ها را در فایل `.sh` ذخیره کنید و بایدن مجوز اجرا بدهید:
```bash
chmod +x script.sh
./script.sh
```

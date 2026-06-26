# جلسه ۷: اسکریپتنویسی پیشرفته (ساختارهای کنترلی)

## بخش اول: مقدمه و مباحث تئوری

### ۱. مرور سریع جلسه قبل

- **متغیرها**: نحوه تعریف و استفاده از متغیرها در اسکریپت
- **دستور `read`**: دریافت ورودی تعاملی از کاربر
- **تفاوت کوتیشنها**: تفاوت بین Single Quote (`'`) و Double Quote (`"`)
  - Single Quote: هر چیزی را به صورت لیترال می‌گیرد
  - Double Quote: متغیرها را expand می‌کند

- **آرگومانهای خط فرمان**: پاس دادن اطلاعات هنگام اجرای اسکریپت (Preview برای این جلسه)

### ۲. دریافت آرگومان از خط فرمان (Command-line Arguments)

به جای پرسیدن از کاربر به صورت تعاملی، می‌توانیم اطلاعات را مستقیماً هنگام اجرای اسکریپت به آن بدهیم.

#### متغیرهای ویژه:

- **`$0`**: نام خود اسکریپت
- **`$1`, `$2`, `$3`, ...**: اولین آرگومان، دومین آرگومان و الی آخر
- **`$#`**: تعداد کل آرگومانهای پاس داده شده (بدون شمردن نام اسکریپت)
- **`$@`**: همه آرگومانها به صورت یک لیست جداگانه (هر کدام می‌تواند شامل فاصله باشد)
- **`$*`**: همه آرگومانها به صورت یک رشته واحد

#### مثال در ترمینال:

```bash
./myscript.sh ali "Operating System" 10
# $0="./myscript.sh"
# $1="ali"
# $2="Operating System"
# $3="10"
# $#=3
```

#### تفاوت `$@` و `$*`:

```bash
#!/bin/bash
for arg in "$@"; do
    echo "Argument: $arg"
done
# هر آرگومان جداگانه است

for arg in $*; do
    echo "Argument: $arg"
done
# ممکن است رشته‌ها تقسیم شوند
```

### ۳. کدهای خروج (Exit Codes) و متغیر `$?`

هر دستوری که در لینوکس اجرا می‌شود، پس از پایان کار یک **"کد وضعیت"** یا **"کد خروج"** برمی‌گرداند.

#### قوانین جهانی:

- **کد خروج `0`**: یعنی دستور با **موفقیت** انجام شد
- **هر کد خروج غیر صفر** (مثلاً `1`, `2`, `127`): یعنی دستور با **خطا** مواجه شد

#### متغیر `$?`:

این متغیر ویژه، همیشه **کد خروج آخرین دستوری** که اجرا شده را در خود نگه می‌دارد. این ابزار ما برای فهمیدن موفقیت یا شکست یک دستور در اسکریپت است.

#### مثال:

```bash
# این فایل وجود دارد
ls /etc/passwd
echo $?          # خروجی: 0

# این فایل وجود ندارد
ls /etc/nonexistentfile
echo $?          # خروجی: 2 (یا عدد غیر صفر دیگری)
```

#### استفاده در اسکریپت:

```bash
grep -q "^$USERNAME:" /etc/passwd
if [[ $? -eq 0 ]]; then
    echo "User exists"
else
    echo "User not found"
fi
```

### ۴. ساختارهای شرطی: `if`, `else`, `elif`

قلب منطق در برنامه‌نویسی. به ما اجازه می‌دهد بر اساس یک شرط، تصمیم بگیریم که کدام بخش از کد اجرا شود.

#### ساختار پایه:

```bash
if [[ condition ]]; then
    # کدهایی که در صورت درست بودن شرط اجرا می‌شوند
fi
```

#### ساختار کامل‌تر:

```bash
if [[ condition1 ]]; then
    # بلاک اول
elif [[ condition2 ]]; then
    # بلاک دوم
elif [[ condition3 ]]; then
    # بلاک سوم
else
    # بلاک آخر (اگر هیچ‌کدام درست نباشند)
fi
```

#### توضیح دستور `test`:

شرط داخل براکت (`[... ]`) در واقع یک دستور به نام `test` است. اگر شرط درست باشد، این دستور کد خروج `0` برمی‌گرداند و `if` اجرا می‌شود. در bash و shell های مدرن، استفاده از `[[ ... ]]` بسیار بیشتر است زیرا:
- **امن‌تر**: از توسع پذیری غیر ضروری جلوگیری می‌کند
- **انعطاف بیشتر**: از عملگرهای Regex و دیگری پشتیبانی می‌کند

> **نکته Portability**: اگر بخواهیم اسکریپت برای تمام shell ها (`sh`, `dash`, `ksh`) کار کند، باید از `[ ]` استفاده کنیم. اما اگر فقط روی `bash` کار می‌کنیم، `[[ ]]` بهتر است.

### ۵. انواع شرطها (Conditionals)

#### الف) مقایسه رشته‌ها (String Comparison):

```bash
[[ "$str1" = "$str2" ]]      # برابرند؟
[[ "$str1" != "$str2" ]]     # برابر نیستند؟
[[ -z "$str" ]]              # رشته خالی است؟
[[ -n "$str" ]]              # رشته خالی نیست؟
```

#### ب) مقایسه اعداد (Numeric Comparison):

```bash
[[ $num1 -eq $num2 ]]        # برابر (equal)
[[ $num1 -ne $num2 ]]        # نابرابر (not equal)
[[ $num1 -gt $num2 ]]        # بزرگتر (greater than)
[[ $num1 -lt $num2 ]]        # کوچکتر (less than)
[[ $num1 -ge $num2 ]]        # بزرگتر یا برابر (greater than or equal)
[[ $num1 -le $num2 ]]        # کوچکتر یا برابر (less than or equal)
```

#### ج) بررسی فایل‌ها (File Testing):

```bash
[[ -f "/path/to/file" ]]     # آیا یک فایل معمولی است؟
[[ -d "/path/to/dir" ]]      # آیا یک دایرکتوری است؟
[[ -e "/path/to/something" ]]# آیا اصالً وجود دارد (فایل یا دایرکتوری)؟
[[ -r "/path/to/file" ]]     # آیا فایل قابل خواندن است؟
[[ -w "/path/to/file" ]]     # آیا فایل قابل نوشتن است؟
[[ -x "/path/to/file" ]]     # آیا فایل قابل اجرا است؟
```

#### د) ترکیب شرطها (Logical Operators):

```bash
[[ condition1 && condition2 ]]    # AND - هر دو درست باشند
[[ condition1 || condition2 ]]    # OR - حداقل یکی درست باشد
[[ ! condition ]]                 # NOT - منفی کن
```

#### مثال ترکیب:

```bash
if [[ -f "$file" && -r "$file" ]]; then
    echo "فایل وجود دارد و قابل خواندن است"
fi

if [[ -z "$name" || "$name" = "admin" ]]; then
    echo "نام یا خالی است یا admin است"
fi
```

### ۶. حلقه `for` - ساختار تکرار

برای تکرار یک سری دستورات روی لیستی از آیتم‌ها.

#### ساختار پایه:

```bash
for VARIABLE in list_of_items
do
    # کدهایی که برای هر آیتم اجرا می‌شوند
done
```

#### انواع استفاده:

**الف) حلقه روی یک لیست مشخص:**

```bash
for PLANET in Mercury Venus Earth Mars Jupiter
do
    echo "Visiting planet $PLANET..."
done
```

**ب) حلقه روی فایل‌های موجود:**

```bash
for FILE in *.txt
do
    echo "Processing file: $FILE"
done
```

**ج) حلقه روی آرگومان‌های اسکریپت:**

```bash
for arg in "$@"
do
    echo "Argument: $arg"
done
```

**د) حلقه با range (نسخه bash):**

```bash
for i in {1..10}
do
    echo "Number: $i"
done
```

**هـ) حلقه بر روی خروجی یک دستور:**

```bash
for file in $(ls *.txt)
do
    echo "File: $file"
done
```

#### مثال عملی - Backup فایل‌ها:

```bash
#!/bin/bash
# Rename all .txt files to .bak

for FILE in *.txt
do
    echo "Backing up $FILE to ${FILE}.bak"
    cp "$FILE" "${FILE}.bak"
done

echo "Backup complete!"
```

### ۷. اهمیت استفاده از Double Quotes

در اسکریپت‌نویسی بهتر است متغیرها را همیشه در Double Quote قرار دهیم:

```bash
# بد - اگر $FILE شامل فاصله باشد
mv $FILE backup/

# خوب - محفوظ است حتی با فاصله‌ها
mv "$FILE" backup/
```

## بخش دوم: کار عملی با اسکریپت‌های منطقی

### ۱. اسکریپت `arg_check.sh` - بررسی آرگومان‌ها

```bash
#!/bin/bash
# A script to check command-line arguments

echo "This script is called: $0"
echo "You provided $# arguments."
echo "The first argument is: $1"
echo "The second argument is: $2"
echo "All arguments are: $@"
```

تست:
```bash
chmod +x arg_check.sh
./arg_check.sh
./arg_check.sh hello
./arg_check.sh hello world "Operating System"
```

### ۲. اسکریپت بررسی کاربر

```bash
#!/bin/bash
# Check if a user exists

read -p "Enter a username to check: " USERNAME

grep -q "^$USERNAME:" /etc/passwd
# آپشن -q: فقط کد خروج را تنظیم کن، خروجی نده

if [[ $? -eq 0 ]]; then
    echo "User '$USERNAME' exists on this system."
else
    echo "User '$USERNAME' does NOT exist."
fi
```

### ۳. اسکریپت بررسی فایل

```bash
#!/bin/bash
# Check file type from argument

if [[ -f "$1" ]]; then
    echo "$1 is a regular file."
elif [[ -d "$1" ]]; then
    echo "$1 is a directory."
else
    echo "$1 does not exist or is not a regular file/directory."
fi
```

تست:
```bash
chmod +x file_check.sh
./file_check.sh /etc/passwd
./file_check.sh /etc
./file_check.sh nonexistent
```

### ۴. حلقه `for` - مثال سیاره‌ها

```bash
#!/bin/bash
# A simple for loop

for PLANET in Mercury Venus Earth Mars Jupiter
do
    echo "Visiting planet $PLANET..."
done
```

### ۵. حلقه `for` برای پردازش فایل‌ها

```bash
#!/bin/bash
# Rename all .txt files to .bak

for FILE in *.txt
do
    echo "Backing up $FILE to ${FILE}.bak"
    cp "$FILE" "${FILE}.bak"
done
```

برای تست:
```bash
echo "Test 1" > file1.txt
echo "Test 2" > file2.txt
chmod +x backup_script.sh
./backup_script.sh
ls -la    # ببینید .bak فایل‌ها ایجاد شده‌اند
```

## بخش سوم: جمعبندی

### مفاهیم کلیدی این جلسه:

1. **آرگومان‌های خط فرمان**: `$0`, `$1`, `$#`, `$@`, `$*`
2. **کد خروج**: `$?` (0 = موفق، غیرصفر = خطا)
3. **ساختار شرطی**: `if/then/else/elif/fi`
4. **انواع شرطها**: رشته‌ها، اعداد، فایل‌ها
5. **حلقه `for`**: تکرار روی لیست‌ها
6. **اهمیت Double Quotes**: محافظت از فاصله‌ها و کاراکتر‌های ویژه

### تمریناتی برای تقویت:

- نوشتن اسکریپت‌هایی که بیش از یک شرط را بررسی کنند
- استفاده از حلقه‌ها برای پردازش بصری و بهتر
- ترکیب آرگومان‌ها و شرطها برای اسکریپت‌های قوی‌تر

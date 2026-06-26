# جلسه 6 - مقدمه‌ای بر اسکریپت‌نویسی (Bash Scripting)

## تمرین‌های عملی در کلاس و پاسخ‌های کامل

### تمرین اصلی: محاسبه سال باقی‌مانده تا 50 سالگی

اسکریپتی بنویسید که نام و سن کاربر را بپرسد و سپس پیغامی مانند زیر چاپ کند:
```
Hello [Name], you will be 50 years old in [X] years
```

#### پاسخ
```bash
#!/bin/bash
# Script to calculate years until turning 50

read -p "What is your name? " NAME
read -p "How old are you? " AGE

# محاسبه سال‌های باقی‌مانده
YEARS_TO_50=$((50 - AGE))

echo "Hello $NAME, you will be 50 years old in $YEARS_TO_50 years"
```

#### مثال اجرا
```bash
$ chmod +x age_calculator.sh
$ ./age_calculator.sh
What is your name? Ali
How old are you? 25
Hello Ali, you will be 50 years old in 25 years
```

#### توضیح
- `read -p`: دریافت ورودی با یک پیام راهنما (prompt)
- `$((50 - AGE))`: محاسبه‌ی تفاوت بین 50 و سن فعلی
- `echo`: چاپ پیغام نهایی

---

## تمرین‌های هفته آینده و پاسخ‌های تفصیلی

### سوال 1: چرا از `./ ` استفاده می‌کنیم؟

**سوال:** چرا هنگام اجرای یک اسکریپت در دایرکتوری فعلی، باید از `./script_name.sh` استفاده کنیم و نمی‌توانیم فقط `script_name.sh` را بنویسیم؟

#### پاسخ

```bash
# این کار نمی‌شود:
script_name.sh

# این کار می‌شود:
./script_name.sh
```

#### توضیح

**دلیل:** متغیر سیستمی `$PATH`

در جلسه یاد گرفتیم که متغیر `PATH` مسیرهایی را نگهداری می‌کند که شل در آن جستجو می‌کند تا دستورات را پیدا کند.

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

**مشکل:**
- شل فقط در مسیرهای موجود در `$PATH` به دنبال `script_name.sh` جستجو می‌کند
- دایرکتوری فعلی (`.`) معمولاً در `$PATH` موجود نیست (برای دلایل امنیتی)
- بنابراین، اسکریپت در دایرکتوری فعلی پیدا نمی‌شود

**راه‌حل:**
- با استفاده از `./script_name.sh` به شل می‌گوییم: "از دایرکتوری فعلی اجرا کن"
- نقطه (`.`) نماد دایرکتوری فعلی است

**اپشن‌های دیگر:**
```bash
# روش 1: مسیر کامل
/home/user/script_name.sh

# روش 2: از دایرکتوری فعلی
./script_name.sh

# روش 3: اضافه کردن دایرکتوری فعلی به PATH (غیر توصیه‌شده!)
export PATH=".:$PATH"
script_name.sh
```

---

### سوال 2: محاسبه محیط و مساحت مستطیل

**سوال:** اسکریپتی بنویسید که طول و عرض یک مستطیل را از کاربر بگیرد و محیط و مساحت آن را محاسبه و چاپ کند.

#### پاسخ

```bash
#!/bin/bash
# Script to calculate perimeter and area of a rectangle

read -p "Enter the length of the rectangle: " LENGTH
read -p "Enter the width of the rectangle: " WIDTH

# محاسبه محیط: (طول + عرض) * 2
PERIMETER=$(( (LENGTH + WIDTH) * 2 ))

# محاسبه مساحت: طول * عرض
AREA=$(( LENGTH * WIDTH ))

echo "Rectangle dimensions:"
echo "  Length: $LENGTH"
echo "  Width: $WIDTH"
echo ""
echo "Perimeter of the rectangle: $PERIMETER"
echo "Area of the rectangle: $AREA"
```

#### مثال اجرا
```bash
$ chmod +x rectangle.sh
$ ./rectangle.sh
Enter the length of the rectangle: 10
Enter the width of the rectangle: 5

Rectangle dimensions:
  Length: 10
  Width: 5

Perimeter of the rectangle: 30
Area of the rectangle: 50
```

#### توضیح
- **محیط**: (طول + عرض) × 2
- **مساحت**: طول × عرض
- استفاده از `$((عملیات))` برای محاسبات ریاضی

#### نسخه بهتر (با بررسی خطا)
```bash
#!/bin/bash
# Script to calculate perimeter and area of a rectangle (improved)

echo "Rectangle Calculator"
echo "===================="

read -p "Enter the length of the rectangle: " LENGTH
read -p "Enter the width of the rectangle: " WIDTH

# بررسی اینکه مقادیر عددی هستند
if ! [[ $LENGTH =~ ^[0-9]+$ ]] || ! [[ $WIDTH =~ ^[0-9]+$ ]]; then
    echo "Error: Please enter valid numbers!"
    exit 1
fi

# محاسبه
PERIMETER=$(( (LENGTH + WIDTH) * 2 ))
AREA=$(( LENGTH * WIDTH ))

echo ""
echo "Results:"
echo "--------"
echo "Perimeter: $PERIMETER"
echo "Area: $AREA"
```

---

### سوال 3: متغیرهای "$*" و "$@"

**سوال:** متغیرهای `$*` و `$@` در اسکریپت‌نویسی چه تفاوتی با هم دارند؟ (در مورد "آرگومان‌های خط فرمان" تحقیق کنید)

#### پاسخ

**آرگومان‌های خط فرمان:** پارامترهایی هستند که هنگام اجرای اسکریپت به آن ارسال می‌شوند.

```bash
./script.sh argument1 argument2 argument3
```

#### متغیرهای ویژه

```bash
#!/bin/bash
# Script to demonstrate command-line arguments

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Number of arguments: $#"
echo ""
echo "All arguments with \$*: $*"
echo "All arguments with \$@: $@"
```

#### مثال اجرا
```bash
$ chmod +x args.sh
$ ./args.sh apple banana cherry
Script name: ./args.sh
First argument: apple
Second argument: banana
Number of arguments: 3

All arguments with $*: apple banana cherry
All arguments with $@: apple banana cherry
```

#### تفاوت اصلی بین $* و $@

**$\* :**
- تمام آرگومان‌ها را به عنوان یک رشته تک
- در حلقه: یک بار اجرا می‌شود

**$@ :**
- تمام آرگومان‌ها را به عنوان آرایه جداگانه
- در حلقه: برای هر آرگومان یک بار اجرا می‌شود

#### مثال عملی (حلقه)

```bash
#!/bin/bash
# Demonstrating the difference between $* and $@

echo "Using \$*:"
for arg in $*
do
    echo "  - $arg"
done

echo ""
echo "Using \$@:"
for arg in $@
do
    echo "  - $arg"
done
```

**نتیجه:**
```bash
$ ./args.sh apple banana cherry
Using $*:
  - apple
  - banana
  - cherry

Using $@:
  - apple
  - banana
  - cherry
```

#### تفاوت واقعی (در کوتیشن‌ها)

```bash
#!/bin/bash

# $* در دابل کوتیشن: یک رشته
echo "Count of \"\$*\": ${#@}"

# $@ در دابل کوتیشن: هر آرگومان جداگانه
set -- one "two three" four
echo "With \$*: $*"
echo "With \$@: $@"
```

---

## خلاصه مفاهیم مهم

| مفهوم | نمونه | توضیح |
|-------|-------|-------|
| Shebang | `#!/bin/bash` | مشخص می‌کند کدام مفسر استفاده شود |
| کامنت | `# This is a comment` | برای توضیح و توثیق کد |
| متغیر | `NAME="Ali"` | ذخیره داده |
| استفاده | `echo $NAME` | بازیابی مقدار متغیر |
| read | `read -p "Prompt:" VAR` | دریافت ورودی |
| محاسبه | `$((5 + 3))` | عملیات ریاضی |
| آرگومان | `$1, $2, $#` | پارامترهای خط فرمان |

---

## نکات کلیدی

1. **همیشه Shebang اضافه کنید**: بدون آن اسکریپت ممکن است با شل غلطی اجرا شود
2. **مجوز اجرا ضروری است**: `chmod +x script.sh`
3. **دابل کوتیشن برای متغیرها**: `"$VAR"` متغیر را جایگزین می‌کند
4. **سینگل کوتیشن برای تحت‌اللفظی**: `'$VAR'` متغیر را جایگزین نمی‌کند
5. **استفاده از $((...))**:  روش مدرن و پاک برای محاسبات
6. **`./` برای دایرکتوری فعلی**: اگر اسکریپت در PATH نباشد
7. **$0**: نام اسکریپت خود
8. **$#**: تعداد آرگومان‌ها
9. **$***: تمام آرگومان‌ها به عنوان رشته واحد
10. **$@**: تمام آرگومان‌ها به عنوان آرایه جداگانه

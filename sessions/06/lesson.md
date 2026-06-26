# جلسه 6 - مقدمه‌ای بر اسکریپت‌نویسی (Bash Scripting)

## بخش اول: مقدمه و مباحث تئوری

### 1. اسکریپت شل چیست؟

**تعریف:** یک اسکریپت شل، یک فایل متنی است که حاوی مجموعه‌ای از دستورات خط فرمان است.

**هدف:** به جای اینکه دستورات را یکی‌یکی و به صورت دستی در ترمینال تایپ کنیم، آنها را در یک فایل ذخیره می‌کنیم تا شل (مثل Bash) آنها را به ترتیب و به صورت خودکار اجرا کند.

**آنالوژی:** فرض کنید هر روز صبح برای آماده شدن، کارهای مشخصی را به ترتیب انجام می‌دهید. نوشتن یک اسکریپت مانند نوشتن این "لیست کارها" روی کاغذ است. از این به بعد، به جای فکر کردن به تک‌تک مراحل، فقط به "لیست کارها" مراجعه کرده و آن را اجرا می‌کنید.

### 2. چرا از اسکریپت‌نویسی استفاده می‌کنیم؟

- **اتوماسیون**: خودکارسازی کارهای تکراری (مثلاً پشتیبان‌گیری روزانه)
- **سفارشی‌سازی**: ساخت دستورات جدید و شخصی‌شده
- **صرفه‌جویی در زمان و کاهش خطا**: اجرای خودکار دستورات سرعت کار را بالا می‌برد و احتمال خطای انسانی را کاهش می‌دهد

### 3. ساختار یک اسکریپت

#### Shebang
```bash
#!/bin/bash
```
- همیشه اولین خط یک اسکریپت باید این باشد
- به سیستم‌عامل می‌گوید: "این فایل را با استفاده از مفسر bash اجرا کن"
- بدون این خط، اسکریپت با شل اشتباهی اجرا شده و درست کار نکند

#### کامنت‌ها (Comments)
```bash
# این یک کامنت است
```
- هر خطی که با `#` شروع شود، توسط مفسر نادیده گرفته می‌شود
- برای توضیح دادن کد استفاده می‌شود

#### مجوز اجرا
```bash
chmod +x script.sh
```
- یک فایل اسکریپت برای اینکه بتواند مستقیماً اجرا شود، باید مجوز اجرا (x) داشته باشد

### 4. متغیرها (Variables) در Bash

**مفهوم:** متغیرها ظرفهایی برای نگهداری داده‌ها هستند.

#### نحوه تعریف (بدون فاصله در اطراف =)
```bash
VARIABLE_NAME="some value"
NUMBER=42
```

#### نحوه استفاده (با عالمت $)
```bash
echo $VARIABLE_NAME
echo $NUMBER
```

#### تفاوت کوتیشن‌ها

**دابل کوتیشن ("):**
```bash
NAME="Ali"
echo "Hello, $NAME"        # خروجی: Hello, Ali
```
متغیرها را جایگزین (expand) می‌کند.

**سینگل کوتیشن ('):**
```bash
NAME="Ali"
echo 'Hello, $NAME'        # خروجی: Hello, $NAME
```
همه چیز را به صورت تحت‌اللفظی در نظر می‌گیرد و هیچ چیزی را جایگزین نمی‌کند.

## بخش دوم: کار عملی و نوشتن اولین اسکریپت‌ها

### 1. اسکریپت "سلام، دنیا!"

```bash
#!/bin/bash
# My first script
echo "Hello, World!"
```

مراحل:
```bash
touch hello.sh
vim hello.sh
chmod +x hello.sh
./hello.sh
```

### 2. کار با متغیرهای تعریف‌شده توسط کاربر

```bash
#!/bin/bash
# A script to show user information
NAME="Ali"
COURSE="Operating System Lab"
SESSION_NUMBER=6

echo "Welcome to session number $SESSION_NUMBER of the $COURSE course."
echo "Your instructor is $NAME."
```

### 3. کار با متغیرهای سیستمی

```bash
#!/bin/bash
# Displaying useful system variables
echo "Hello, $USER!"
echo "Your home directory is: $HOME"
echo "You are using the $SHELL shell."
echo "The current path is: $PATH"
```

متغیرهای مهم:
- `$USER`: نام کاربری فعلی
- `$HOME`: دایرکتوری خانه کاربر
- `$SHELL`: شل فعلی
- `$PATH`: مسیر جستجو برای دستورات

### 4. دریافت ورودی از کاربر

#### دستور `read`
```bash
#!/bin/bash
# An interactive script
echo "What is your name?"
read USER_NAME
echo "Hello, $USER_NAME! Nice to meet you."
```

#### با `-p` (prompt)
```bash
read -p "What is your favorite programming language? " LANG
echo "$LANG is a great language!"
```

### 5. عملیات ریاضی ساده

#### روش 1: `let`
```bash
let a=5+3
echo $a  # 8
```

#### روش 2: `$((...))` (بهتر و مدرن‌تر)
```bash
sum=$((5 + 3))
diff=$((10 - 2))
product=$((4 * 5))
div=$((20 / 4))

echo $product  # 20
```

#### روش 3: `expr`
```bash
RESULT=$(expr 10 \* 5)  # نکته: برای ضرب از \* استفاده کنید
echo $RESULT  # 50
```

### 6. مثال: ماشین‌حساب ساده

```bash
#!/bin/bash
# Simple calculator
read -p "Enter first number: " NUM1
read -p "Enter second number: " NUM2
let SUM=$NUM1+$NUM2
PRODUCT=$(expr $NUM1 \* $NUM2)
echo "Sum is: $SUM"
echo "Product is: $PRODUCT"
```

## بخش سوم: جمع‌بندی
در این جلسه با این مفاهیم آشنا شدیم:
- ساختار یک اسکریپت (shebang، کامنت‌ها، مجوز اجرا)
- متغیرها و نحوه استفاده از آن‌ها
- متغیرهای سیستمی
- دریافت ورودی با `read`
- عملیات ریاضی
- نوشتن اسکریپت‌های تعاملی

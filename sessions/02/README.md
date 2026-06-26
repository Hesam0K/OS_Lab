# جلسه 2 - مدیریت فایل‌ها و دایرکتوری‌ها

## تمرین‌های عملی و پاسخ‌ها

### تمرین 1
یک ساختار دایرکتوری مانند زیر ایجاد کنید (فقط با یک دستور `mkdir`):

```text
project/
├── assets/
│   ├── css/
│   └── py/
└── pages/
```

#### پاسخ
```bash
mkdir -p project/assets/css project/assets/py project/pages
```

یا راه حل هوشمندانه تر:

```bash
mkdir -p project/assets/{css,py} project/pages
```

### تمرین 2
یک فایل متنی به نام `info.txt` بسازید. برای آن یک hard link به نام `info_h.txt` و یک symbolic link به نام `info_s.txt` ایجاد کنید. سپس فایل `info.txt` را حذف کنید. چه اتفاقی برای دو لینک دیگر می‌افتد؟

#### پاسخ
```bash
touch info.txt
ln info.txt info_h.txt
ln -s info.txt info_s.txt
rm info.txt
cat info_h.txt
cat info_s.txt
```

نتیجه:
- `info_h.txt` همچنان قابل استفاده است و محتویات فایل را نمایش می‌دهد.
- `info_s.txt` شکسته می‌شود و دیگر به فایل اصلی اشاره نمی‌کند.

### تمرین 3
آپشن دستور `rm` که قبل از حذف هر فایل از شما تایید می‌خواهد چیست؟ و آپشنی که این تایید را غیرفعال می‌کند (force) چیست؟

#### پاسخ
برای دیدن این گزینه‌ها می‌توان از دستور زیر استفاده کرد:

```bash
man rm
```

- آپشن `-i`: قبل از حذف هر فایل از کاربر تأیید می‌گیرد.
- آپشن `-f`: حذف اجباری است و تایید را غیرفعال می‌کند.

### تمرین عملی در کلاس
1. در دایرکتوری خانه خود، یک دایرکتوری به نام `lab2` بسازید.
2. داخل `lab2` دو دایرکتوری به نام‌های `backup` و `source` بسازید.
3. وارد دایرکتوری `source` شده و فایلی به نام `app.py` بسازید.
4. یک کپی از `app.py` در دایرکتوری `backup` قرار دهید.
5. نام فایل `app.py` اصلی را به `main.py` تغییر دهید.
6. با استفاده از `vim` داخل فایل `main.py` عبارت `#My First Code` را بنویسید و ذخیره کنید.

#### نمونه پاسخ
```bash
mkdir ~/lab2
mkdir ~/lab2/backup ~/lab2/source
cd ~/lab2/source
touch app.py
cp app.py ../backup/
mv app.py main.py
vim main.py
```

درون فایل `main.py`:
```python
#My First Code
```

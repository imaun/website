---
author: imun
category: projects
cover_image: https://raw.githubusercontent.com/imaun/catdocs/refs/heads/master/assets/catdocs-header.png
created_at: '2025-06-10 20:00:00'
description: CLI tool and .NET library that addresses this by allowing OpenAPI files
  to be split into modular components, validated, and reassembled reliably.
post_type: blog
slug: catdocs
summary: By using Catdocs, each team can manage their part of the API independently—paths,
  schemas, components—while still contributing to a single source of truth that is
  validated and bundled in CI.
tags:
- openapi
- dotnet
- csharp
- automation
- ci-cd
title: 'Catdocs: Managing Modular OpenAPI Specifications at Scale'
updated_at: '2025-06-10 20:00:01'
---

حفظ یک مشخصات OpenAPI در قالب یک فایل بزرگ اغلب به پیچیدگی منجر می‌شود، به‌ویژه در محیط تیم‌های کاری. تداخل نسخه‌ها، مرجع‌های اشتباه جایگذاری‌شده و تعریف‌های حجیم باعث می‌شود همکاری با خطا شود و کند بشود.

**[Catdocs](https://github.com/imaun/catdocs)** ابزاری CLI و کتابخانه‌‌.NET است که به این موضوع رسیدگی می‌کند با اجازه دادن به فایل‌های OpenAPI تا به اجزای ماژولار تقسیم شوند، اعتبار سنجی شوند، و به‌طور قابل اعتماد مجدداً مرتب شوند.

## یک مورد کاربرد عملی: هماهنگی کار API در سراسر تیم‌ها
یک شرکت فینتک را در نظر بگیرید که یک API استراحت دارد که شامل چندین خدمات می‌شود: تأیید هویت، پرداخت‌ها، مدیریت حساب، و غیره. هر خدمت توسط یک تیم جداگانه انجام می‌شود. حفظ یک مشخصات OpenAPI متحد در این تنظیمات، اصطکاک ایجاد می‌کند:

- توسعه‌دهندگان بطور مداوم تغییرات همدیگر را بازنویسی می‌کنند.
- درخواست‌های pull به‌طور مکرر به دلیل مسیرهای `$ref` حل نشده معطل می‌شوند.
- نسخه‌بندی API بدون یک ساختار خسته‌کننده می‌شود.

با استفاده از **Catdocs**، هر تیم می‌تواند بخش خود از API را به طور مستقل مدیریت کند - مسیرها، طرح‌ها، اجزا، در حالی‌که هنوز به یک منبع واحد از حقیقت کمک می‌کند که در CI pipelines اعتبارسنجی و بسته‌بندی شده است.

## بررسی اجمالی: چه چیزی را Catdocs فراهم می‌کند
Catdocs برای موارد زیر طراحی شده است:
- توسعه OpenAPI ماژولار.
- اعتبار سنجی و پردازش مبتنی بر CLI.
- ادغام در سیستم‌های ساخت و workflowهای CI/CD.
- تیم‌هایی که بر بخش‌های متفاوتی از یک API مشترک کار می‌کنند.

### قابلیت‌های اصلی:
- `split`: یک مشخصات بزرگ را به فایل‌های قابل استفاده مجدد تقسیم کنید.
- `bundle`: فایل‌هایی ماژولار را در یک سند OpenAPI معتبر ترکیب کنید.
- `convert`: مشخصات را بین YAML و JSON تبدیل کنید.
- `stats`: سند‌های OpenAPI را تحلیل و اعتبار سنجی کنید.

از هر دو OpenAPI 2 و 3 پشتیبانی می‌کند و با یا `.yaml` یا `.json` کار می‌کند.

---

## شروع به کار

### نصب از طریق Git
```bash
git clone https://github.com/imaun/catdocs.git
cd catdocs
dotnet build
```

### اجرای یک دستور (مثال: دریافت آمار)
```bash
dotnet run -- stats --file path/to/your/openapi.yaml --format yaml
```

### یا استفاده از Docker
```bash
docker pull imaun/catdocs:latest
docker run --rm imaun/catdocs:latest stats --file path/to/your/openapi.yaml
```

---

## بررسی اجمالی دستورات CLI

### ٠. `stats`: تحلیل و اعتبارسنجی
مشخصات را اعتبار سنجی می‌کند و آمار ساختاری را چاپ می‌کند (مسیرها، طرح‌ها، پاسخ‌ها).

```bash
catdocs stats --file example-openapi.yaml --spec-ver 3 --format json
```

---

### ١. `split`: ساختار مشخصه را تجزیه کنید
یک سند OpenAPI متحد را به فایل‌های کوچکتر، قابل مدیریت تجزیه می‌کند.

```bash
catdocs split --file OpenApi.yaml --spec-ver 3 --format yaml --outputDir examples/bundle-pipeline
```

بعد از اجرا، دایرکتوری شامل پوشه‌های جداگانه برای `paths`, `components`, و غیره خواهد بود.

---

### ٢. `bundle`: فایل‌های تجزیه‌شده را ترکیب کنید
فایل‌های تجزیه‌شده را به یک سند OpenAPI تک کرد، حل کننده مراجع.

```bash
catdocs bundle --file examples/bundle-pipeline/OpenApi.yaml \
  --spec-ver 3 --format yaml --output examples/bundle-pipeline/output.yaml
```

---

### ٣. `convert`: JSON ⇄ YAML
فرمت مشخصات OpenAPI را در هنگام اعتبارسنجی ساختار تغییر می‌دهد.

```bash
catdocs convert --file docs.json --spec-ver 3 --format json --output docs.yaml
```

---

## مثال ادغام CI/CD

در یک لوله CI، از `bundle` استفاده کنید تا مطمئن شوید که تمامی اجزاء بعد از هر ارتکاب درست ترکیب می‌شوند:

```yaml
# .github/workflows/openapi.yaml
name: Validate and Bundle OpenAPI

on:
  push:
    branches: [main]

jobs:
  bundle:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run Catdocs
        run: |
          docker run --rm -v ${{ github.workspace }}:/data \
            imaun/catdocs:latest bundle \
            --file /data/examples/bundle-pipeline/OpenApi.yaml \
            --output /data/final-openapi.yaml
```

---

## ادغام .NET

به طور داخلی، Catdocs از بسته رسمی `Microsoft.OpenApi` برای تجزیه و پردازش مشخصات OpenAPI استفاده می‌کند. این باعث سازگاری با استانداردهای OpenAPI که به طور گسترده پذیرفته‌شده‌اند می‌شود و اجازه استفاده مستقیم در پروژه‌های .NET را می‌دهد.

شما می‌توانید از Catdocs به عنوان یک کتابخانه در برنامه‌های .NET خود استفاده کنید تا بر سند‌های OpenAPI را به‌طور برنامه‌ای دستکاری کنید، که از همان داخلی `Microsoft.OpenApi` بهره می‌برد.

```bash
dotnet add package Catdocs.OpenAPI
```

مثال استفاده:
```csharp
using Catdocs.OpenAPI;
// Access and manipulate OpenAPI specs directly
```
---

## کی از Catdocs استفاده نکنیم
در حالی‌که Catdocs مدیریت API ماژولار را خوب حل می‌کند، این به طور کلی مفید نیست:

- اگر مشخصات API شما کوچک است و توسط یک نفر مدیریت می‌شود، تقسیم کردن ممکن است چیزها را پیچیده تر کند.
- این پشتیبانی از پیش‌نمایش‌های بصری مانند Redocly Studio را ندارد.
- تیم‌هایی که با ساختار OpenAPI آشنایی ندارند ممکن است مسیرهای `$ref` اشتباه را آغاز کنند.

---

## خلاصه
Catdocs مجموعه ابزاری را ارائه می‌دهد که کار workflowهای OpenAPI ماژولار را قابل اعتمادتر می‌کند. این پلتفرم‌های API با تمام ویژگی‌ها را جایگزین نمی‌کند، اما فقط به اندازه کافی ساختار و اتوماسیون را برای تیم‌هایی که می‌خواهند انعطاف‌پذیری بدون قفل سازنده را داشته باشند فراهم می‌کند.

مخزن: **[github.com/imaun/catdocs](https://github.com/imaun/catdocs)**

---
[🔗 Source for this blog post](https://github.com/imaun/website/blob/master/data/blog/posts/catdocs.md)
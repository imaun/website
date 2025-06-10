---
author: imun
category: projects
cover_image: https://github.com/imaun/catdocs/blob/master/assets/catdocs-header.png
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

حفظ مشخصات OpenAPI بزرگ به صورت یک فایل واحد اغلب منجر به پیچیدگی می شود، به خصوص در محیط های تیمی. تناقضات متحد الشکل، ارجاعات اشتباه و تعاریف بادرنده همکاری را اشتباه آمیز و کند می سازد.

**[Catdocs](https://github.com/imaun/catdocs)** یک ابزار CLI و کتابخانه .NET است که این مشکل را با اجازه دادن به فایل های OpenAPI برای تقسیم شدن به اجزای ماژولار، اعتبار سنجی شده، و مجدداً به طور قابل اعتماد می گیرد.

## یک مورد کاربرد عملی: هماهنگی کار API در بین تیم ها
یک شرکت فینتک را در نظر بگیرید که دارای API REST است که شامل چندین سرویس است: احراز هویت، پرداخت ها، مدیریت حساب و غیره. هر سرویس توسط یک تیم جداگانه اداره می شود. حفظ یک مشخصات OpenAPI متحد در این راه اندازی باعث می شود:

- توسعه دهندگان به طور دائم تغییرات یکدیگر را بازنویسی می کنند.
- درخواست های کشیده شده به دلیل مسیرهای `$ref` حل نشده غالباً می شکنند.
- نسخه بندی API بدون یک ساختار خسته کننده می شود.

با استفاده از **Catdocs**، هر تیم می تواند قسمت خود را از API را به صورت مستقل مدیریت کند - مسیرها، طرح ها، کامپوننت ها، در حالی که هنوز هم به یک منبع واقعیت که اعتبار سنجی و بسته بندی شده در CI pipelines کمک می کند.

## بررسی کلی: آنچه Catdocs ارائه می دهد
Catdocs برای:
- توسعه  OpenAPI ماژولار.
- اعتبا سنجی و پردازش مبتنی بر CLI.
- ادغام در سیستم های ساخت و CI/CD workflows.
- تیم هایی که روی بخش های مختلف یک API مشترک کار می کنند.

طراحی شده است.

### توانایی های اصلی:
- `split`: شکستن یک مشخصات بزرگ به فایل های قابل استفاده مجدد.
- `bundle`: ترکیب کردن فایل های ماژولار به یک سند OpenAPI معتبر.
- `convert`: تبدیل مشخصات بین YAML و JSON.
- `stats`: تجزیه و تحلیل و اعتبار سنجی سند OpenAPI.

هر دو OpenAPI 2 و 3 را پشتیبانی می کند و با `.yaml` یا `.json`  کار می کند.

---

## شروع کار

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

## بررسی کلی دستورات CLI

### 0. `stats`: تجزیه و تحلیل و اعتبار سنجی
مشخصات را اعتبار سنجی می کند و آمار ساختمانی را چاپ می کند (مسیرها، طرح ها، پاسخ ها).

```bash
catdocs stats --file example-openapi.yaml --spec-ver 3 --format json
```

---

### 1. `split`: شکستن مشخصات
تقسیم یک سند OpenAPI متحد به فایل های کوچکتر، قابل مدیریت.

```bash
catdocs split --file OpenApi.yaml --spec-ver 3 --format yaml --outputDir examples/bundle-pipeline
```

پس از اجرا، دایرکتوری شامل پوشه های جداگانه برای `paths`، `components`، و غیره خواهد بود.

---

### 2. `bundle`: بازترکیب فایل های تقسیم شده
بازچینی فایل‌های تکمیلی شده در یک اسناد OpenAPI واحد، حل مرجع ها.

```bash
catdocs bundle --file examples/bundle-pipeline/OpenApi.yaml \
  --spec-ver 3 --format yaml --output examples/bundle-pipeline/output.yaml
```

---

### 3. `convert`: JSON ⇄ YAML
تبدیل فرمت مشخصات OpenAPI در حالی که ساختار را تأیید می کند.

```bash
catdocs convert --file docs.json --spec-ver 3 --format json --output docs.yaml
```

---

## مثال ادغام CI/CD

در یک خط لوله CI، `bundle` را استفاده کنید تا اطمینان حاصل شود که تمام کامپوننت ها بعد از هر ارسال صحیح ادغام می شوند:

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

Catdocs به طور داخلی از بسته رسمی `Microsoft.OpenApi` برای تجزیه و پردازش مشخصات OpenAPI استفاده می کند. این باعث تطبیق پذیری با استانداردهای OpenAPI به طور گسترده می شود و اجازه استفاده مستقیم در پروژه های .NET را می دهد.

شما می توانید Catdocs را به عنوان یک کتابخانه در برنامه های .NET خود استفاده کنید تا به طور برنامه ای اسناد OpenAPI را دستکاری کنید، با استفاده از همان داخلی `Microsoft.OpenApi`.

```bash
dotnet add package Catdocs.OpenAPI
```

مثال استفاده:
```csharp
using Catdocs.OpenAPI;
// دسترسی و دستکاری مستقیم مشخصات OpenAPI
```
---

## زمانی که از Catdocs استفاده نکنید
در حالی که Catdocs به خوبی مدیریت API ماژولار را حل می کند، اما به طور جهانی سودمند نیست:

- اگر مشخصات API شما کوچک باشد و توسط یک نفر مدیریت می شود، تقسیم ممکن است چیزها را پیچیده کند.
- این پشتیبانی از پیش نمایش های تصویری مانند Redocly Studio را ندارد.
- تیم هایی که با ساختار OpenAPI آشنا نیستند ممکن است مسیرهای `$ref` اشتباه را معرفی کنند.

---

## خلاصه
Catdocs یک مجموعه ابزاری را ارائه می دهد که Workflow های OpenAPI ماژولار را قابل اعتمادتر می کند. این جایگزین کامل برای پلتفرم های API نیست، اما فقط ساختار و اتوماسیون کافی را برای تیم هایی که می خواهند انعطاف پذیری داشته باشند بدون قفل شدن تامین کننده فراهم می کند.

مخزن: **[github.com/imaun/catdocs](https://github.com/imaun/catdocs)**

---
[🔗 متن اصلی برای این پست بلاگ](https://github.com/imaun/website/blob/master/data/blog/posts/catdocs.md)
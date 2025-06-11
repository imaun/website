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

حفظ یک مشخصات OpenAPI در قالب یک فایل بزرگ، اغلب باعث پیچیدگی می‌شود، به‌ویژه در محیط تیم‌های کاری. تداخل نسخه‌ها، مرجع‌های اشتباه قرار داده‌شده و تعاریف حجیم باعث می‌شود تا همکاری با خطا شود و کند بشود.

**[Catdocs](https://github.com/imaun/catdocs)** ابزاری CLI و کتابخانه‌‌.NET است که با این مسئله درگیر شده و اجازه می‌دهد که فایل‌های OpenAPI به اجزای ماژولار تقسیم شوند، تأییدیه گرفته شوند، و به‌طور قابل اعتماد دوباره مرتب شوند.

## یک مورد کاربرد عملی: هماهنگی کار API در سراسر تیم‌ها
یک شرکت فین‌تک را در نظر بگیرید که دارای یک API است که شامل چندین خدمات می‌شود: تأیید هویت، پرداخت‌ها، مدیریت حساب، و غیره. هر خدمت توسط یک تیم جداگانه انجام می‌شود. حفظ یک مشخصات OpenAPI متحد در این تنظیمات، اصطکاک ایجاد می‌کند:

- توسعه‌دهندگان بطور مداوم تغییرات یکدیگر را بازنویسی می‌کنند.
- درخواست‌های pull به‌طور مکرر به دلیل مسیرهای `$ref` حل نشده معطل می‌شوند.
- نسخه‌بندی API بدون یک ساختار محکم خسته کننده می‌شود.

با استفاده از **Catdocs**، هر تیم می‌تواند بخش خود از API را به طور مستقل مدیریت کند - مسیرها، طرح‌ها، اجزا، در حالی که هنوز به یک منبع واحد حقیقت کمک می‌کند که در CI pipelines تأییدیه گرفته و بسته‌بندی شده است.

## بررسی اجمالی : چه چیزی را Catdocs ارائه می‌دهد
Catdocs برای موارد زیر طراحی شده است:
- توسعه OpenAPI ماژولار.
- تأییدیه و پردازش مبتنی بر CLI.
- ادغام در سیستم‌های ساخت و workflowهای CI/CD.
- تیم‌هایی که بر برخی اجزای مختلف یک API مشترک کار می‌کنند.

### ویژگی‌های کلیدی :
- `split`: یک مشخصات بزرگ را به فایل‌های قابل استفاده مجدد تقسیم کنید.
- `bundle`: فایل‌های ماژولار را در یک سند OpenAPI معتبر ترکیب کنید.
- `convert`: مشخصات را بین YAML و JSON تبدیل کنید.
- `stats`: سند‌های OpenAPI را تحلیل و تأییدیه کنید.

هم OpenAPI 2 و هم 3 را پشتیبانی می‌کند و با انتهای فایل  `.yaml` یا `.json` کار می‌کند.

---

## شروع کار

### نصب از طریق Git
```bash
git clone https://github.com/imaun/catdocs.git
cd catdocs
dotnet build
```

### اجرای یک دستور (مثلا دریافت آمار)
```bash
dotnet run -- stats --file path/to/your/openapi.yaml --format yaml
```

### یا استفاده کردن از Docker
```bash
docker pull imaun/catdocs:latest
docker run --rm imaun/catdocs:latest stats --file path/to/your/openapi.yaml
```

---

## بررسی اجمالی دستورات CLI

### ۰. `stats`: تحلیل و تأییدیه
مشخصات را تأییدیه می‌کند و آمار ساختاری را پرینت می‌کند (مسیرها، طرح‌ها، پاسخ‌ها).

```bash
catdocs stats --file example-openapi.yaml --spec-ver 3 --format json
```

---

### ۱. `split`: ساختار مشخصه را بشکنید
یک سند OpenAPI واحد را به فایل‌های کوچکتر، قابل مدیریت می‌شکند.

```bash
catdocs split --file OpenApi.yaml --spec-ver 3 --format yaml --outputDir examples/bundle-pipeline
```

پس از اجرا، مسیری وجود خواهد داشت که شامل پوشه‌های جداگانه برای طرق (`paths`)، کامپوننت‌ها، و غیره می‌باشد.

---

### ۲. `bundle`: فایل‌های شکسته را مرتب کنید
فایل‌های شکسته‌شده را به یک سند OpenAPI واحد تبدیل می‌کند، که حل کننده مراجع است.

```bash
catdocs bundle --file examples/bundle-pipeline/OpenApi.yaml \
  --spec-ver 3 --format yaml --output examples/bundle-pipeline/output.yaml
```

---

### ۳. `convert`: JSON ⇄ YAML
برای سند OpenAPI زمانی که ساختار تأیید شده، فرمت آن را تغییر می‌دهد.

```bash
catdocs convert --file docs.json --spec-ver 3 --format json --output docs.yaml
```

---

## مثال ادغام CI/CD

در یک لوله CI، از `bundle` استفاده کنید تا مطمئن شوید که تمامی اجزاء بعد از هر ارتکاب به درستی مرتب می‌شوند: 

```yaml
#.github/workflows/openapi.yaml
name: Validate and Bundle OpenAPI

on:
  push:
    branches: [ main ]

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

به طور داخلی، Catdocs از بسته رسمی `Microsoft.OpenApi` برای تجزیه و پردازش مشخصات OpenAPI استفاده می‌کند. این باعث می‌شود که با استانداردهای OpenAPI که به طور وسیع پذیرفته‌شده‌اند سازگار باشد و اجازه می‌دهد تا به طور مستقیم در پروژه‌های .NET استفاده شود.

شما می‌توانید از Catdocs به عنوان یک کتابخانه در برنامه‌های .NET خود استفاده کنید تا بتوانید سند‌های OpenAPI را به‌طور برنامه‌ای مدیریت کنید، که از همان خودرو داخلی `Microsoft.OpenApi` بهره می‌برد.

```bash
dotnet add package Catdocs.OpenAPI
```

مثال استفاده :
```csharp
using Catdocs.OpenAPI;
//Dastresi be va dast kari ba maaref OpenAPI be soorate moestagel
```

---

## کی از Catdocs استفاده نکنیم 
در حالی که Catdocs moshkeli dar idare API mozular ra kheili khob hal mikonad, in dar kol be omoom be kaar miad:

- agar maaref API shoma koochak ast va tavasote yek nafar idare mishavad, taghsim shodane an mitavanad chizha ra pechede tar konad.
- in dastegahi az pishnamayash visi mesle Redocly Studio ra nadarad.
- tim haye i ke ba sakhatar OpenAPI ashena nistan, mitavand maasire $ref e ghalat ra shoro konand.

---

## kholase
Catdocs baste abzar i ra pishnahad mikonad ke kar Workflow haye OpenAPI mozular ra ghabele etemad tar mikonad. In platformhaye API ba kolli vizhegi ra jaygozin nemikonad, vali be tanha ye andaze aye ke kafi ast taros va outomation ra baraye tim hay i ke mikhahand flexbilite bi nabze sazandeh ra daran, faramoosh mikonad.

anbar: **[github.com/imaun/catdocs](https://github.com/imaun/catdocs)**

---
[🔗 Manba baraye in post blog](https://github.com/imaun/website/blob/master/data/blog/posts/catdocs.md)
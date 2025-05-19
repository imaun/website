---
author: imun
category: develop
cover_image: https://raw.githubusercontent.com/imaun/website/refs/heads/master/assets/img/hello-world.png
created_at: '2024-10-25 20:00:00'
description: This blog post is a story — a behind-the-scenes peek — into how I built
  this website from scratch
post_type: blog
slug: hello-world
summary: Welcome to my blog! In this post, I will share how I built my website using
  a mix of static & dynamic site generator!
tags:
- asp.net
- blogging
- csharp
- python
- automation
title: How I Built My Website (The Hard/Easy/Fun Way)
updated_at: '2025-05-13 20:00:01'
---

## سلام ! 👋

به گوشه‌ی کوچک من در وب خوش آمدید - و مهم‌تر از همه، به اولین پست وبلاگ من در طی 8 سال! (بله، مدتی شده... به نظر می‌رسد انکار کنیم.)

این پست وبلاگ داستانی است — یک نگاه پشت صحنه — برای چگونگی ساخت این وب‌سایت از ابتدا. بدون وردپرس. بدون هوگو. بدون CMS بیش از حد مهندسی شده. فقط کدها، کنجکاوی، و کمی لجبازی.

**بریم پایین 🚀**

### ✨ ویژن: ساده، قدرتمند، شخصی

همه چیزی که می خواستم بود:

-  وبلاگ تمیز و مینیمال
-  چند صفحه برای به اشتراک گذاشتن کارهای من
-  راهی برای مدیریت راحت محتوا، و بدون CMS
-  میزبانی شده در ازور
-  چند API و ربات تلگرام را در پس زمینه ارائه کنید (شما می دانید، چیزهای مخفی️ 😎)

مطمئناً، می توانستم طی یک بعدازظهر یک سایت وردپرسی بسازم یا یک مولد سایت استاتیک مانند هوگو را استفاده کنم. اما سرگرمی در آن کجاست؟ 
بنابراین خودم آن را ساختم. با C#. با ریزور صفحات. چرا که... چرا که نه؟

### 🏗️ قدم 1: محتوا بدون پایگاه داده

من نمی خواستم از یک پایگاه داده سنتی برای محتوای سایت من استفاده کنم. در نهایت اکثر اوقات متن استاتیکی است - چرا باید SQL به آن اضافه شود؟

به جای آن، من یک مخزن GitHub ایجاد کردم تا همه محتوای وب سایت خود را در آن نگه دارم:

- فایل های HTML برای صفحات منظم
- فایل های Markdown (yaml front-matter) برای پست های وبلاگ
همه میزبانی شده در گیتهاب! [📁 مخزن محتوای وب سایت](https://github.com/imaun/website)

### 🧱 قدم 2: Wdata — یک بارگیر محتوای سفارشی

برای بازیابی محتوا از دیسک یا مستقیم از GitHub، یک کتابخانه C# به نام Wdata ایجاد کردم. بسیار ساده استفاده کردن و در NuGet موجود است:

[🔗 Wdata در NuGet](https://www.nuget.org/packages/Wdata)  
[🔗 مخزن GitHub Wdata](https://github.com/imaun/wdata)  

به شدت قابل پیکربندی است! شما می توانید به پوشه های محلی یا URL های دور دست مانند مسیرهای محتوای خام GitHub اشاره کنید. فقط این را به appsettings خود اضافه کنید:

```xml
WebsiteData": {
    "DefaultSource": "remote",
    "Sources": [
        {
            "Type": "local",
            "BasePath": ""
        },
        {
            "Type": "remote",
            "BasePath": "https://raw.githubusercontent.com/imaun/website/refs/heads/master/"
        }
    ]
}
```

این بدان معنی است که من منبع پیش فرض خود را به حالت دور تنظیم کرده ام، و سپس می توانم با ارسال مسیر نسبی به محتوای من در github، تماس های http را به منبع دور بگیرم. فقط یک مشتری http ساده است، چیزی خاصی نیست!

نمونه استفاده:

```cs
using Wdata;

public class BlogPost : PageModel 
{
    private readonly IWebsiteDataService _dataService;

    public BlogPost(IWebsiteDataService dataService) {
        _dataService = dataService;
    }

    public async Task OnGetAsync()
    {
        LatestPosts = await _dataService.GetPostIndexAsync("remote", "data/blog/index.json");
        HeadingPost = await _dataService.GetWebsitePostAsync("remote", "data/blog/heading.md");
        FeaturedPosts = await _dataService.GetPostIndexAsync("remote", "data/blog/featured.json");
        Post = await _dataService.GetWebsitePostAsync("remote", "data/blog/website-sample-post.md");
    }
}
```

Wdata پشتیبانی از بارگیری فهرست پست‌ها از فایل‌های JSON و بازنمایی پست‌های وبلاگ Markdown به صورت فرد به HTML در زمان اجرا را دارد، بدون نیاز به CMS یا پایگاه داده.

### ☁️ قدم 3: ASP.NET + Razor Pages + Azure
من یک برنامه وب کلاسیک ASP.NET Core با Razor Pages ساختم. Razor Pages سبک، بیانگر، و کنترل کامل را به من می دهد.
من Wdata را برای بازیابی محتوای پویا (مانند پست های وبلاگ و پروژه های پیشنهادی) ادغام کردم، و کل چیز را با استفاده از یک خط لوله GitHub Actions به Azure App Service ارسال کردم.

```yaml
trigger:
  branches:
    include:
    - refs/heads/main

name: $(Build.DefinitionName)-$(SourceBranchName)-$(date:yyyy.MM.dd)-$(rev:r)

jobs:
- job: Phase_1
  displayName: AgentJob
  cancelTimeoutInMinutes: 1
  pool:
    vmImage: ubuntu-latest
  steps:
  - checkout: self

  - task: UseDotNet@2
    displayName: 'Use .NET 8'
    inputs:
      packageType: 'sdk'
      version: '8.0.x'

  - task: DotNetCoreCLI@2
    displayName: dotnet restore
    inputs:
      command: restore
  
  - task: DotNetCoreCLI@2
    displayName: dotnet publish
    inputs:
      command: publish
      publishWebProjects: false
      projects: '**/Imun.Web.csproj'
      arguments: --output "$(Build.ArtifactStagingDirectory)/WebDeploy/" --configuration "Release"

  - task: PublishBuildArtifacts@1
    displayName: 'Publish Artifact: WebDeploy'
    inputs:
      PathToPublish: $(Build.ArtifactStagingDirectory)/WebDeploy/
      ArtifactName: WebDeploy
```

وب سایت من محتوا را مستقیماً از GitHub در زمان اجرا با استفاده از Wdata می کشد، بنابراین نیازی به مجدداً ارسال برای به روزرسانی یک پست وبلاگ وجود ندارد.

### 🌍 قدم 4: چندزبانه کردن - به صورت خودکار

در اینجا جایی است که شادی می شود.

من می خواستم پست های خود را به انگلیسی بنویسم، اما همچنین آنها را به فارسی منتشر کنم. ترجمه همه چیز به صورت دستی؟ خیر.

بنابراین یک GitHub Action ساختم که از API GPT OpenAI برای انجام این کار برای من استفاده می کند. دارد:

- تغییرات فایل در مخزن محتوای GitHub را تشخیص می دهد
- محتوای تغییر کرده را به فارسی ترجمه می کند
- نسخه های ترجمه شده را دوباره در مخزن قرار می دهد

[🔗 عمل ترجمه GPT](https://github.com/imaun/gpt-translate-action)  

خط لوله برای مخزن گیتهاب وبسایت من:
```yaml
name: Translate Files

on:
  push:
    branches:
      - master

jobs:
  translate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3
        with:
          fetch-depth: 2

      - name: Run Translation Action
        uses: imaun/gpt-translate-action@v1.9.0
        with:
          api_key: ${{ secrets.OPENAI_API_KEY }}
          ai_service: "openai"
          ai_model: "gpt-4"
          target_lang: "Persian"
          target_lang_code: "fa"
          file_exts: "md,html,json"
          output_format: "*-{lang}.{ext}"
          base_branch: "master"
```

حالا، هر زمان که یک پست را به روز یا منتشر می کنم، به طور جادویی به فارسی نیز ظاهر می شود. ✨

## 🔮 طرف تاریک (یعنی ویژگی های اضافی)

چیزهای بیشتری در اینجا در حال رخ دادن است که چشم نمی بیند:

- کارگران پس زمینه که ربات های تلگرام من را ارائه می دهند
- API ها برای ابزارهای داخلی که من استفاده می کنم

اما این داستان برای یک پست دیگر است 😉

### 🔚 نگرانی‌های نهایی

وب سایت من هنوز در حال پیشرفت است. اما این زیبایی ساخت آن خودتان است - شما هرگز واقعاً تمام نیستید، و هر ویژگی متعلق به شما برای شکل دادن است.

اگر شما مانند من یک توسعه‌دهنده هستید، نیازی به یک CMS بزرگ برای ساخت یک وبلاگ ندارید. فقط Markdown، یک سرور وب، و کمی کد بسیار می توانند راه بروند. و اگر حس لوکسی دارید، وب سایت خود را آموزش دهید تا در حالی که خوابیدن به چندین زبان صحبت کند 💤

با تشکر برای خواندن - و لطفا در GitHub من بگردید تا بتوانید هر ایده را بدزدید 😄  

---
[🔗 منبع این پست وبلاگ](https://github.com/imaun/website/blob/master/data/blog/posts/hello-world.md)
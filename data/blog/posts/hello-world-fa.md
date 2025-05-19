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

به گوشه کوچک من در وب خوش آمدید - و مهمتر از همه، به اولین پست وبلاگ من در طی 8 سال گذشته! (بله، مدتی شده است... بیایید جعل کنیم که نشده.)

این پست وبلاگ یک داستان است - نگاهی پشت صحنه - برای اینکه من چگونه این وبسایت را از صفر ساختم. هیچ وردپرسی. هیچ هوگویی. هیچ CMS بیش از حد مهندسی شده ای. فقط کد، کنجکاوی، و کمی خودسری.

**بیایید شروع کنیم 🚀**

### ✨ Vision: ساده، قدرتمند، شخصی

تمام آنچه که می خواستم بود:

- یک وبلاگ تمیز، ساده
- چند صفحه برای به اشتراک گذاشتن کارهایم
- روشی برای مدیریت محتوا به راحتی، و بدون CMS
- میزبانی شده در Azure
- در پس زمینه چند APIs و ربات های تلگرام را ارائه دهید (می دانید، چیزهای مخفی 😎)

البته، می توانستم ظهری یک سایت وردپرسی را به هم بفهمم یا از یک مولد سایت ثابت مانند Hugo استفاده کنم. اما سرگرمی کجاست؟
بنابراین من خودم آن را ساختم. در C#. با صفحات رزور. چراکه... چرا نه؟

### 🏗️ گام 1: محتوا بدون پایگاه داده

من نمی خواستم از یک پایگاه داده سنتی برای محتوای سایتم استفاده کنم. اکثر مواقع متن ساده است، تا آخر - چرا SQL را به ترکیب اضافه کنیم؟

به جای آن، من یک مخزن گیت هاب را ساخته ام تا تمام محتوای وب سایت من را ذخیره کند:

- فایل های HTML برای صفحات معمولی
- فایل های Markdown (front-matter yaml) برای پست ها وبلاگ
همه چیز میزبانی شده در گیت هاب! [📁 مخزن محتوای وب سایت](https://github.com/imaun/website)

### 🧱 گام 2: Wdata — بارگذار محتوای سفارشی

برای دریافت محتوا از دیسک یا مستقیماً از گیت هاب، من یک کتابخانه C# به نام Wdata ساختم. استفاده از آن بسیار ساده است و در NuGet در دسترس است:

[🔗 Wdata در NuGet](https://www.nuget.org/packages/Wdata)
[🔗 مخرن گیت هاب Wdata](https://github.com/imaun/wdata)

تنظیمات آن بسیار با دقت است! شما می توانید به پوشه های محلی یا URL های از راه دور مانند مسیرهای محتوای خام گیت هاب اشاره کنید. فقط این موارد را به appsettings خود اضافه کنید:

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

بدین معنی است که من منبع پیش فرض خود را به حالت "remote" تنظیم کرده ام، و سپس می توانم تماس های http را به منبع از راه دور برقرار کنم با گذراندن مسیر نسبی به محتوای من در گیت هاب. فقط یک مشتری http ساده، هیچ چیز خیلی خاص!

یک مثال برای استفاده:

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

Wdata از بارگذاری لیست پست ها از فایل های JSON و تولید پست های وبلاگ Markdown به HTML در زمان اجرا پشتیبانی می کند، نیازی به CMS یا پایگاه داده ندارد.

### ☁️ گام 3: هسته ASP.NET + صفحات رزور + Azure
یک برنامه وب ASP.NET Core کلاسیک با صفحات رزور ساختم. صفحات رزور سبک ، بیانگر ، و تمام کنترل را به من می دهد.
من Wdata را به منظور دریافت محتوای پویا (مانند پست های وبلاگ و پروژه های برجسته) ادغام کردم و تمام این امور را با استفاده از یک خط لوله Actions گیت هاب به خدمات App Azure ارسال کردم. 

``` yaml
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
    displayName: ' استفاده از .NET 8'
    inputs:
      packageType: 'sdk'
      version: '8.0.x'

  - task: DotNetCoreCLI@2
    displayName: بازیابی dotnet
    inputs:
      command: restore
  
  - task: DotNetCoreCLI@2
    displayName: انتشار dotnet
    inputs:
      command: publish
      publishWebProjects: false
      projects: '**/Imun.Web.csproj'
      arguments: --output "$(Build.ArtifactStagingDirectory)/WebDeploy/" --configuration "Release"

  - task: PublishBuildArtifacts@1
    displayName: 'انتشار موجودیت: WebDeploy'
    inputs:
      PathToPublish: $(Build.ArtifactStagingDirectory)/WebDeploy/
      ArtifactName: WebDeploy
```

وب سایت من محتوا را مستقیماً از گیت هاب در زمان اجرا با استفاده از Wdata دریافت می کند، بنابراین نیازی به اعتبار دوباره فقط برای به روزرسانی یک پست وبلاگ نیست.

### 🌍 گام 4: ساخت آن چند زبانه - به صورت خودکار

اینجاست که جالب می شود.

من می خواستم پست هایم را به انگلیسی بنویسم، اما همچنین آنها را به فارسی منتشر کنم. همه چیز را دستی به دست ترجمه کنند؟ نه ممنون.

بنابراین من یک Action گیت هاب ساختم که از API GPT OpenAI استفاده می کند تا برای من این کار را انجام دهد. آن:

- تغییرات فایل را در مخزن محتوای گیت هاب تشخیص می دهد
- محتوای تغییر یافته را به فارسی ترجمه می کند
- نسخه های ترجمه شده را دوباره به مخزن وارد می کند

[🔗 عمل ترجمه GPT](https://github.com/imaun/gpt-translate-action)

خط لوله برای مخزن گیت هاب وب سایت من:
``` yaml
name: ترجمه فایل ها

on:
  push:
    branches:
      - master

jobs:
  translate:
    runs-on: ubuntu-latest

    steps:
      - name:   بررسی مخزن
        uses: actions/checkout@v3
        with:
          fetch-depth: 2

      - name: اجرای عملیات ترجمه
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

پس حالا، هر زمان که من یک پست را به روز کنم یا منتشر کنم، به صورت جادویی هم به زبان فارسی ظاهر می شود. ✨

## 🔮 سمت تاریک (همان قابلیت های اضافی)

بیشتر از آنچه چشم می بیند در اینجا اتفاق می افتد:

- کارگران پس زمینه به خدمت ربات های تلگرام من
- APIs برای ابزارهای داخلی که من استفاده می کنم

اما این قسمت را به پست دیگری می گذرانم 😉

### 🔚 نظرات نهایی

وب سایت من هنوز یک کار در حال پیشرفت است. اما این زیبایی ساختن خود شماست - شما واقعاً برای همیشه تمام نمی شوید، و هر قابلیت به شکل شما است.

اگر شما توسعه دهنده ای مثل من هستید، شما به یک CMS بزرگ برای ساخت وبلاگ نیاز ندارید. فقط Markdown، یک وب سرور، و کمی کد می تواند راه زیادی را برود. و اگر خودتان را حس خاصی به خود ببخشید، به سایت خود بیاموزد تا در حالی که شما خواب هستید چندین زبان صحبت کند 💤

از خواندن متشکرم - و حساب گیت هاب من را به دلخواه بررسی کنید تا هر ایده ای را بدزدید 😄
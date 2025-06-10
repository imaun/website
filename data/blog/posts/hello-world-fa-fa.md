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

به لحظات کوچک من در وب خوش آمدید - و مهم تر از همه، به اولین پست وبلاگ من در طی 8 سال گذشته! (بله، مدتی از آخرین بار گذشته است ... بیایید فرض کنیم اینطور نبوده است.)

این نوشته وبلاگ یک داستان است - یک نگاه پای پرده به پشت صحنه - دریژه ای از چگونگی ساخت این وبسایت از پایه. بدون وردپرس. بدون هوگو. بدون سیستم مدیریت محتوای (CMS) پیچیده بیش از حد. فقط کد نویسی، کنجکاوی ، و یک استقامت درخشان.

** بیایید شروع کنیم 🚀** 

### ✨ چشم انداز: ساده، قدرتمند، شخصی

تمام چیزی که خواستم عبارت بود از :

- یک وبلاگ  تمیز، کم حجم
- چند صفحه برای به اشتراک گذاری کارم
- یك راه برای مدیریت آسان محتوا بدون استفاده از سیستم مدیریت محتوا 
- میزبانی شده بر روی آژور
- ارائه چندین API و بات تلگرام در پس زمینه (شما میدانید، چیزهای مخفی 😎)

مطمئنا ، می توانستم در یک عصر سایت وردپرس را به هم بچسبانم یا از یک تولید کننده سایت استاتیک مانند هوگو استفاده کنم. اما لذت آن کجاست؟
پس خودم آن را ساختم. با سی شارپ. با صفحات رازر. زیرا... چرا که نه؟

### 🏗️ گام اول: محتوا بدون دیتابیس 

من نمی خواستم از یک پایگاه داده سنتی برای محتوای سایت خود استفاده کنم. بالاخره ، اغلب آن متن استاتیک است - چرا SQL را به مجموعه اضافه کنیم؟

به جای آن ، یک مخزن گیت هاب برای ذخیره کل محتوای وب سایتم ایجاد کردم:

- فایل های HTML برای صفحات معمولی
- فایل های Markdown (front-matter yaml) برای پست های وبلاگ
همه آنها بر روی گیت هاب میزبانی می شوند! [📁 مخزن محتوای وب سایت](https://github.com/imaun/website)

### 🧱 گام دوم: Wdata — یک لودر محتوای سفارشی 

برای بازیابی محتوا از دیسک یا مستقیما از گیت هاب ، یک کتابخانه سی شارپ به نام Wdata ایجاد کردم. استفاده از آن بسیار ساده است و در NuGet در دسترس است:

[🔗 Wdata در NuGet](https://www.nuget.org/packages/Wdata)  
[🔗 مخزن گیت هاب Wdata](https://github.com/imaun/wdata)  

پیکربندی آن بسیار آسان است! شما می توانید به پوشه های محلی یا URL های از راه دور مانند مسیرهای محتوای خام گیت هاب اشاره کنید. فقط این را به تنظیمات برنامه خود اضافه کنید:

```xml
"WebsiteData": {
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

این به معنای این است که من منبع پیش فرض خود را به حالت راه دور تنظیم کرده ام و سپس می توانم تماس های http را به منبع راه دور ایجاد کنم با ارسال مسیر نسبی به محتوایم در گیت هاب. این فقط یک مشتری http ساده است، هیچ چیز پیچیده ای در آن وجود ندارد!

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

وی دیتا از بارگیری فهرست پست ها از فایل های JSON و رندرینگ پست های وبلاگ Markdown به شکل فردی به HTML در زمان اجرا پشتیبانی می کند، نیازی به سیستم مدیریت محتوا یا پایگاه داده وجود ندارد.

### ☁️ گام سوم: ASP.NET + صفحات رازور + آژور 

یک برنامه وب کلاسیک ASP.NET Core با صفحات رازور ساختم. صفحات رازور سبک وزن ، خلاقانه هستند و کنترل کامل را به من می دهند.
من Wdata را برای دریافت محتوای پویا (مانند پست های وبلاگ و پروژه های برجسته) یکپارچه سازی و تمام اینها را در سرویس برنامه آژور با استفاده از خط لوله کاری Actions گیت هاب منتشر کردم.

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

وب سایت من محتوا را مستقیما از گیت هاب در زمان اجرا با استفاده از وی دیتا جمع آوری می کند ، بنابراین نیازی به انتشار مجدد فقط برای به روز کردن یک پست وبلاگ وجود ندارد.

### 🌍 گام چهارم: ایجاد چند زبانه خودکار

در اینجا جایی است که کار جذاب می شود.

من می خواستم پست های خود را به زبان انگلیسی بنویسم ، اما آنها را نیز به زبان فارسی منتشر کنم. ترجمه همه چیزها به صورت دستی؟ نه ممنون.

پس یک Action گیت هاب را که از API GPT OpenAI برای انجام این کار به من کمک می کند ساختم. آن:

- تغییرات فایل در مخزن محتوی گیت هاب را تشخیص می دهد
- محتوی تغییر یافته را به فارسی ترجمه می کند
- نسخه های ترجمه شده را دوباره به مخزن ارسال می کند   

[🔗 GPT Translate Action](https://github.com/imaun/gpt-translate-action)  

خط لوله برای مخزن گیت هاب وب سایت من:
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

بنابراین حالا، هر زمان که یک پست را به روز یا منتشر می کنم، آن به صورت جادویی به زبان فارسی نیز ظاهر می شود. ✨ 


### 🔚 فکرهای نهایی 

وب سایت من هنوز کار در حال انجامی است. اما این زیبایی ساختن آن توسط خود شما است - شما هرگز واقعا کار را تمام نمی کنید و هر قابلیت قابل شکل گیری توسط شماست.

اگر شما مانند من توسعه دهنده هستید، برای ساختن یک وبلاگ نیازی به یک سیستم مدیریت محتوای بزرگ ندارید. فقط Markdown، یک سرور وب و کمی کد می تواند راه زیادی ببرد. و اگر دلتان می خواهد خاص بشوید، سایت خود را یاد بدهید تا در حالی که شما در حال خواب هستید به چند زبان صحبت کند💤

از خواندن متشکرم — و احساس راحتی کنید که در گیت هاب من دور بزنید تا ایده هایی از تجربه من بگیرید 😄  

---
[🔗 مرجع برای این پست وبلاگ](https://github.com/imaun/website/blob/master/data/blog/posts/hello-world.md)
---
slug: "hello-world"
title: "How I Built My Website (The Hard/Easy/Fun Way)"
author: imun
post_type: blog
description: "This blog post is a story — a behind-the-scenes peek — into how I built this website from scratch"
summary: "Welcome to my blog! In this post, I will share how I built my website using a mix of static & dynamic site generator!"
category: "projects"
tags:
    - asp.net
    - blogging
    - csharp
    - python
    - automation
cover_image: "https://raw.githubusercontent.com/imaun/website/refs/heads/master/assets/img/hello-world.png"
created_at: "2024-10-25 20:00:00"
updated_at: "2025-05-13 20:00:01"
---

## Hello ! 👋

Welcome to my little corner of the web — and more importantly, to my very first blog post in over 8 years! (Yes, it’s been a while... let’s pretend it hasn’t.)

This blog post is a story — a behind-the-scenes peek — into how I built this website from scratch. No WordPress. No Hugo. No over-engineered CMS. Just code, curiosity, and a sprinkle of stubbornness.

**Let’s dive in 🚀**

### ✨ The Vision: Simple, Powerful, Personal

All I wanted was:

- A clean, minimal blog
- A few pages to share my work
- A way to manage content easily, and without a CMS
- Hosted on Azure
- Serve a few APIs and Telegram bots in the background (you know, the secret stuff 😎)

Sure, I could’ve slapped together a WordPress site in an afternoon or used a static site generator like Hugo. But where’s the fun in that?
So I built it myself. In C#. With Razor Pages. Because… why not?

### 🏗️ Step 1: Content Without a Database

I didn’t want to use a traditional database for my site content. It’s mostly static text, after all — why add SQL into the mix?

Instead, I created a GitHub repository to store all my website content:

- HTML files for regular pages
- Markdown files (yaml front-matter) for blog posts
All hosted on github! [📁 Website Content Repo](https://github.com/imaun/website)

### 🧱 Step 2: Wdata — A Custom Content Loader

To fetch content from disk or directly from GitHub, I created a C# library called Wdata. It’s dead simple to use and available on NuGet:  

[🔗 Wdata on NuGet](https://www.nuget.org/packages/Wdata)  
[🔗 Wdata GitHub Repo](https://github.com/imaun/wdata)  

It’s highly configurable! You can point to local folders or remote URLs like GitHub’s raw content paths. Just add this to your appsettings:

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

It means that I have set my default source to remote, and then I can make http calls to the remote source by passing relative path to my contents on github. It just a simple http client, nothing fancy!

Example usage:

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

Wdata supports loading lists of posts from JSON files and rendering individual Markdown blog posts to HTML at runtime, no CMS or database needed.

### ☁️ Step 3: ASP.NET + Razor Pages + Azure
I built a classic ASP.NET Core web app with Razor Pages. Razor Pages is lightweight, expressive, and gives me full control.
I integrated Wdata to fetch dynamic content (like blog posts and featured projects), and deployed the whole thing to Azure App Service using a GitHub Actions pipeline. 

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

My website pulls content directly from GitHub at runtime using Wdata, so there’s no need to redeploy just to update a blog post.

### 🌍 Step 4: Making It Multilingual — Automatically

Here’s where it gets fun.

I wanted to write my posts in English, but also publish them in Persian (Farsi). Translating everything manually? No thanks.

So I built a GitHub Action that uses OpenAI’s GPT API to do it for me. It:

- Detects file changes in the GitHub content repo
- Translates changed content to Persian
- Pushes the translated versions back into the repo  

[🔗 GPT Translate Action](https://github.com/imaun/gpt-translate-action)  

The pipeline for my website github repo:
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

So now, every time I update or publish a post, it magically appears in Persian too. ✨


### 🔚 Final Thoughts

My website is still a work in progress. But that’s the beauty of building it yourself — you’re never really done, and every feature is yours to shape.

If you’re a developer like me, you don’t need a big CMS to build a blog. Just Markdown, a web server, and a bit of code can go a long way. And if you’re feeling fancy, teach your site to speak multiple languages while you sleep 💤

Thanks for reading — and feel free to poke around my GitHub to steal any ideas 😄  

---
[🔗 Source for this blog post](https://github.com/imaun/website/blob/master/data/blog/posts/hello-world.md)


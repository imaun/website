---
slug: "hello-world"
title: "How I Built My Website (The Hard/Easy/Fun Way)"
author: imun
post_type: blog
description: "This blog post is a story — a behind-the-scenes peek — into how I built this website from scratch"
summary: "Welcome to my blog! In this post, I will share how I built my website using a mix of static & dynamic site generator!"
category: "develop"
Tags:
    - asp.net
    - blogging
    - csharp
    - python
    - automation
cover_image: "https://raw.githubusercontent.com/imaun/website/refs/heads/master/assets/img/hello-world.png"
created_at: "2024-10-25 20:00:00"
updated_at: "2025-05-13 20:00:01"
---

## Hello internet wanderer! 👋

Welcome to my little corner of the web — and more importantly, to my very first blog post in over 8 years! (Yes, it’s been a while... let’s pretend it hasn’t.)

This blog post is a story — a behind-the-scenes peek — into how I built this website from scratch. No WordPress. No Hugo. No over-engineered CMS. Just code, curiosity, and a sprinkle of stubbornness.

**Let’s dive in 🚀**

### ✨ The Vision: Simple, Powerful, Personal

All I wanted was:

- A clean, minimal blog
- A few pages to share my work
- A way to manage content easily
- Hosted on Azure
- Serve a few APIs and Telegram bots in the background (you know, the secret stuff 😎)

Sure, I could’ve slapped together a WordPress site in an afternoon or used a static site generator like Hugo. But where’s the fun in that?
So I built it myself. In C#. With Razor Pages. Because… why not?

### 🏗️ Step 1: Content Without a Database

I didn’t want to use a traditional database for my site content. It’s mostly static text, after all — why add SQL into the mix?

Instead, I created a GitHub repository to store all my website content:

HTML files for regular pages

Markdown files for blog posts

📁 Website Content Repo

### 🧱 Step 2: Enter Wdata — A Custom Content Loader

To fetch content from disk or directly from GitHub, I created a C# library called Wdata. It’s dead simple to use and available on NuGet:

🔗 Wdata on NuGet
🔗 Wdata GitHub Repo

Example usage:

```cs
using Wdata;

var loader = new WebsiteDataService("https://raw.githubusercontent.com/imaun/website/main/");
string markdown = await loader.GetFileContentAsync("data/blog/posts/my-first-post.md");
```

You can load local or remote content — and it's perfect for Markdown-powered blogs.

### ☁️ Step 3: ASP.NET + Razor Pages + Azure

Next, I spun up a classic ASP.NET Core web app with Razor Pages. Razor Pages is simple, fast, and gives me full control. Then I deployed it to Azure Web Apps. Nothing fancy, just good ol’ CI/CD with GitHub Actions and Azure DevOps.

My website pulls content directly from GitHub at runtime using Wdata, so there’s no need to redeploy just to update a blog post.

### 🌍 Step 4: Making It Multilingual — Automatically

Here’s where it gets fun.

I wanted to write my posts in English, but also publish them in Persian (Farsi). Translating everything manually? No thanks.

So I built a GitHub Action that uses OpenAI’s GPT API to do it for me. It:

Detects file changes in the GitHub content repo

Translates changed content to Persian

Pushes the translated versions back into the repo

🔗 GPT Translate Action

Example Workflow:
```yaml
name: Translate Markdown to Persian

on:
  push:
    branches: [ main ]

jobs:
  translate:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run Translation
      uses: imaun/gpt-translate-action@v1
      env:
        OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      with:
        target_lang: fa
```

So now, every time I update or publish a post, it magically appears in Persian too. ✨

## 🔮 The Dark Side (a.k.a. Bonus Features)

There’s more going on here than meets the eye:

Background workers serving my Telegram bots

APIs for internal tools I use

A secret admin dashboard for managing blog metadata (still under construction 🔧)

But that’s a story for another post 😉

### 🔚 Final Thoughts

My website is still a work in progress. But that’s the beauty of building it yourself — you’re never really done, and every feature is yours to shape.

If you’re a developer like me, you don’t need a big CMS to build a blog. Just Markdown, a web server, and a bit of code can go a long way. And if you’re feeling fancy, teach your site to speak multiple languages while you sleep 💤

Thanks for reading — and feel free to poke around my GitHub to steal any ideas 😄

— Iman


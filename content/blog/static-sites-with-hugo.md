+++
author = "Mridul Sahu"
categories = ["Hugo"]
tags = ["tutorial", "aws", "blog"]
date = "2019-05-17"
description = "Generating Static Sites with Hugo"
featured = "hugo_feature.png"
featuredalt = "Hugo - The World's fastest framework for building websites"
featuredpath = "date"
title = "Generating sites with Hugo"
type = "post"
+++

## What is Hugo?

{{< url-link "Hugo" "https://gohugo.io/">}} is a  fast and modern static site generator written in Go. It is a general purpose website-framework which generates
static pages based on your content. Unlike systems that dynamically build a page with each visitor request, Hugo builds pages when you create or update your content.

In technical terms, Hugo takes a source directory of files and templates and uses these as input to create a complete website.

## Who Should Use Hugo? 
1. Hugo is for people who want to hand code their own website without worrying about setting up complicated runtimes, dependencies and databases.

2. Hugo is for people building a blog, a company site, a portfolio site, documentation, a single landing page, or a website with thousands of pages.

In this tutorial, commands that you enter will start with the `$` prompt. The output will follow. Lines that start with `#` are comments that I've added to explain a point.

## Installing
**MacOS**
```
$ brew install hugo
```
**Debian and Ubuntu**
```
$ sudo apt-get install hugo
```

_For other platforms and installation types check this **{{< url-link "link" "https://gohugo.io/getting-started/installing/" >}}**_

**Test Installation**
```
$ hugo help
```
***
# Building Our First Hugo Site:yum:
Let's start a new project and see where can we go from there. The way you start a new hugo project is as simple as running this command (feel free to replace {{<color-text "lets_blog" "#F26D21">}} with what you like).
```
$ hugo new site lets_blog
```
Running the generator from the command line will create a directory structure with the following elements:
```
.
├── archetypes
├── assets
├── config
├── content
├── data
├── layouts
├── static
└── themes
```
_More information about this structure can be found **{{< url-link "here" "https://gohugo.io/getting-started/directory-structure/" >}}**_

## Adding a Theme
Let's add theme to our site. You can find lots of awesome hugo themes **{{< url-link "here" "https://themes.gohugo.io/" >}}**. We will use the **_{{< url-link "Ananke Gohugo Theme" "https://themes.gohugo.io/gohugo-theme-ananke/" >}}_** for the purpose of this tutorial.

1. Initilaze `git` and add a submodule for the theme:

    ```
    $ cd lets_blog
    $ git init
    $ git submodule add git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
    ``` 

2. Specify the theme to use in `config.toml`:

    ```
    $ echo 'theme = "ananke"' >> config.toml
    ```

## Adding some content
Now that we have the basic setup ready, it's time to add some content. Let's add out first post to the site. To add a new post run the following command:
```
$ hugo new posts/my-first-post.md
```
The `hugo new` uses the content-section to find the most suitable archetype template in your project. If your project does not contain any archetype files, it will also look in the theme.

The above will create a new content file in `content/posts/my-first-post.md` using the first archetype file found of these:

1. `archetypes/posts.md`
2. `archetypes/default.md`
3. `themes/my-theme/archetypes/posts.md`
4. `themes/my-theme/archetypes/default.md`

The last two list items are only applicable if you use a theme named {{<color-text "my-theme" "#F26D21">}} (in our case _ananke_).

### What are Archetypes?
Archetypes are content template files in the archetypes directory of your project that contain preconfigured {{< url-link "front matter" "https://gohugo.io/content-management/front-matter/" >}} and possibly also a content disposition for your website’s content types. These will be used when you run `hugo new`.  
_(For more information about Archtypes visit this {{< url-link "page" "https://gohugo.io/content-management/archetypes/" >}})_

### Draft, Future, and Expired Content
Hugo allows you to set draft, publishdate, and even expirydate in your content’s front matter. By default, Hugo will not publish:

1. Content with a future `publishdate` value
2. Content with `draft: true` status
3. Content with a past `expirydate` value

All three can be overridden during both local development and deployment by adding the following flags respectively, or by changing the boolean values assigned to the fields of the same name (without --) in your configuration:

1. `--buildFuture`
2. `--buildDrafts`
3. `--buildExpired`

## Starting the Hugo Server
Let's start the Hugo server with drafts enabled and take a look at our first hugo website:metal:  
Run the following command and goto **{{< url-link "http://localhost:1313/" "http://localhost:1313/">}}**
```
# Publish Drafts too
$ hugo server --buildDrafts
```

## Customizing the site
Let's change a few things in our new site and see Hugo's **{{< url-link "live reload" "https://gohugo.io/getting-started/usage/#livereload" >}}** in action.

### Site Configuration
Open up `config.toml` in a text editor:
```
baseURL = "http://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "ananke"
```
Replace the `title` above with something you like. We'll come back to `baseURL` in a bit as this value is not needed when running the local development server.

### Publishing Our First Post
Open up `/content/posts/my-first-post.md` in a text editor:
```
---
title: "My First Post"
date: 2019-05-17T20:05:14+05:30
draft: true
---
```

Add your post to this file and mark `draft: false`.  
_(This {{< url-link "CheatSheet" "https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet" >}} might be useful while creating the post)_

```
---
title: "My First Post"
date: 2019-05-17T20:05:14+05:30
draft: false
---

East or West, Hugo is the best!
```

## Deploying Our Site:sparkles:
For the pupose of this tutorial we'll be using **Amazon's S3** for hosting our site. You'll need an AWS account: This is the Amazon cloud service which contains the S3 service for file hosting. Their free tier is good for a whole year and after it expires, the S3 cost is almost inexpensive if the traffic of your website doesn’t skyrocket. _(Be advised that a credit card is required to sign up for the service.)_

### Create a bucket
1. Goto **{{< url-link "S3's Home" "https://s3.console.aws.amazon.com/s3/home" >}}** and create a bucket.
2. Choose a bucket name (Ex: _site.my-bucket-name.com_) and keep going next, make sure to not block public access.
    {{< fancybox "date" "aws-hugo-step-1.png" "Create a Bucket" "AWS" >}}
3. Goto Bucket Properties and select **Static Website Hosting**.
    {{< fancybox "date" "aws-hugo-step-2.png" "Static Website Hosting" "AWS" >}}
4. Select **_Use this bucket to host a website_** and fill **index.html** as **_Index Document_**, **404.html** as **_Error Document_** and remember the **Endpoint**.
    {{< fancybox "date" "aws-hugo-step-3.png" "Static Wesite Hosting Settings" "AWS" >}}
5. Under the **_Permissions_** tab, select **_Bucket Policy_** and assign the following policy(_Remember to change the bucket name_):
    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::site.my-bucket-name.com/*"
            }
        ]
    }
    ```

### Configuring Base URL
Open up `config.toml` in a text editor:
```
baseURL = "http://example.org/"
languageCode = "en-us"
title = "My Hugo Site"
theme = "ananke"
```
Replace the value of the `baseURL` with the **AWS Endpoint**.
```
baseURL = "http://site.my-bucket-name.com.s3-website-us-east-1.amazonaws.com"
languageCode = "en-us"
title = "Hugo Site"
theme = "ananke"
```

### Bulding and Uploading
Run `hugo` inside the site directory, this will generate a `public` folder. This folder contains all you site pages, generated by Hugo. Upload all the content of the **public** folder to your **S3 Bucket**. You can now visit your site at the **AWS Endpoint** (Ex:  `http://site.my-bucket-name.com.s3-website-us-east-1.amazonaws.com/`)

## _Congratulations!_ Your Site is now live and kicking:thumbsup:
_(I highly recommend to checkout the {{< url-link "Official Hugo Site" "https://gohugo.io/" >}} to learn more about this amazing stuff or checkout this site's code at {{< url-link "Github" "https://github.com/mridul-sahu/hugo-blog" >}})_
---
title: Adding Netlify CMS to an existing Hugo website
date: 2020-02-20T18:35:23.431Z
image: /images/uploads/netlify_cms.jpg
description: >-
  In this post we are going to see how to add Netlify-CMS to an existing Hugo
  website.
---


This blog runs on Hugo. 

Hugo is a great piece of software, it's insanely fast, has a great documentation, and a great ecosystem.

By great ecosystem I mean that there are plenty of[ open-source theme](https://themes.gohugo.io/) available, an official forum where people a friendly and ready to help. 

One thing that I don't really like about static site generators is that *I find that editing markdown files in my code editor is a bit sad.* 😞

I also like the fact of having some sort of content management system, where I can visually see the different drafts I have: 

![Netlify-CMS editorial workflow](/images/uploads/editorial_workflow.png)

I've been looking at the headless CMS landscape for a long time but I never took the time to actually test one.

I look at the ecosystem, and several CMS seems interesting to me:

* [strapi.io ](https://github.com/strapi/strapi)which is open-source
* [Netlify-cms](https://github.com/netlify/netlify-cms) which is open source and can be hosted directly on Netlify cloud for free
* [Forestry](https://forestry.io/) looks really nice but is commercial 

Since I already new about Netlify and the Github repository has a lot of activity, I decided to try their CMS.

## Prerequisites

* A Netlify account
* A recent version of Hugo (I'm using 0.60)
* 5 minutes ⏰

## Installation

You need to create an `admin` folder inside your `static/` folder with those two files:

```x
admin
 ├ index.html
 └ config.yml
```

Copy this code to the `index.html` :

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>
  <script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>

</head>
<body>
  <!-- Include the script that builds the page and powers Netlify CMS -->
  <script src="https://unpkg.com/netlify-cms@^2.0.0/dist/netlify-cms.js"></script>
</body>
</html>
```

Also add this to your head partial: 

```html
<script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>
```

Now you need to update the `config.yml` file. I use Github and my deploy branch is master.

```yaml
backend:
  name: git-gateway
  branch: master # Branch to update (optional; defaults to master)

# This line should *not* be indented
publish_mode: editorial_workflow

# These lines should *not* be indented
media_folder: "static/images/uploads" # Media files will be stored in the repo under static/images/uploads
public_folder: "/images/uploads" # The src attribute for uploaded media will begin with /images/uploads

collections:
  - name: "blog"
    label: "Blog"
    folder: "content/blog"
    create: true
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - { label: "Date", name: "date", widget: "date" }
      - { label: "Body", name: "body", widget: "markdown" }
      - { label: "Featured Image", name: "image", widget: "image"}
      - { label: "Description", name: "description", widget: "string"}
```

The collections part is the most important one that you will need to adapt. 

Collections represent how our content is structured. Generally you have one or several fields in your Front-matter, in my case title, date, image and description. 

Those fields are assigned a widget type (date will be a datepicker and so on). 

Be careful about the **folder** value, in my case it's "content/blog" because my hugo theme works this way, but the default value for Hugo is "content/posts".

Now commit & push.

## Netlify settings

You need to log into your Netlify account to set up the identity service (to control who will be able to authenticate to your admin area):

* Settings > Identity > **Enable Identity service**
* Registration preferences -> invitation only 
* You can choose to allow external provider like Google authentication or Github
* Services > Git Gateway, and click Enable Git Gateway
* Invite yourself :)

Now if you go to your site/admin/: 

![Netlify CMS authentication](/images/uploads/netlify-cms-auth.png)

And voilà! 

I'm going to test this new setup for a while and I'll update the post with my feedback.

I hope this blog post was helpful 😀 

If you liked it don't hesitate to [follow me on Twitter for updates](https://www.twitter.com/sahinkevin).

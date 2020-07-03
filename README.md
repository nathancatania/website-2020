# Personal Website
[![Netlify Status](https://api.netlify.com/api/v1/badges/1958f9dd-27c6-42b8-b7d9-f11d4dc2f0eb/deploy-status)](https://app.netlify.com/sites/icebreaker-382490/deploys)

My personal website ([nathancatania.com](https://nathancatania.com)) created with Hugo and served with Netlify. The theme is a [modified](https://github.com/nathancatania/website/blob/master/CHANGELOG) version of the [hello-friend-ng](https://github.com/rhazdon/hugo-theme-hello-friend-ng) theme.

## Setup
1. Install [Hugo](https://gohugo.io/getting-started/quick-start/) and verify the install:
```
brew install hugo
hugo version
```

2. Clone the repository:
```
git clone https://github.com/nathancatania/website.git
```

3. Install the hello-friend-ng base theme:
```
cd themes
git submodule add https://github.com/rhazdon/hugo-theme-hello-friend-ng.git themes/hello-friend-ng
```

4. Start the Hugo server with drafts and future-dated posts enabled:
```
hugo server -D -F
```

5. Navigate to [http://localhost:1313/](http://localhost:1313/) to preview the website.

6. Publish and build the static pages when finished:
```
hugo -D
```

## Posts
### Content Organization
Posts are created and writtin as markdown `.md` and stored in the `content/posts` directory.

This isn't great for organization (images for posts need to live elsewhere by default), so you can use [Page Bundles](https://gohugo.io/content-management/page-bundles/) to group post content together.
  * Create a folder with the post title in the `content/posts` directory, eg: `content/posts/my-new-post`.
  * The name of the directory will be what Hugo sets as the permalink to the post!
  * The markdown file is renamed to `index.md` and placed into the `my-new-post` directory.
  * All images related to the post are also stored in the `my-new-post` directory along with the markdown file.
  * This way, you can refer to images directly in the markdown file and they can be previewed correctly while you are writing.

### Front Matter
Each post should start with the following at the top of the markdown documents:
```yaml
---
title: "My New Post"
date: 2020-05-17
description: "This is a description for my brand new post."
cover: banner.png
tags: [azure, lab, notes]
comments: false
toc: true
draft: false
---
```

| Attribute   | Description                                                                                                                                                  |
|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| title       | The title of the post. NOT used for the URL/permalink, but is what shows up on the front page and list views of the website.                                 |
| date        | Date of the post in YYYY-MM-DD format. Time and timezone was removed as part of this theme so are not needed.                                                |
| description | Brief summary of the post. Not used in this theme at the moment.                                                                                             |
| cover       | The image file (if any) to be used as the cover image for the post (the large image at the top of the post).                                                 |
| tags        | Array of tags relevant to the post                                                                                                                           |
| comments    | Not used yet. Need to re-implement Disqus...                                                                                                                 |
| toc         | Whether to enable the Table of Contents at the top of the post. If true, ToC is automatically created using the H1/H2/H3 headings (each nested accordingly). |
| draft       | Whether this post is a draft or not. If true, this post will not be published.                                                                               |


### Creating a new post
Create a new post via the following:
```
hugo new posts/my-new-post/index.md
```
This will create a folder in `content/posts` called `my-new-folder` and the `index.md` file associated with it. Front matter will automatically be created and inserted into the markdown file (although it will differ from the above slightly).

## Images and Static Content
The `static` folder is used for all static files for the website - images, js, css, and so on.

Files in this folder are served from the site root. For example:
* `static/image.png` can be accessed at `/image.png` or `http://mydomain.com/image.png`
* `static/js/main.js` can be accessed at `/js/main.js`, etc.

Any images or files included in a page bundle (folder for a post that contains the markdown file and all related images and files for that post, eg: `content/posts/my-new-post/...`) are served NOT from the site root, but relative to that folder, eg: `![image](image.png)`.

## Customizing
Don't modify the theme directly (under `themes/hello-friend-ng`). Instead, copy the files you want to change to the appropriate directory in the root of the site, and make your changes to the copy. Hugo will use these files first, before looking in the `themes` folder. This method allows you to use an up-to-date version of the base theme, without having to go through and overwrite your changes again.

For example:
* `/themes/<theme-name>/layouts/...` -> `/layouts/...` (anything HTML or relating to structure)
* `/themes/<theme-name>/static/fonts/...` -> `/static/fonts/...` (fonts, or anything static to be served)
* `/themes/<theme-name>/assets/scss` -> `/assets/scss/...` (scss, so essentially any style changes you make)

You may need to manually create the `assets` folder in the site root.

## Deployment
### Initial Deployment
The Hugo team have posted a great guide to initial deployment with Netlify [here](https://gohugo.io/hosting-and-deployment/hosting-on-netlify/).

### Updates
Netlify will build and redeploy the site every time changes are pushed to the git repo; no additional taks required (CD).

## TODO
* Customize archetype to use correct front matter
* Re-add comments (?)
* Favicons
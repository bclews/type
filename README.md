# Type

Minimal and Clean Jekyll Theme.

---

* [Configurations](#configurations)
* [Deployment](#deployment)
* [Posts](#posts)
* [Pages](#pages)
* [Navigation](#navigation)
* [Social Media Links](#social-media-links)
* [Update favicon](#update-favicon)

---

### Configurations

Maxima theme comes with different customizations in the `_config.yml` file:

```sh
title:       Type
email:       ''
description: ''
baseurl:     '' # The subpath of your site, e.g. /blog
url:         '' # The base hostname & protocol for your site
twitter:     ''
github:      ''
instagram:   ''
facebook:    ''

markdown:  kramdown
permalink: pretty
paginate:  60

sass:
  style: compressed

gems:
  - jekyll-paginate
  - jekyll/tagging

include:
  - _pages

exclude:
  - vendor
  - Gemfile
  - Gemfile.lock

# Tags
tag_page_dir:         tag
tag_page_layout:      tag_page
tag_permalink_style:  pretty

# Pages path
defaults:
  - scope:
      path: '_pages'
    values:
      permalink: /:basename:output_ext
```

---

### Deployment

To run the theme locally, navigate to the theme directory and run `bundle install` to install the dependencies, then run `jekyll serve` to start the Jekyll server.

I would recommend checking the [Deployment Methods](https://jekyllrb.com/docs/deployment-methods/) page on Jekyll website.

---

### Posts

To create a new post, you can create a new markdown file inside the `_posts` directory by following the [recommended file structure](https://jekyllrb.com/docs/posts/#creating-post-files).

The following is a post file with different configurations you can add as example:

```sh
---
layout: post
title: Welcome to Jekyll!
featured: true
tags: [frontpage, jekyll, blog]
image: '/images/welcome.jpg'
---
```

You can set the author, featured or not, tags, and the post image.

The `featured` key is to mark the post as a featured post, this will add a simple star icon (☆) to the post card.

To keep things more organized, add post images to **/images/pages** directory, and add page images to **/images/pages** directory.

To create a draft post, create the post file under the **_drafts** directory, and you can find more information at [Working with Drafts](http://jekyllrb.com/docs/drafts/).

For tags, try to not add space between two words, for example, `Ruby on Rails`, could be something like (`ruby-on-rails`, `Ruby_on_Rails`, or `Ruby-on-Rails`).

Note that tags are not working with GitHub Pages, that's because the used [jekyll-tagging
](https://github.com/pattex/jekyll-tagging) plugin is not [whitelisted](https://pages.github.com/versions/) by GitHub.

To make this work, I use [Netlify.com](https://www.netlify.com/) for deployment.

---

### Pages

To create a new page, just create a new markdown file inside the `_pages` directory.

The following is the `about.md` file that you can find as an example included in the theme with the configurations you can set.

```sh
---
layout: page
title: About
image: '/images/pages/about.jpeg'
---
```

Things you can change are: `title` and `image` path.

---

### Navigation

The navigation on the sidebar will automatically include all the links to the pages you have created.

---

### Social Media Links

Social media links included in `_includes/footer.html` file.

The theme is using [Evil Icons](http://evil-icons.io/), which contains very simple and clean icons. The following is a list of the social media icons to use:

Twitter

```html
<span data-icon='ei-sc-twitter' data-size='s'></span>
```

Facebook

```html
<span data-icon='ei-sc-facebook' data-size='s'></span>
```

Instagram

```html
<span data-icon='ei-sc-instagram' data-size='s'></span>
```

Pinterest

```html
<span data-icon='ei-sc-pinterest' data-size='s'></span>
```

Vimeo

```html
<span data-icon='ei-sc-vimeo' data-size='s'></span>
```

Google Plus

```html
<span data-icon='ei-sc-google-plus' data-size='s'></span>
```

SoundCloud

```html
<span data-icon='ei-sc-soundcloud' data-size='s'></span>
```

Tumblr

```html
<span data-icon='ei-sc-tumblr' data-size='s'></span>
```

Youtube

```html
<span data-icon='ei-sc-youtube' data-size='s'></span>
```

---

### Update favicon

You can find the current favicon (favicon.ico) inside the theme root directory, just replace it with your new favicon.

---

👉 Visit [aspirehemes.com](http://aspirethemes.com) for more Jekyll, Ghost, and WordPress themes.
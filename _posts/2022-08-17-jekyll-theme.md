---
title: "How to setup Jekyll theme"
date: 2022-08-17 09:51:19 +0900
permalink: ":categories/programming/:title"
---

## How to setup Jekyll theme

A good website should not be just a plain boring looking UI. It should have a bit of kick to it. Let's see how we can set up Jekyll theme this time.

First, go to `rubygems.org`.

Then, enter `jekyll-theme` in the search box.

Select one of the themes that shows up in the result. For instance, `jekyll-theme-hacker`.

On the page, there is a `homepage` link on the bottom right corner. Click it to see the sample theme:

![DB Schema](/blog/assets/2022-08-17-jekyll-theme/jekyll-theme-hacker.png)
_Figure 1. Jekyll theme detail page_

This will lead you to a github repo. Inside the github repo, click `preview the theme...`:

![DB Schema](/blog/assets/2022-08-17-jekyll-theme/preview.png)
_Figure 1. Jekyll theme github preview_

See if you like the following `jekyll-theme-hacker`:
![DB Schema](/blog/assets/2022-08-17-jekyll-theme/hacker-theme.png)

Once you've found the theme you like, add the following line inside `Gemfile` of your project root folder:

```gemfile
gem jekyll-theme-<name-of-the-theme>
exmaple: gem jekyll-theme-hacker
```

Then replace the theme inside the `_config.yml`:

```yaml
theme: jekyll-theme-hacker
```

Then run `bundle install` to install the added theme.

Then run `bundle exec jekyll serve` to see if the theme applied correctly.

Gotcha: depending on the layout of the page, you have to tweak a bit to make your file compatible to the applied theme. Will dig deeper in a later post.

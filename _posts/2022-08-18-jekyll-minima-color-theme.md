---
title: "How to change Jekyll minima color theme"
date: 2022-08-18 11:00:19 +0900
permalink: ":categories/programming/:title"
---

## How to change Jekyll minima color theme

Changing color theme might be the easiest yet most powerful UI change one can make to a website. A proper color change can increase productivity by so much as we've all witnessed in the case of night/day themes of many websites. Let's see how we can do that in our default Jekyll minima theme.

First you have to update your already added `minima` theme from the `Gemfile`.

1. Go to `Gemfile`
1. Update Gemfile to specify using minima via the github repo:

   ```Gemfile
   gem "minima", git: "https://github.com/jekyll/minima"
   ```

1. Run `bundle install && bundle update` so we can apply all changes.
1. Edit `_config.yml` to apply for the color change:

   ```Yaml
   minima:
     skin: dark
   ```

1. Run `bundle exec jekyll serve` to see changes locally.

Voila!

![minimadark](/blog/assets/2022-08-19-jekyll-minima-color-theme/minima-dark.png)

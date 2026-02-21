---
title: "Adding Custom CSS to Minima v3 on GitHub Pages"
---
At the moment, I'm hosting [the current iteration of my blog]({% post_url 2025-12-29-starting-again %}) on [GitHub Pages](https://pages.github.com/), and I'm using Jekyll's default theme -- [Minima](https://github.com/jekyll/minima) -- for simplicity. Using the method described in this post, I've added some custom CSS to refine Minima's appearance.

## Minima v2 Versus v3

I've made one important change to the default GitHub Pages configuration for Jekyll, which is that I've loaded in Minima v3 via the [jekyll-remote-theme](https://github.com/benbalter/jekyll-remote-theme) plugin with the following line in my site's `_config.yml` file:
```
remote_theme: "jekyll/minima"
```

I've made this change because, at the time of this writing, GitHub Pages still uses Minima v2 by default. This is important, since Minima v3 (which hasn't yet been officially released) includes several non-backwards-compatible changes, including what's discussed in this post.

With that out of the way, let's talk about adding custom CSS to Minima v3.

## Using custom-styles.scss

Minima v3 provides a built-in method for adding your own custom CSS. All you have to do is create a file at `_sass/minima/custom-styles.scss`, relative to the root of your Git repository. Any CSS you add to this file will augment, or override, Minima's default styles.

If you want to add Sass variables and mixins, there's a file for that, too: `_sass/minima/custom-variables.scss`.

Here's an excerpt from the ["customizing templates" section of Minima's README](https://github.com/jekyll/minima?tab=readme-ov-file#customizing-templates) that explains exactly how these override files are meant to be used:
> To have your CSS overrides in sync with upstream changes released in future versions, you can collect all your overrides for the Sass variables and mixins inside a sass file placed at \_sass/minima/custom-variables.scss and all other overrides inside a sass file placed at path \_sass/minima/custom-styles.scss.

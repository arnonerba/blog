---
title: "Adding Custom CSS to Minima v3 on GitHub Pages"
---
At the moment, I'm hosting [the current iteration of my blog]({% post_url 2025-12-29-starting-again %}) on [GitHub Pages](https://pages.github.com/), and I'm using Jekyll's default [Minima](https://github.com/jekyll/minima) theme for simplicity. Using the method described in this post, I've added some custom CSS to refine Minima's appearance.

## Using custom-styles.scss

Minima v3 provides a built-in method for adding your own custom CSS. All you have to do is create a file at `_sass/minima/custom-styles.scss`, relative to the root of your Git repository. Any CSS you add to this file will augment -- or override -- Minima's default styles.

If you want to add Sass variables and mixins, there's a file for that, too: `_sass/minima/custom-variables.scss`.

Here's an excerpt from the ["customizing templates" section of Minima's README](https://github.com/jekyll/minima?tab=readme-ov-file#customizing-templates) that explains exactly how these override files are meant to be used:
> To have your CSS overrides [remain] in sync with upstream changes released in future versions, you can collect all your overrides for the Sass variables and mixins inside a sass file placed at `_sass/minima/custom-variables.scss` and all other overrides inside a sass file placed at path `_sass/minima/custom-styles.scss`.

## Bonus Section: Minima v2 Versus v3

At the time of this writing, GitHub Pages still uses Minima v2 by default. This is important, since Minima v3 (which hasn't yet been officially released) includes several non-backwards-compatible changes, including what's discussed in this post. Adding custom CSS to Minima v2 is totally different.

If you want to use Minima v3 on GitHub Pages -- like I've done on this site -- you have to load the theme in via the [jekyll-remote-theme](https://github.com/benbalter/jekyll-remote-theme) plugin. This can be accomplished with a single line in your site's `_config.yml` file:
```
remote_theme: "jekyll/minima"
```

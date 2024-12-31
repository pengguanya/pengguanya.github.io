---
title: "Integrating LaTeX in GitHub Pages with MathJax and KaTeX"
date: 2024-08-21
author: peng
categories: [Blogging, Tech]
tags: [jekyll, github-pages, latex, mathjax, katex]
image:
  path: assets/headers/2024-09-06-RL-tabular-q-learning.webp
  alt: LaTeX Rendering in GitHub Pages
---

## Introduction

This post provides a guide on how to integrate LaTeX rendering into your GitHub Pages site using two popular libraries: MathJax and KaTeX.

## MathJax Integration

MathJax is a JavaScript library that allows for the display of mathematical notation in web browsers. It supports a wide range of LaTeX commands.

### Setup

1. **Include MathJax Script**: Add the following script to your site's HTML header.

```html
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
```

If you use theme such as `jekyll-theme-chirpy` copy from source repo `_includes/head.html` and paste the two lines there at proper position.
A more orgnized way is to create a new file `_includes/mathjax.html` and paste the two lines there, then include this file in `_includes/head.html` at proper position.Same for the KaTeX setup below. 


2. **Use LaTeX in Markdown**: Enclose your LaTeX expressions within `$$` for block equations or `$` for inline equations.

```markdown
Here is an inline equation: $E = mc^2$.

And here is a displayed equation:

$$
x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$
```
MathJax can be very slow in rendering large documents with many equations. In such cases, consider using KaTex.

## KaTeX Integration

KaTeX provides faster rendering of LaTeX on the web. It works by converting LaTeX into HTML and CSS at build time.

### Setup

1. **Install Necessary Gems**: Add these lines to your Gemfile and run `bundle install`.

```ruby
gem "kramdown-math-katex"
gem "execjs"
gem "therubyracer"
```

2. **Configure Jekyll**: Add the following to your `_config.yml`.

```yaml
kramdown:
  math_engine: katex
```

## KaTeX Integration

KaTeX is a fast, easy-to-use library for rendering LaTeX in web pages. Below are two methods to integrate KaTeX into your Jekyll site hosted on GitHub Pages.

### Method 1: Using the Auto-Render Extension

This method uses KaTeX's auto-render extension to automatically find and render LaTeX within the text. It's suitable for GitHub Pages as it works purely on the client side.

1. **Include KaTeX CSS and JS**: Add the following lines to your site's HTML header in `_includes/head.html`:

```html
<head>
  ...
  <!-- KaTeX -->
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.11/dist/katex.min.css" integrity="sha384-nB0miv6/jRmo5UMMR1wu3Gz6NLsoTkbqJghGIsx//Rlm+ZU03BU6SQNC66uf4l5+" crossorigin="anonymous">
  <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.11/dist/katex.min.js" integrity="sha384-7zkQWkzuo3B5mTepMUcHkMB5jZaolc2xDwL6VFqjFALcbeS9Ggm/Yr2r3Dy4lfFg" crossorigin="anonymous"></script>
  <script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.11/dist/contrib/auto-render.min.js" integrity="sha384-43gviWU0YVjaDtb/GhzOouOXtZMP/7XUzwPTstBeZFe/+rCMvRwr4yROQP43s0Xk" crossorigin="anonymous"></script>
  <script>
      document.addEventListener("DOMContentLoaded", function() {
          renderMathInElement(document.body, {
              delimiters: [
                  {left: '$$', right: '$$', display: true},
                  {left: '$', right: '$', display: false},
                  {left: '\\(', right: '\\)', display: false},
                  {left: '\\[', right: '\\]', display: true}
              ],
              throwOnError: false
          });
      });
  </script>
</head>
```

**Options Explained**:
- `delimiters`: Specifies which delimiters should be recognized as math expressions in the text. This allows you to use LaTeX style delimiters like `$$...$$` for display math and `$...$` for inline math.
- `throwOnError`: If set to `false`, KaTeX will render errors in the console instead of breaking the rendering process. This helps in ensuring that the rest of the math on the page renders even if one expression has issues.

### Method 2: Using kramdown with KaTeX

This method involves using Jekyll's built-in support for kramdown with the KaTeX engine for server-side rendering of LaTeX.

2. **Include KaTeX CSS**: Since the HTML is pre-rendered, you only need to include the CSS for styling. Add this to your `_includes/head.html`:

Downloaded the [latest release](https://github.com/KaTeX/KaTeX/releases) of KaTeX on GitHub. Create `/assets/plugins/katex.<version>/` folder in the Jekyll site and copy the contents of the folder of `fonts/` and `katex.min.css` from the downloaded release to this folder.

Add the following lines to the site's HTML header in `_includes/head.html`:

```html
<head>

  ...

  <!-- KaTeX -->
  <link rel="stylesheet" href="/assets/plugins/katex.<version>/katex.min.css">

</head>
```

As explained for MathJax above, you can create a new file `_includes/katex.html` and paste the above lines there, then include this file in `_includes/head.html` at proper position.


3. **Use LaTeX in Markdown**: Similar to MathJax, use `$$` for block equations and `$` for inline equations.

```markdown
Here is an inline equation: $E = mc^2$.

And here is a displayed equation:

$$
x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$
```

## Toggling Between KaTeX and MathJax in Jekyll Sites

KaTeX and MathJax can be easily toggled through the markdown front matter in Jekyll sites. This example uses the Chirpy theme, but the approach can be adapted to other themes by modifying file names and folder structures.

### Setup in Theme Files

1. **Create Include Files**: In your `_includes` directory, create files named `katex_head.html` and `mathjax_head.html`. These should contain the necessary scripts and styles for KaTeX and MathJax respectively.

2. **Modify the Layout File**: Depending on your theme, locate the main layout file (e.g., `default.html` in Chirpy or `base.html` in recent versions of Minima). Include the following logic before the `<body>` tag to toggle between KaTeX and MathJax based on page-specific front matter:

```html
...
{% raw %}
{% include head.html %}

{% if page.katex %}
  {% include katex_head.html %}
{% elsif page.mathjax %}
  {% include mathjax_head.html %}
{% endif %}
{% endraw %}
...
```

This setup checks if `katex` or `mathjax` is enabled in the front matter of a post and includes the corresponding head file.

### Enabling in Posts

Specify in the front matter of your posts whether to use KaTeX or MathJax. Here's how you set it up in your post's Markdown file:

For KaTeX:

```markdown
---
title: "Example Post with KaTeX"
date: YYYY-MM-DD
categories: tutorial
katex: True
---
```

For MathJax:

```markdown
---
title: "Example Post with MathJax"
date: YYYY-MM-DD
categories: tutorial
mathjax: True
---
```

## References

For further reading and additional resources, refer to the following links:

- [KaTeX Auto-render Documentation](https://katex.org/docs/autorender)
- [Converting Formulas to HTML with Kramdown](https://gendignoux.com/blog/2020/05/23/katex.html#converting-formulas-to-html-with-kramdown)
- [How to LaTeX in Jekyll using KaTeX](https://www.xuningyang.com/blog/2021-01-11-katex-with-jekyll)

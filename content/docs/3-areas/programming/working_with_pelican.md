---
date: '2020-01-09'
title: 'Working with Pelican: Assorted Notes'
---

# Working with Pelican: Assorted Notes
## Using Pandoc to Convert Formats


Note that the `-s` specifies to create a standalone document. The
general structure:

```shell
pandoc inputfile.ext -f fromtype -t totype -s -o outputfile.ext
```

When converting from Markdown (specifically Kramdown, when I was editing
files using VS Code before the switch to Emacs) to reStructuredText, I
was getting some weird character rendering. Is there a neat
Unicode-related argument that I can pass to Pandoc to sort this out?

Okay, the Pandoc docs say that it works with UTF-8, and the lower-right
hand corner of my VS Code window says that my Markdown file is encoded
with UTF-8. So that's not it.

## Rendering Mathematics with Pelican

To neatly render the mathematics inside my Pelican posts, I use the
Pelican plugin
[render\_math](https://github.com/getpelican/pelican-plugins/tree/master/render_math).
Since I don't want to clone / download the entire set of Pelican
plugins, I used [DownGit](https://minhaskamal.github.io/DownGit/#/home)
to grab `render_math` and place it in
`mostly-conjecture/plugins/render_math`. Then I modify my
`pelicanconf.py` file:

```python
PLUGIN_PATHS = ['plugins']
PLUGINS = ['render_math']
```

Boom! Math is rendering nicely.

## Pelican, reStructuredText and Centering Images


In our minimalist stylesheet, we are not currently providing support for
the image tag `:align: center`, i.e., if we place an image in an .rst
file,

``` {.sourceCode .rst}
.. image:: images/webpage_build_02.jpg
   :width: 600px
   :alt: Minimalist victory trumpets please.
   :align: center
```

the centering will be ignored, because the correct class, in this case,
`align-center`, is not defined in the stylesheet. This apparently is a
problem with the default Penguin themes as well, see [this
issue](https://github.com/getpelican/pelican/issues/776). The solution
provided there is what I\'ve implemented; I\'ve added the following to
my `style.css`:

```css
.align-center {
   display: block;
   margin-left: auto;
   margin-right: auto;
}
```

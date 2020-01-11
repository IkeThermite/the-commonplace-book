---
date: '2020-01-08'
title: A Minimal Webpage with Pelican
---

# A Minimal Webpage with Pelican

## Getting Started

I already had Anaconda Python installed, so the first thing to do was to
create a `conda` environment, which I called `pelicansite`,:

```shell
conda create --name pelicansite
```

and then activate it: :

```shell
conda activate pelicansite
```

This is all done from the Anaconda Prompt so that the correct
environment variables are set. Now I can install the `pelican` package
(I don't need the optional argument for using markdown). :

```cmd
conda install pelican
```

Then I create a folder for my webpage and navigate to it. :

```cmd
mkdir mostly-conjecture
cd conjecture
```

Finally, I can run `pelican-quickstart`. My answers to the various
prompts are provided below.

```shell
Welcome to pelican-quickstart v4.2.0.

This script will help you create a new Pelican-based website.

Please answer the following questions so this script can generate the files
needed by Pelican.


> Where do you want to create your new web site? [.]
> What will be the title of this web site? Mostly Conjecture
> Who will be the author of this web site? Ralph Rudd
> What will be the default language of this web site? [English]
> Do you want to specify a URL prefix? e.g., https://example.com   (Y/n) n
> Do you want to enable article pagination? (Y/n) n
> What is your time zone? [Europe/Paris] EST
> Do you want to generate a tasks.py/Makefile to automate generation and publishing? (Y/n) y
> Do you want to upload your website using FTP? (y/N) n
> Do you want to upload your website using SSH? (y/N) n
> Do you want to upload your website using Dropbox? (y/N) n
> Do you want to upload your website using S3? (y/N) n
> Do you want to upload your website using Rackspace Cloud Files? (y/N) n
> Do you want to upload your website using GitHub Pages? (y/N) y
> Is this your personal page (username.github.io)? (y/N) y
```

This created the following set of files: :

```shell
<DIR> content
<DIR> output
Makefile
pelicanconf.py
publishconf.py
tasks.py
```

At this point I'm quite worried that I'm going to need to edit the
`Makefile` before this is all over. It's been about 8 years since I've
been in Make-hell, but I don't miss it in the least. Okay, before I can
generate the site for the first time, I need some content.

Pelican will categorize the content based on the folder structure,
unless I disable this. Since I intend to use the P.A.R.A. organizational
system, that suits me just fine. I create the following file: :

```shell
/content/areas/online-portfolio/creating_this_webpage.rst
```

I fill it with the bare minimum necessary for it to build. For reasons
unknown, it fails unless I include the date. So at this stage,
`creating_this_webpage.rst` looks like this:

```rst
Creating this Webpage with Pelican
==================================
:date: 2020-01-08
```

Now I navigate to the root directory, `mostly-conjecture`, and build my
webpage for the first time. :

```shell
pelican content
```

This gives me the following warning: :

```cmd
WARNING: Docutils has no localization for 'english'. Using 'en' instead.
```

This bothers me because the default for the `pelican-quickstart` prompt
was `english`. I shudder to think I may have to edit the `Makefile`, but
no, this is a variable set inside the `pelicanconf.py` file as
`DEFAULT_LANG`. I change it from `"English"` to `"En"` and re-run
`pelican content`. This time it runs without warnings or errors. :

    Done: Processed 1 article, 0 drafts, 0 pages, 0 hidden pages and 0 draft pages in 3.69 seconds.

In a separate Anaconda prompt I navigate to my site's directory and
run:

```cmd
pelican --listen
```

I fire up Chrome and navigate to `localhost:8000`. Trumpets of victory!
The first build of the webpage looks like this:

![Victory trumpets please.](/img/webpage_build_01.jpg#center)

## Modifying the Quickstart Page

However, when I check my second Anaconda prompt (the one
''listening''), I see that I've picked up a warning:
```cmd
WARNING: Unable to find `/favicon.ico` or variations:
```
It goes on to list some variations of the file it's looking for. I
don't know what a `favicon` is, but
[Wikipedia](https://en.wikipedia.org/wiki/Favicon) is to the rescue.
After a quick google (and it might have been a Quorum answer), I head to
[favicongenerator.net](https://realfavicongenerator.net/). I modify my
Steam profile icon to be the recommended 260 by 260 pixels and
[favicongenerator.net](https://realfavicongenerator.net/) generates a
host of different size and types of `favicons` for me. But where exactly
do I put it?

Luckily, someone on StackOverflow has already asked [\"How to add a
favicon to a pelican
blog?\"](https://stackoverflow.com/questions/31270373/how-to-add-a-favicon-to-a-pelican-blog).
So I place my `favicon.ico` inside the newly created
`mostly-conjecture\content\extras` and add the following lines to my
`pelicanconf.py`:

```python
STATIC_PATHS = [
    'extra',
    ]

EXTRA_PATH_METADATA = {
    'extra/favicon.ico': {'path': 'favicon.ico'},
    }
```

I rerun `pelican content` and `pelican --listen` (on the other prompt)
and BOOM! The warning has disappeared.

There's a lot happening on the default page and I want to start by
stripping some of that away. By commenting out the `LINKS` and `SOCIAL`
variables inside my `pelicanconf.py` the page already shrinks quite a
bit. But I don't want to work with the `notmyidea` theme; as there is
too much going on. Instead, I follow [this guide from Matt
Makai](https://www.fullstackpython.com/blog/generating-static-websites-pelican-jinja2-markdown.html)
(the \"Modify Site Theme\" section) as well as the [Pelican
documentation](https://docs.getpelican.com/en/4.0.1/themes.html) (see
the Example at the end) and create my own, very basic theme.

First, I create the theme directory. I'm calling my own theme
`notsosimple`. From my site's root directory:
```cmd
mkdir themes/notsosimple/templates
mkdir themes/notsosimple/static/css
```
I create a `templates/base.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>{% block title %}{% endblock %}</title>
  <link rel="stylesheet" type="text/css" href="{{ SITEURL }}/theme/css/style.css" />
</head>
<body>
 <div class="container">
  {% block content %}{% endblock %}
 </div> 
</body>
</html>
```

As well as a `static/css/style.css`:

```css
body {
  max-width: 70ch;
  padding: 2ch;
  margin: auto;
}
```

The code for `base.html` is different from Matt's guide, as instead of
using the
[Bootstrap](https://getbootstrap.com/docs/4.3/getting-started/introduction/)
framework, I'm linking to my custom stylesheet. The stylesheet is from
[58 bytes of css to look great nearly
everywhere](https://jrl.ninja/etc/1/).

Now the page looks like this:

![Minimalist victory trumpets please.](/img/webpage_build_02.jpg#center)

And that's it. About as minimal as it's going to get.

To-do:
======

-   Replace my favicon with something which is not a beloved and
    trademarked character
-   Create a StackOverflow account to upvote the answers which helped
    create this page
-   Contact Matt about `article.locale_date`.

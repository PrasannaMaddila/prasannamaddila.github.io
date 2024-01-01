---
layout: post
title: "Math for GitHub pages using MathJax"
categories: code
---

Here, I just want to illustrate my setup for rendering math on my GitHub Pages blog. To enable this, we will first enable [MathJax](https://www.mathjax.org/) rendering using [kramdown](https://kramdown.gettalong.org/index.html), and configure how MathJax is triggered. 

## Changing the MD Rendering Engine

Firstly, we need to specify that Markdown has to be rendered with `kramdown`. This is because I didn't have too much luck with the default engine used by GitHub. This can be done quite simply by adding the following line in the `_config.yml` file (usually in the root directory)

```yml
markdown: kramdown
```

Make sure this is visible on the top level of the YAML file. `kramdown` is only used to render the Markdown files, so this shouldn't have any noticeable changes for now. 

> __Note__: Ideally, this step is obsolete as of late 2023. Refer to [GitHub Docs](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll) for more information.


## MathJax Header

Now, we need to enable and configure MathJax support. This can be done by adding a few lines to the `head` of each post when it is rendered as HTML content. Since we write in Markdown, and the HTML processing is done after we're finished writing, this may be non-obvious.

However, there is a way! A GitHub theme usually has a HTML document called a __layout__, which it uses to render each post. Consider the example from [Minima](https://github.com/jekyll/minima/blob/master/_layouts/base.html). Now, `base.html` includes by default a HTML head element defined in `_includes/head.html`. The coup de grâce is that this header element allows the user to define custom content in the `_includes/custom-head.html` file ! Note that all this is specific to the [Minima](https://github.com/jekyll/minima) theme that I use - your mileage may vary.

**TL;DR**, we need to define a file `custom-head.html` in the `_includes/` directory. The contents of my custom header are exactly this:
```html
<script type="text/x-mathjax-config"> 
    MathJax.Hub.Config({ 
        TeX: { equationNumbers: { autoNumber: "all" } } 
    }); 
</script>
<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
        tex2jax: {
         inlineMath: [ ['$','$'], ["\\(","\\)"] ],
         processEscapes: true
        }
    });
</script>
<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript">
</script>
```

This code achieves a few things. First, it loads in MathJax to render our math equations. Secondly, it also makes it possible to define inline math equations using the single `$` - the double `$$` is the default. Therefore, we can write this 

```markdown
$f : \mathbb{R} \rightarrow \mathbb{R}$
```

to get the following code: $f : \mathbb{R} \rightarrow \mathbb{R}$. Lastly, it also enables equation numbering. 

With this, we have our inline math support. However, to support multi-line equations, we can simply write 

```markdown
$$
\begin{align}
f(x) ( g(x) + h(x) ) &= f(x)g(x) + f(x)h(x) \\
( f(x) + g(x) ) h(x) ) &= f(x)h(x) + g(x)h(x) \\
\end{align}
$$
```

to get 

$$
\begin{align}
f(x) ( g(x) + h(x) ) &= f(x)g(x) + f(x)h(x) \\
( f(x) + g(x) ) h(x) ) &= f(x)h(x) + g(x)h(x) \\
\end{align}
$$

## Conclusions 

A few simple steps, and we're well on our way to writing math in our blogs! A future post will deal with numbered theorem environments, which I currently use in my setup for this blog.

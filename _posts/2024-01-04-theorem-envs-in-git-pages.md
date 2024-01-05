---
layout: post
title: "General Numbered Theorem Environments for GitHub Pages"
categories: code
---

This post is about getting LaTeX-style theorem-type environments for my GitHub Pages blog. Now, I'm not sure there's a way to do this with MathJax, so I decided to go the HTML/CSS route. By avoiding the TeX rendering, we make it useable for those who don't use it, so win :) In full disclosure, this approach has been inspired heavily by [felix11h](https://felix11h.github.io/blog/mathjax-theorems)'s blog post on this, so feel free to check that one out ! 

## Plan

The plan is to create a CSS class for each type of environment that we want to create. For example, we'd love classes for Theorems, Definitions, Propositions, and so on. The catch is that most of these environments share a lot of features such as italicised text, and list support. So, we can define a base environment, and then use SCSS to specialise it to all the environments we actually want.

We know that all the theorem-style environments must 

- support Lists
- be numbered
- be in italics
- have a customisable prefix; it must indicate what type of environment it is (for example, "Theorem 1.") and optionally include some indication on the contents, such as 

<div class="thm" text="Great Theorem">
This is a great theorem.
</div>

## SCSS and the Minima theme

Now, GitHub Pages allows us to provide SCSS schema which will be used to render our Markdown posts into HTML. The Minima theme that I use provides a neat little hook where we can place our own custom SCSS. This is the `_sass/minima/custom-styles.scss` file, so if you don't have it already, create it now.

## Base Environment 

We first get the basic properties right for our base environment. We make the text italic, give it a border to highlight it from the text, and a little padding for some visual flair.

```scss
.base-env {
  display: block;
  font-style: italic;

  // border style and padding
  margin: 0 0 2em;
  border: 1px solid #ccc;
  padding: 1em;
  border-radius: 5px;

  ...
}
```

Now, we want to add our prefix properties. The only constant among all the possible environments is that the prefix is in bold, and definitely not in italics. This goes for any descriptive text we might want to add as well. Therefore, we add the following code in the `base-env` class (before the curly braces):

```scss
  ... 

  &:before {
    font-weight: bold;
    font-style: normal;
  } // end prefix 
  
  &[text]:before {
    font-weight: bold;
    font-style: normal;
  }// end custom text prefix
  
  ...
```

Lastly, we need to add list support. I haven't yet found a way to transform Markdown lists into HTML lists, but for now, the following code will do the trick. Add this just after the above two, but before the curly-brace (don't forget to delete the dots!)

```scss
  ...

  & ul {
    margin-bottom: 0;
    list-style-type: decimal;
    list-style-position: inside;
  } // end list-env
  
  & ul li {
    margin-bottom: .5em;
    padding-left: 20px;
  } // end list-element

} // end base-env
```

Now, we're done with the definition of our base environment.

## Specialised environments and SCSS

We can use the base environment to define all the environments that we actually want. We will include definitions and theorems here, but feel free to add the ones you like. These are all stored in the `names` dictionary. (This is all outside `base-env`!)

```scss
$names: (
  thm: 'Theorem', 
  defn: 'Definition', 
)
```

Furthermore, to have numbered blocks, each environment needs to have its own counter. We iterate over each of the `env`-`name` pairs in the `names` variable to define them dynamically. To do this, we exploit SCSS's interpolation capabilities. 

SCSS has a neat trick where we can substitute the name of our variable directly into the CSS file that it compiles into. Suppose we have a variable `var` declared as `$var: 'Variable'`. Then, `$var` will fetch its contents but `#{$var}-new` will declare a new variable called `Variable-counter`. We're going to abuse this trick :) 

For each `env`, we declare first a new block environment called `#{$env}`. Then, we can define its individual counter `#{$env}-counter`. For example, we first declare the `thm` environment, with its own counter `thm-counter`. Using this counter, we can define the numbered prefixes, with and without supplementary comments/text.

```scss
@each $env, $name in $names{
  .#{$env}{
    @extend .base-env;
    counter-increment: #{$env}-counter;
  
    &:before{
      content: "#{$name} " counter(#{$env}-counter) ".";
    }
    &[text]:before{
      content: "#{$name} " counter(#{$env}-counter) " (" attr(text) ").";
    }
   }
}
```

Don't forget to extend the base environment when defining the new ones!

## Conclusions 

Now, with any luck, you should be able to write this in your Markdown post:
```html
<div class="thm" text="Second Great Theorem">
The following are equivalent.
<ol>
<li> True </li>
<li> Not False </li>
</ol>
</div>
```

which will be rendered as

<div class="thm" text="Second Great Theorem">
The following are equivalent.
<ol>
<li> True </li>
<li> Not False </li>
</ol>
</div>

Alternatively, 
```html
<div class="defn" text="First Definition">
A prime number is a number with only 2 factors, 1 and itself. For example,
<ol>
<li> 2 is a prime number. </li>
<li> 33 is not a prime number, since it has the factors 1, 3, 11, 33. </li>
</ol>
</div>
```

which will be rendered as

<div class="defn" text="First Definition">
A prime number is a number with only 2 factors, 1 and itself. For example,
<ol>
<li> 2 is a prime number. </li>
<li> 33 is not a prime number, since it has the factors 1, 3, 11, 33. </li>
</ol>
</div>

Happy hacking!

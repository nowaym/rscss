RSCSS
=====

Reasonable Standard* for CSS Stylesheet Structure.

(`*`: "S" can also stand for "suggestions")

:construction: This document is a work in progress.

<br>

Problem
-------

Any CSS greater than 1000 lines will get unwieldy. You'll eventually run into these common pitfalls:

* "What does this class mean?"
* "Is this class still being used?"
* "If I make a new class `green`, will there be a clash?"

<br>

Structure
---------

### Think in components

![](images/component-example.png)

Think of each piece of your UI is an individual "component." Components will be named with **at least two words**, separated by a dash. Examples of a component:

* A like button (`.like-button`)
* A search form (`.search-form`)
* A news article card (`.article-card`)

### Elements

![](images/component-elements.png)

Each component may have elements. They should have classes that are only **one word**.

```scss
.search-form {
  .field { /* ... */ }
  .label { /* ... */ }
}
.article-card {
  .title { /* ... */ }
  .author { /* ... */ }
}
```

For those that need two or more words, concatenate them without dashes or underscores.

```scss
.profile-box {
  .firstname { /* ... */ }
  .lastname { /* ... */ }
  .avatar { /* ... */ }
}
```

If you're going to use elements that don't have class names, make sure that you use the `>` descendant selectors so as to avoid clashes with sub-components.

```scss
.article-card {
  > h3 { /* ... */ }
}
```

But in general, avoid tag selectors whenever possible.

### Variants

![](images/component-modifiers.png)

Components may have variants. Their classes will be prefixed by a dash (`-`).

```scss
.like-button {
  &.-wide { /* ... */ }
  &.-short { /* ... */ }
  &.-disabled { /* ... */ }
}
```

Elements may also have variants.

```scss
.shopping-card {
  .title { /* ... */ }
  .title.-small { /* ... */ }
}
```

Why a dash? Because:

* it prevents ambiguity with elements
* a CSS class can only start with a letter, `_` or `-`
* dashes are easier to type than underscores
* it kind of resembles switches in UNIX commands (`gcc -O2 -Wall -emit-last`)

### Nested components

![](images/component-nesting.png)

Sometimes it's necessary to nest components.

```html
<div class='article-link'>
  <div class='vote-box'>
    ...
  </div>
  <h3 class='title'>...</h3>
  <p class='meta'>...</p>
</div>
```

### Simplifying nested components

Sometimes, when nesting components, your markup can get dirty:

```html
<div class='search-form'>
  <input class='input' type='text'>
  <button class='search-button -red -large'></button>
</div>
```

You can simplify this by using your CSS preprocessor's `@extend` mechanism:

```html
<div class='search-form'>
  <input class='input' type='text'>
  <button class='submit'></button>
</div>
```

```scss
// sass
.search-form {
  .submit {
    @extend .search-button;
    @extend .search-button.-red;
    @extend .search-button.-large;
  }
}
```

### Layout

![](images/layouts.png)

Components should be made in a way that they're reusable in different contexts. Avoid putting these properties in components:

* Positioning (`position`, `top`, `left`, `right`, `bottom`)
* Floats (`float`, `clear`)
* Margins (`margin`)
* Dimensions (`width`, `height`) *

Exception here would be elements with a fixed width, such as avatars and logos.

If you need to define these, try to define them in whatever context whey will be in. In this example below, notice that the widths and floats are applied on the *list* component, not the component itself.

```css
.article-list {
  & {
    @include clearfix;
  }

  .article-card {
    width: 33.3%;
    float: left;
  }
}

.article-card {
  & { /* ... */ }
  .image { /* ... */ }
  .title { /* ... */ }
  .category { /* ... */ }
}
```

<br>

CSS structure
-------------

### One component per file

```scss
/* css/components/search-form.scss */
.search-form {
  .button { /* ... */ }
  .field { /* ... */ }
  .label { /* ... */ }

  // variants
  &.-small { /* ... */ }
  &.-wide { /* ... */ }
}
```

In sass-rails and stylus, this makes including all of them easy:

```scss
@import 'components/*';
```

### Avoid over-nesting

Use no more than 1 level of nesting. It's easy to get lost with too much nesting.

```scss
/* bad */
.image-frame {
  .description {
    /* ... */

    > strong {
      /* ... */
    }
  }
}
```

Consider instead:

```scss
.image-frame {
  .description { /* ... */ }
  .description > strong { /* ... */ }
}
```

<br>

Pitfalls
--------

### Bleeding through nested components

Be careful about nested components where the nested component has an element of the same name.

```html
<article class='article-link'>
  <div class='vote-box'>
    <button class='up'></button>
    <button class='down'></button>
    <span class='count'>4</span>
  </div>

  <h3 class='title'>Article title</h3>
  <p class='count'>3 votes</p>
</article>
```

```scss
.article-link {
  .title { /* ... */ }
  .count { /* ... (!!!) */ }
}

.vote-button {
  .up { /* ... */ }
  .down { /* ... */ }
  .count { /* ... */ }
}
```

In this case, `.article-link .count` will also apply to the `.vote-button .count` element. To prevent this, you may need to use descendant selectors.

<br>

But...
------

### But dashes suck

You're free to omit it and just use regular words, but keep the rest of the ideas in place (components, elements, variants).

### But I can't think of 2 words

Some components will only need one word to express their purpose, such as `alert`. In these cases, consider that using some suffixes will make it clearer that it's a block-level element:

* `.alert-box`
* `.alert-card`
* `.alert-block`

Or for inlines:

* `.link-button`
* `.link-span`

<br>

Other solutions
---------------

### BEM

[BEM] in nice, but some may be irked at its unconventional syntax. RSCSS pretty much follows BEM conventions, only with a different syntax.

```html
<!-- BEM -->
<form class='site-search site-search--full'>
  <input  class='site-search__field' type='text'>
  <button class='site-search__button'></button>
</form>
```

```html
<!-- rscss -->
<form class='site-search -full'>
  <input  class='field' type='text'>
  <button class='button'></button>
</form>
```

### Terminologies

The same concepts exist in similar ways in other CSS structuring ideologies.

| RSCSS     | BEM      | SMACSS        |
| ---       | ---      | ---           |
| Component | Block    | Module        |
| Element   | Element  | ?             |
| Layout    | ?        | Layout        |
| Variant   | Modifier | Theme & State |

<br>

Summary
-------

* Think in **components**, named with 2 words (`.screenshot-image`)
* Components have **elements**, named with 1 word (`.blog-post .title`)
* Name **variants** with a dash prefix (`.shop-banner.-with-icon`)
* Components can nest
* Remember you can extend to make things simple

[Smacss]: https://smacss.com/
[BEM]: http://bem.info/

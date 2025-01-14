---
title: Component shortcodes
---

You can embed React, Vue, Svelte, and more into your existing templates. Let's learn how!

## Prerequisites

{% include 'prereqs.md' %}

## Basic usage

Using the `component` [shortcode](https://www.11ty.dev/docs/shortcodes/), you can insert components into any static template that 11ty supports.

First, ensure your component is **located in your includes directory.** This defaults to `_includes` for new 11ty projects, but you can [override this from your eleventy config](https://www.11ty.dev/docs/config/#directory-for-includes).

For instance, say you've written a Vue component under `_includes/Component.vue`. You can insert this component into your templates like so:

{% slottedComponent 'Tabs.svelte', hydrate=true, id='shortcode-basics', store='templates', tabs=['nunjucks', 'liquid'] %}
{% renderTemplate 'md' %}
<section>

```html
{% component 'Component.vue' %}
```
</section>
<section hidden>

```html
{% component 'Component.vue' %}
```
</section>
{% endrenderTemplate %}
{% endslottedComponent %}

> These examples work from markdown (`.md`) and HTML (`.html`) files as well! You can use liquid syntax within either of these by default, though we recommend using Nunjucks instead. See [our recommend config options](/docs/config/#recommended-config-options) for more.

This will do a few things:
1. Find `_includes/Component.vue`. Note that 1) we'll always look inside the `_includes` directory to find your components, and 2) the file extension is _required_ (`.jsx`, `.tsx`, `.svelte`, etc).
2. [Prerender](https://jamstack.org/glossary/pre-render/) your component at build time. This means you'll always see your component, even when disabling JS in your browser ([try it!](https://developer.chrome.com/docs/devtools/javascript/disable/)).

One feature we _won't_ provide automatically: hydrating on the client. In other words, your React `useState`, Vue `refs`, or Svelte stores won't work immediately. You'll need to opt-in to sending JavaScript to the browser, which brings us to our next section...

## Choose how and when to hydrate

Slinkity requires a special `hydrate` or `renderWithoutSSR` prop to ship JavaScript alongside your components. This lets you use your favorite component framework for templating guilt-free, and opt-in to JS bundles when the need arises.

To hydrate that `Component.vue` from earlier, you can apply the `hydrate=true` flag:

{% include 'examples/hydrate-component-shortcodes.md' %}

For a full list of options to fine-tune how and when JavaScript is loaded on the client...

**[💧 Learn more about partial hydration →](/docs/partial-hydration)**

## Pass props to shortcodes

You can also pass data to your components as key / value pairs.

Let's say you have a date component written in your favorite framework (`_includes/Date.jsx|vue|svelte`) that wants to use [11ty's supplied `date` object](https://www.11ty.dev/docs/data-eleventy-supplied/). You can pass this "date" prop like so:

{% slottedComponent 'Tabs.svelte', hydrate=true, id='shortcode-date-prop', store='templates', tabs=['nunjucks', 'liquid'] %}
{% renderTemplate 'md' %}
<section>

```html
{% component 'Date.jsx|vue|svelte', date=page.date %}
```

Note that you can pass the `hydrate` flag alongside this prop as well. `hydrate` and `renderWithoutSSR` are considered just another prop, like any other! This should work fine for instance:

```html
{% component 'Date.jsx|vue|svelte', date=page.date, hydrate=true %}
```
</section>
<section hidden>

```html
{% component 'Date.jsx|vue|svelte' 'date' page.date %}
```

Note that you can pass the `hydrate` flag alongside this prop as well. `hydrate` is considered just another prop, like any other! This should work fine for instance:

```html
{% component 'Date.jsx|vue|svelte' 'date' page.date 'hydrate' true %}
```
</section>
{% endrenderTemplate %}
{% endslottedComponent %}

You can access either of these props inside your component like so:

{% slottedComponent "Tabs.svelte", hydrate="eager", id="component-props", tabs=["React", "Vue", "Svelte"] %}
{% renderTemplate "md" %}
<section>

```jsx
export default function ViewDate({ date, hydrate }) {
  return (
    <>
      <span>{date}</span>
      {/* ex. only show a button when the component is hydrated */}
      {hydrate ? <button>Something interactive!</button> : null}
    </>
  )
}
```
</section>
<section hidden>

```html
<template>
  <span>{{ date }}</span>
  <!--ex. only show a button when the component is hydrated-->
  <button v-if="hydrate">Something interactive!</button>
</template>

<script>
export default {
  props: ["date", "hydrate"],
}
</script>
```
</section>
<section hidden>

```html
<script>
  export let date = '';
  export let hydrate = '';
</script>

<span>{date}</span>
{#if hydrate}
<!--ex. only show a button when the component is hydrated-->
<button>Something interactive!</button>
{/if}
```
</section>
{% endrenderTemplate %}
{% endslottedComponent %}

### Pass multiple props

You're free to pass as many key / value pairs as you want. The names and ordering of your keys shouldn't matter, since they're all crunched into a single "props" object for your component.

{% slottedComponent 'Tabs.svelte', hydrate=true, id='shortcode-multiple-props', store='templates', tabs=['nunjucks', 'liquid'] %}
{% renderTemplate 'md' %}
<section>

```html
{% component 'Apropcalypse.jsx|vue|svelte', date=page.date,
url=page.url, hydrate=true, fileSlug=page.fileSlug %}
```
</section>
<section hidden>

```html
{% component 'Apropcalypse.jsx|vue|svelte' 'date' page.date
'url' page.url 'hydrate' true 'fileSlug' page.fileSlug %}
```
</section>
{% endrenderTemplate %}
{% endslottedComponent %}

> Is that a "hydrate" flag sandwiched between the other props? Yep! Do we care? Nope.

## Pass children / unnamed slots to component

We have a separate [paired shortcode](https://www.11ty.dev/docs/shortcodes/#paired-shortcodes) for passing child HTML: `slottedComponent`. 

Say you have a simple component to wrap text with a dropdown toggle. That component could be written like so:

{% slottedComponent "Tabs.svelte", hydrate="eager", id="slotted-component", tabs=["React", "Vue", "Svelte"] %}
{% renderTemplate "md" %}
<section>

```jsx
export default function Dropdown({ heading, children }) {
  return (
    <details>
      <summary>{heading}</summary>
      {children}
    </details>
  )
}
```
</section>
<section hidden>

```html
<template>
  <details>
    <summary>{{ heading }}</summary>
    <slot />
  </details>
</template>

<script>
export default {
  props: ["heading"],
};
</script>
```
</section>
<section hidden>

```html
<script>
  export let heading = "";
</script>

<details>
  <summary>{heading}</summary>
  <slot />
</details>
```
</section>
{% endrenderTemplate %}
{% endslottedComponent %}

You can use this component the same way as the `component` shortcode, now with children:

{% slottedComponent 'Tabs.svelte', hydrate=true, id='slotted-shortcode-usage', store='templates', tabs=['nunjucks', 'liquid'] %}
{% renderTemplate 'md' %}
<section>

```html
{% slottedComponent 'Dropdown.jsx|vue|svelte', heading='Full disclosure' %}
<p>"details" and "summary" are kinda confusing element names</p>
{% endslottedComponent %}
```
</section>
<section hidden>

```html
{% slottedComponent 'Dropdown.jsx|vue|svelte' 'heading' 'Full disclosure' %}
<p>"details" and "summary" are kinda confusing element names</p>
{% endslottedComponent %}
```
</section>
{% endrenderTemplate %}
{% endslottedComponent %}

### Important gotcha: templating in children

You may be rushing to try slotted components in your markdown:

```md{% raw %}
{% slottedComponent 'FancyBackground.jsx|vue|svelte' %}
### Why I love markdown

- Bulleted lists are easy
- Paragraph and code blocks are even easier

{% endslottedComponent %}{% endraw %}
```

🚨 **Careful, this won't work as written!** Paired shortcode content is processed as plain HTML, so markdown syntax won't work as expected 😢

But all is not lost. Since you _can_ still nest other shortcodes as paired shortcode content, you can use [11ty's handy `renderTemplate`](https://www.11ty.dev/docs/plugins/render/) like so:

```md{% raw %}
{% slottedComponent 'FancyBackground.jsx|vue|svelte' %}
{% renderTemplate 'md' %}
### Why I love markdown

- Bulleted lists are easy
- Paragraph and code blocks are even easier

{% endrenderTemplate %}
{% endslottedComponent %}{% endraw %}
```

> Be sure to set up `renderTemplate` in your 11ty config before trying this. [See their docs](https://www.11ty.dev/docs/plugins/render/) for more.

This will process your markdown, then pass the result to your component.

***

So injecting components into templates is nice... but what if we want to build the entire route using a component framework?

**[Learn about page-level components →](/docs/component-pages-layouts)**
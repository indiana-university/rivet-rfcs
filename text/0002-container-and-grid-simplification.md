# Rivet container and grid simplification
- Start Date: (Fill me in with today's date, 2019-11-26)
- RFC PR: (Leave empty—will fill in once PR exists)
- Rivet Issue or PR: (Leave this empty—will fill in with implementation PR if accepted)

## Summary
This RFC proposes a few updates and additions to the Rivet container and grid system that aim to make it a bit easier to use and understand and make it more flexible.

## Motivation
After 2-plus years and some valuable feedback from developers, it's seems clear that we could reduce the amount of CSS classes it takes to create a container for layout. I would like to see a reduction in the amount of CSS classes it takes to create a layout container with a max-width that is centered once the viewport is larger than that max-width. 

In addition to reducing the amount of CSS classes for containers the class names for the grid could use an naming update to help make it easier to understand how they work. E.g.`rvt-columns-3` or `rvt-cols-3` is more descriptive of what the CSS class does than `rvt-grid__item-3`. We'll dig deeper into this in the _Detailed design_ section that follows.

## Detailed design
This section will compare the current container and grid functionality/design with the proposed updated functionality.

### Current container functionality
Currently you would need this markup in Rivet to create a layout container that is centered and has a max-width.

```html
<div class="rvt-container rvt-container--junior rvt-container--center">
  Some content with a max-width of 1140px
</div>
```

It takes three CSS classes to create a what is generally the default behavior for a layout container (a max-width that is centered). I say _generally_ because sometimes a developer might want a fluid container (100% width), **but that should be the exception not the rule**. And, with Rivet utility classes, making a fluid container with padding on either side is just as simple as using the base `rvt-container` CSS class.

```html
<div class="rvt-p-lr-lg">
  <!--
    A generic element that will have 2rem of padding on the
    left and right sides.
  -->
</div>
```

### Proposed container conventions
To make it easer and less verbose to create a container, these new container conventions and functionality should do the following:

1. Each container size should handle its own `max-width` and `margin` properties, which eliminates the need for a BEM modifier-based convention. E.g. `rvt-container rvt-container--junior rvt-container--center`
1. The container size naming conventions should be brought inline with all other Rivet size naming conventions e.g. `-sm, -md, -lg, -xl, etc.`

Given those requirements a new Rivet container element would looks something like this.

```html
<!-- Now a "small" (formerly "--freshman") container only takes one CSS class-->
<div class="rvt-container-sm">
  <!-- content goes here -->
</div>

<!-- Now a "medium" (formerly "--sophomore") container only takes one CSS class-->
<div class="rvt-container-md">
  <!-- content goes here -->
</div>

<!-- Now a "large" (formerly "--junior") container only takes one CSS class-->
<div class="rvt-container-lg">
  <!-- content goes here -->
</div>

<!-- Now a "Extra-large" (formerly "--senior") container only takes one CSS class-->
<div class="rvt-container-xl">
  <!-- content goes here -->
</div>
```

### Proposed grid conventions
As a part of this update, this RFC also proposes that the grid CSS class names be updated. Similar to the the container proposal in the previous section, the updated grid naming conventions aim to make creating grids faster and more readable for developers. To accomplish that, this RFC proposes the following:

1. Rename `rvt-grid` to `rvt-row`
1. Rename grid `__item-*` to `-cols-*`
1. Rethink the responsive variant naming conventions E.g. `__item-4-md-up`, more specifically the `-md-up` part.

Given those proposed changes, and new grid might be written something like the following:

```html
<!-- rvt-grid becomes rvt-row -->
<div class="rvt-row">
  <!--
    rvt-grid__item-4-md-up becomes rvt-cols-4-md. The responsive functionality
    would remain the same, except all breakpoints would be specified
    using only the breakpoint suffix e.g. `-md` instead of `-md-up`.
  -->
  <div class="rvt-cols-4-md">
    Four columns
  </div>
  <div class="rvt-cols-4-md">
    Four columns
  </div>
  <div class="rvt-cols-4-md">
    Four columns
  </div>
</div>
```

### New feature: Gutter variants
As a new feature proposal, it would be useful for developers to be able to easily modify the size of grid gutters based on the design for their site, or application. For example, a marketing website with expressive typography and generous use of white space, might benefit from more generous gutters.

To do this we could create a set of modifier classes for the new `rvt-row` element that set's the child `-cols-*` elements gutters based which variant the developer chooses.

Take this example:

```html
<div class="rvt-row rvt-row--loose">
  <!--
    The `--loose` modifier here would set a more generous padding value for
    each of the `rvt-cols-*` children inside of it.
  -->
  <div class="rvt-cols-3-md">
    Three columns
  </div>
  <div class="rvt-cols-5-md">
    Five columns
  </div>
  <div class="rvt-cols-4-md">
    Four columns
  </div>
</div>
```

We could accomplish this fairly easily using [substring matching attribute selectors](https://www.w3.org/TR/selectors-3/#attribute-substrings) which are supported back to at least IE11.

Example:

```css
.rvt-row {
  display: flex;
  /*
   * This cancels out extra padding on outside grid columns. In this case
   * the default grid gutter would be 1.5rem.
  */
  margin: 0 -1.5rem;
}

.rvt-row--loose {
  margin: 0 -2.5rem
}

.rvt-row--loose > [class^="rvt-col"] {
  /* All `rvt-cols-*` inside the modifier would get increased padding */
  padding: 0 2.5rem;
}
```

#### Potential gutter variants
- `rvt-row` (default) = 1.5rem gutter — Sensible, all-purpose default
- `rvt-row--tight` = .75rem gutter — Good for complex applications where screen space is at a premium.
- `rvt-row--loose` = 2.5rem — Good for expressive marketing layouts that make generous use of white space, etc.

## How we teach this
For the most part the core functionality and concepts of the Rivet grid would remain the same. Based on the changes laid out in the previous sections, common use cases of the current grid could be mostly migrated from the old version using a find and replace.

In the next version of the docs we should make sure to call this out, and make it part of the migration guide from 1.x.x to 2.0.0.

## Drawbacks
The only drawbacks would be any migration effort fo developers, but as is mentioned above, the majority of that work could be accomplished via find and replace functionality available in almost every IDE/code editor.

## Alternatives
None other than keeping things the same.

## Unresolved questions
Biggest question here is how this affects other responsive sizing naming conventions. We're proposing changing that for the grid here (e.g. `-md-up`), so that means this change would need to be made in other places in the codebase to keep things consistent.

For example the responsive type scale classes. `rvt-ts-16-md-up` sets the `font-size` of an element to  `1rem` at the medium breakpoint. Updates would be need to made to make these two conventions match, e.g. `rvt-ts-16-md`
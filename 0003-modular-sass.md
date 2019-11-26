# Modular Sass
- Start Date: 2019-11-26
- RFC PR: (Leave empty—will fill in once PR exists)
- Rivet Issue or PR: (Leave this empty—will fill in with implementation PR if accepted)

## Summary
This RFC proposes a new completely modular approach to Rivet's Sass (SCSS) code base with a goal of making it easer for developers to include only the components and styles that they need for their site or application.

A lot of the concepts in this RFC are very much inspired by the way the Github Primer team [has structured their codebase](https://github.com/primer/css).

## Motivation
In the 1.x.x versions of Rivet the same Sass source files that we build Rivet with have always been included in the npm package and downloads. While it has always been possible for developers to use Rivet's Sass source files we (the Rivet team):

1. Haven't done the best job of documenting how the Sass files are structured
1. Haven't made it easy for those who do you Sass in their tool chain to use only the components and styles they need to import

In this RFC we'll outline a plan for restructuring the Sass code base to make it easier for developers to use how they want/need to with their own build tools.

## Detailed design
The majority of this RFC will focus on a proposed file structure for organizing `.scss` files so that they are more modular where each component is self-contained (.i.e `@import`s all the dependencies it needs) and can be imported by itself on an as-needed basis.

### Standard component file structure
To create a predictable experience for developers wanting to import we'll need to first establish a standard directory structure for each component in our Sass code base. Each directory/component should meet the following criteria:

- Have its own entry point that follows a standard name. The most familiar convention would be `_index.scss`
- All CSS producing component styles should be imported into `_index.scss`. This provides the flexibility to split complex components up into small .scss partials for better readability and organization.
- All components `_index.scss` files should import a "core" set of global, non-CSS-producing variables, mixins, etc. This keeps every component isolated so that it could be imported by a developer even if they forgot to include/import Rivet's global Sass variables.

#### Standard component directory structure
The following is a proposed standard directory structure using Rivet's modal component as an example.

```shell
/src/sass/modal/
  |-- _index.scss
  |-- _modal-header.scss
  |-- _modal-body.scss
  |-- _modal-footer.scss
```

The contents of the `_index.scss` file would container the following.

```scss
// Import all global dependencies (e.g. _variables.scss, _mixins.scss)
@import "../core/index";

/// Import all modal partials (relative to _index.scss)
@import "./modal-header";
@import "./modal-body";
@import "./modal-footer";
```

With this standard structure in place, a developer could import any component in Rivet without needing to know about the global "core" dependencies.

### The "core"
The core should contain all the global variables, mixins, and functions all other components depend on. **Noting in the core should actual produce a CSS ouput**.

Here's an example of how the core could be structured.

```shell
/src/sass/core
  |-- _index.scss
  |-- /variables
    |-- _colors.scss
    |-- _fonts.scss
    |-- _spacing.scss
    |-- _type-scale.scss
    ...
  |-- /mixins
    |-- _media-query.scss
    |-- _misc.scss
    ...
```

The contents of the `/src/sass/core/_index.scss` file would container the following.

```scss
@import "./variables/colors";
@import "./variables/fonts";
@import "./variables/spacing";
@import "./variables/type-scale";

// rest of variable imports ...
@import "./mixins/media-query";
@import "./mixins/misc";
// rest of mixin imports
```

### The "base" import
One thing we'll need to think about is that some components' styles will be dependent on _globally_ applied CSS, or CSS declarations with generic HTML element selectors.

Take these CSS selectors as an example.

```scss
body {
  line-height: 1.5;
}

h1,
h2,
h3,
h4,
h5,
h6 {
  line-height: 1.2;
  // Some other styles...
}

// Rest of globally applied CSS
```

This types of "global" generically applied CSS would need to be contained in it's own set of Sass partials/imports. I would be fairly straightforward to keep those in their own directory following the standard file structure outlined in the previous section.

```shell
/src/sass/base/
  |-- _index.scss
  |-- _base.scss
  |-- _normalize.scss
  |-- _fonts.scss
  ... Additional "base" declarations (e.g. generic element selectors)
```

It will be important to make it clear in the documentation that **this "base" import is required** for all of the other components to render properly. Given that requirement, a developer could then install rivet via npm and then begin building their own custom stylesheet via their tool chain/build.

These file paths are quite long, but developers using node-sass could take advantage of the [`includePaths` option](https://github.com/sass/node-sass#includepaths) to specify a path to the Rivet npm package for easier importing.

```scss
// Import the required base package
@import "../../node_modules/path-to-rivet-sass/base/index";

// Start importing only what you need
@import "../../node_modules/path-to-rivet-sass/utilities/index";
@import "../../node_modules/path-to-rivet-sass/dropdown/index";
@import "../../node_modules/path-to-rivet-sass/modal/index";
@import "../../node_modules/path-to-rivet-sass/pagination/index";
// That's it. That's all I need...
```

## How we teach this
As mentioned in the summary of this RFC, the Rivet 1.x.x Sass codebase has suffered from a lack of documentation. Most of the functionality proposed in this RFC will need to be thoroughly documented with dedicated section on the new Rivet documentation site.

### Axioms and shared language
It will be important to develop a shared language for the way we talk about this new, more modular Sass architecture.

- By **_core_** we mean all the global variables, mixins, and functions all other components depend on. **Noting in the core should actual produce a CSS ouput**.
- By **_base import_** we mean all the globally applied CSS, or CSS declarations that use generic HTML elements as selectors.

## Drawbacks
The main drawback is that this is a breaking change for those who have taken advantage of using the Rivet 1.x.x Sass source in their projects.

## Unresolved questions
There may be some dependency on the potential _design tokens_ concept the Rivet team is planning to implement.

If a design tokens strategy is implemented as laid out [in this issue](https://github.com/indiana-university/rivet-source/issues/214), there would need to be some coordination in how the main `rivet-source` codebase consumes Sass variable files generated from design tokens.
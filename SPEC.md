**NOTE:** This is a copy of the official KSS spec, which is available at [github.com/kneath/kss](https://github.com/kneath/kss/blob/master/SPEC.md). This copy includes modifications of the spec which only apply to the kss-node implementation available at [github.com/kss-node/kss-node](https://github.com/kss-node/kss-node). The kss-node-specific differences will be clearly marked with the text: **kss-node only:**

# Knyle Style Sheets

KSS attempts to provide a methodology for writing maintainable, documented CSS within a team. Specifically, KSS is a documentation specification and style guide format. It is **not** a preprocessor, CSS framework, naming convention, or specificity guideline.

However, there are several software projects that provide preprocessors that read and parse KSS-compliant documentation to produce full-blown style guides in HTML. Most of those implementations provide:

1. a tool to generate a style guide by parsing CSS source files to find KSS docs, and
2. a language-dependent API that represents the style guide as a data structure.

This specification does not discuss those implementations.

**kss-node only:** (Actually, this copy of the spec *does* discuss one of those implementations.) For example, here's how kss-node is related to kss:

![KSS is a spec for docs. kss-node is a preprocessor and JavaScript API.](https://raw.githubusercontent.com/kss-node/kss/spec/kss-vs-kss-node.png)

## Purpose

KSS is a set of guidelines to help you produce an HTML style guide tied to CSS documentation that is nice to read in plain text, yet structured enough to be automatically extracted and processed by a machine. It is designed with CSS preprocessors (such as Sass or LESS) in mind, and flexible enough to accommodate a multitude of CSS frameworks (such as Bootstrap or Foundation).

KSS focuses on *how people work with CSS* — it does not define code structures, naming conventions, or methods for abstraction. It is important to understand that the style guide format and documentation format are intrinsically tied to one another.

## Style Documentation

Not every CSS rule should be documented. You should document a rule declaration when the rule can accurately describe a visual UI element in the style guide. Each element should have one documentation block describing that particular UI element's various states.

KSS documentation is hierarchical in nature — any documentation block at any point within the style guide hierarchy applies to the documentation blocks beneath that level. This means that documentation for 2.1 applies to documentation for 2.1.3.

### Format

The basic format for KSS documentation can be explained best in an example:

```css
/*
Star button

A button suitable for giving stars to someone.

:hover             - Subtle hover highlight.
.stars-given       - A highlight indicating you've already given a star.
.stars-given:hover - Subtle hover highlight on top of stars-given styling.
.disabled          - Dims the button to indicate it cannot be used.

Styleguide 2.1.3
*/
.star-button {
  ...
}
.star-button.stars-given {
  ...
}
.star-button.disabled {
  ...
}
```

When using a preprocessor that supports the functionality, use `//` to prefix your comment sections (Sass example):

```scss
// Star button
//
// A button suitable for giving stars to someone.
//
// :hover             - Subtle hover highlight.
// .stars-given       - A highlight indicating you've already given a star.
// .stars-given:hover - Subtle hover highlight on top of stars-given styling.
// .disabled          - Dims the button to indicate it cannot be used.
//
// Styleguide 2.1.3
.star-button {
  ...
  &.stars-given {
    ...
  }
  &.disabled {
    ...
  }
}
```

Each KSS documentation block consists of two required parts and a few optional parts:

1. a heading *(required)*
2. a description of what the style does or looks like *(optional)*
3. **kss-node only:** the name of file containing the HTML markup the CSS applies to, or a copy of the markup inline *(optional)*
4. a list of modifier classes or pseudo-classes and how they modify the style *(optional)*
5. a reference to the style's position in the style guide *(required)*
6. **kss-node only:** a numerical weight that can be used to re-position a style outside of the normal alphabetical order *(optional)*

### The heading and description sections

The description should be plain sentences of what the CSS rule or hierarchy does or looks like. A good description gives guidance toward the application of elements the CSS rules style. The first paragraph in the description will be used as the heading for that style guide section.

```scss
// Activity feed
//
// A feed of activity items. Within each section, there should be many
// articles which are the feed items.
```

CSS rules that depend on specific HTML structures should describe those structures using `<element#id.class:pseudo>` notation. For example:

```scss
// A feed of activity items. Within each <section.feed>, there should be many
// <article>s which are the feed items.
```

To describe the status of a set of rules, you should prefix the description with **Experimental** or **Deprecated**.

**Experimental** indicates CSS rules that apply to experimental styling. This can be useful when testing out new designs before they launch (staff only), alternative layouts in A/B tests, or beta features.

```scss
// Experimental: An alternative signup button styling used in AB Test #195.
```

**Deprecated** indicates that the rule is slated for removal. Rules that are deprecated should not be used in future development. This description should explain what developers should do when encountering this style.

```scss
// Deprecated: Styling for legacy wikis. We'll drop support for these wikis on
// July 13, 2007.
```

### The modifiers section

If the UI element you are documenting has multiple states or styles depending on added classes or pseudo-classes, you should document them in the modifiers section.

```scss
// :hover             - Subtle hover highlight.
// .stars-given       - A highlight indicating you've already given a star.
// .stars-given:hover - Subtle hover highlight on top of stars-given styling.
// .disabled          - Dims the button to indicate it cannot be used.
```

### The markup section

**kss-node only:**

You can include sample markup in your style guide entries. This is not only helpful for newcomers to a project, but is also used by the kss-node generator to include samples in your style guide. Just start a paragraph in your section with `Markup:` like so:

```scss
// Button
//
// Buttons can and should be clicked.
//
// Markup:
// <button class="button {{modifier_class}}">
//
// :hover - Highlight the button when hovered.
//
// Styleguide 1.1
```

If you include `{{modifier_class}}` in the markup, the generated style guide will be able to use the correct CSS class when it displays the sample for each of the modifiers listed in the modifier section.

Instead of using inline HTML markup, you could point at a separate file that contains your HTML. kss-node supports either a plain *.html file or a Handlebars *.hbs file. The kss-node generator will search for the markup file in any of the `--source` folders.

```scss
// Button
//
// Markup: button.hbs
//
// Styleguide 1.1
```

Lastly, if your application or component library is built with Node.js, instead of using the `markup` section, you could use the JavaScript API provided to parse the KSS documentation and build a style guide using the HTML templating library of your choice.

### The styleguide section

If the UI element you are documenting has an example in the style guide, you should reference it using the "Styleguide [ref]" syntax.

```scss
// Styleguide 2.1.3.
```

References can be integer sections separated by periods. Each period denotes a hierarchy of the style guide. Style guide references can point to entire sections, a portion of the section, or a specific example.

References may also be period seperated word keys. Leading words denote hierarchy.

```scss
// Styleguide Forms.Checkboxes.
```

Finally, references may be more readable word phrases.

```scss
// Styleguide Forms - Special Checkboxes
```

If there is no example, then you must note that there is no reference.

```scss
// No styleguide reference.
```

**kss-node only:** If word keys or word phrases are used, kss-node will automatically generate a hierarchal number for each section since "section 4.1.3" is easier to communicate to others than "section forms - widgets - special checkboxes".

### The weight section

**kss-node only:**

If you are using word keys or word phrases as your style guide references, kss-node will sort all sections using alphabetical order. This can be problematic if, for example, you want the documentation about `sass.variables` to come before the docs about `sass.mixins`.

With kss-node, you can specify a numeric weight for style guide section that overrides the default alphabetical ordering of sibling sections. If no weight is specified for a section, the default weight of 0 is used. Lighter weights (like -10) will "rise above" sibling sections with the default weight of 0 and heavier weights (like 10) will "sink below".

For example:

```scss
// Sass mixins
//
// Style guide: sass.mixins

// Sass colors
//
// Style guide: sass.colors

// Sass variables
//
// Weight: -10
//
// Style guide: sass.variables

// Sass
//
// Weight: -1
//
// Style guide: sass

// Components
//
// Style guide: components
```

The above KSS docs will produce a style guide hierarchy like this:

```
1: Sass
  1.1: Sass variables
  1.2: Sass colors
  1.3: Sass mixins
2: Components
```

First, you'll notice that kss-node has auto-generated hierarchal numbering for you. Also note that the weights only affect the ordering of siblings in the hierarchy; the weight of `sass.variables` only affects its ordering relative to `sass.colors` and `sass.mixins`.

Finally, any sibling sections with the same weight will be put into alphabetical order next to each other. In our example, if a new `sass.modules` section is added with `Weight: -10`, it would be alphabetically sorted before the `sass.variables` section with `Weight: -10`.

## Preprocessor Helper Documentation

If you use a CSS preprocessor like Sass or LESS, you should document all helper code (like functions or mixins).

```scss
// gradient($start, $end)
//
// Creates a linear gradient background, from top to bottom.
//
// $start - The color hex at the top.
// $end   - The color hex at the bottom.
//
// Compatible in IE6+, Firefox 2+, Safari 4+.
//
// Styleguide sass.mixins.gradient
@mixin gradient($start, $end) {
  ...
}
```

Each documentation block should have a description section, parameters section, and compatibility section. The description section follows the same guidelines as style documentation.

### The parameters section

If the mixin takes parameters, you should document each parameter and describe what sort of input it expects (hex, number, etc).

```scss
// $start - The color hex at the top.
// $end   - The color hex at the bottom.
```

### The compatibility section

You must list out what browsers this helper method is compatible in.

```scss
// Compatible in IE6+, Firefox 2+, Safari 4+.
```

If you do not know the compatibility, you should state as such.

```scss
// Compatibility untested.
```

## Style guide

In order to fully take advantage of KSS, you should create a living style guide. A living style guide is a *part of your application* and includes all of the CSS, Javascript, and layout the rest of your application does.

### Organization

The style guide should be organized by numbered sections. These sections can go as deep as you like. Every element should have a numbered section to refer to.

**kss-node only:** If word keys or word phrases are used instead of numbered sections, kss-node will automatically generate a hierarchal number for each section based on the alphabetical and weight-based sorting of the sections.

For example:

    1. Buttons
      1.1 Form Buttons
        1.1.1 Generic form button
        1.1.2 Special form button
      1.2 Social buttons
      1.3 Miscelaneous buttons
    2. Form elements
      2.1 Text fields
      2.2 Radio and checkboxes
    3. Text styling
    4. Tables
      4.1 Number tables
      4.2 Diagram tables

The goal here is to create an organizational structure that is flexible, but rigid enough to be machine processed and referenced inside of documentation.

### Example

This style guide is automatically generated from KSS documentation using the ruby library.

![](http://share.kyleneath.com/captures/Styleguide_-_GitHub_Team-20111202-160539.png)

**kss-node only:** And this style guide is automatically generated from KSS documentation using the kss-node generator: http://kss-node.github.io/kss-node/

The actual templates generating the style guide just reference the Styleguide section and example HTML. The modified states are generated automatically. Refer to the README for more information on how to generate style guides using the KSS ruby library.

**kss-node only:** Refer to the [kss-node README](https://github.com/kss-node/kss-node#kss-node-) for more information on how to generate style guides using the kss-node generator.

Overall, keep in mind that style guides should adapt to the application they are referencing and be easy to maintain and as automatic as possible.

### Acknowledgements

KSS was inspired by [TomDoc](http://tomdoc.org).

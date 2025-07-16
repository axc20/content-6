# Accessibility

## Research

- [a11y 101](https://a11y-101.com/)
  - [Tables](https://a11y-101.com/development/tables)
  - [Skip Link](https://a11y-101.com/development/skip-link)
- [a11y style guide](https://a11y-style-guide.com/style-guide/)
- [navigation menu](https://www.w3.org/WAI/ARIA/apg/example-index/disclosure/disclosure-navigation.html)
- collapsible sections
  - https://inclusive-components.design/collapsible-sections/
  - https://accessibility.huit.harvard.edu/technique-expandable-sections
- modal
  - [dialog heading level -> h2](https://github.com/w3c/aria-practices/issues/551)
  - https://www.w3.org/WAI/ARIA/apg/example-index/dialog-modal/dialog
  - https://w3.org/TR/wai-aria-practices-1.2/examples/dialog-modal/dialog.html
- tabs
  - [role="presentation"](https://stackoverflow.com/questions/31028008/why-do-bootstrap-tabs-have-role-presentation)
  - https://www.w3.org/WAI/ARIA/apg/example-index/tabs/tabs-automatic
- [don't use display: none;](https://css-tricks.com/places-its-tempting-to-use-display-none-but-dont/)
- [role="status"](https://stackoverflow.com/questions/48894010/accessibility-regarding-role-status)

## Resources

- [<abbr title="Web Accessibility Initiative - Accessible Rich Internet Applications">WAI-ARIA</abbr> Recommendation](https://www.w3.org/TR/wai-aria-1.1)
- [<abbr title="Web Accessibility Initiative - Accessible Rich Internet Applications">WAI-ARIA</abbr> Authoring Practices](https://www.w3.org/TR/wai-aria-practices-1.1)
- [<abbr title="Web Content Accessibility Guidelines">WCAG</abbr>](https://www.w3.org/TR/WCAG21)
- MDN docs
  - [ARIA live regions (aria-live)](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)
  - [aria-busy](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-busy)
  - [aria-hidden](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-hidden): Elements may be removed from the accessibility tree and hidden from screen readers by setting the `aria-hidden` attribute to `"true"`. Use this attribute to hide decorative content, duplicated content (repeated text), or collapsed content (menus).
  - [aria-label](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label): An `aria-label` attribute provides a descriptive, invisible label for an element. While visual appearance may be enough to signify an element's purpose to sighted users, `aria-label` can provide context when a visible label doesn't exist. However, use `aria-label` as a last resort - `<label>` or text referenced with `aria-labelledby` is preferred.
  - [Using ARIA: roles, states, and properties](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques)
- [Can I Use](https://caniuse.com)

## Markup

#### `role="grid"`

The `grid` role is used for a widget that contains one or more rows of cells. Some or all of the cells are focusable by using the keyboard, primarily the directional arrow keys. The `grid` role establishes a relationship between grouped elements - visually for those with vision and semantically for those using screen readers. See [Grid (W3C)](https://www.w3.org/TR/wai-aria-1.1/#grid) and [Grid Role (MDN)](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/Grid_Role).

The `gridcell` role is used to represent cells within a `grid` or `treegrid`. It's intended to mimic the behavior of `td` cells within a `table` element. In the accessibility tree, elements with `role="gridcell"` must be children of an element with `role="row"` which must, in turn, be children of an element with `role="grid"` or `role="treegrid"`. It's best not to have elements between these, but, if unavoidable, they must have `role="presentation"` which removes them from the accessibility tree.

```html
<fieldset>
  <legend id="customize_label">Choose up to 10 customization options</legend>
  <div role="grid" aria-labelledby="customize_label">
    <div role="row" data-row-index="0">
      <div class="checkbox" role="gridcell" tabindex="-1">
        <input type="checkbox" name="columns" tabindex="0" id="checkbox-1" />
        <label for="checkbox-1">Change $</label>
      </div>
      <div class="checkbox" role="gridcell" tabindex="-1">
        <input type="checkbox" name="columns" tabindex="-1" id="checkbox-2" />
        <label for="checkbox-2">Change %</label>
      </div>
      <div class="checkbox" role="gridcell" tabindex="-1">
        <input type="checkbox" name="columns" tabindex="-1" id="checkbox-3" />
        <label for="checkbox-3">Day's gain</label>
      </div>
    </div>
    <!-- ... -->
  </div>
</fieldset>
```

A component utilizing the `grid` role can use a [roving tabindex](https://www.w3.org/TR/wai-aria-practices/#kbd_roving_tabindex) to manage focus. In the example above, the first checkbox has a `tabindex="0"` while the rest have `tabindex="-1"` so anyone using a keyboard focuses on the first checkbox, then out of the component. To properly manage focus, a script would need to:

- listen for when arrow keys (or other navigational keys) have been pressed
- move focus to the active checkbox when pressing any navigational keys by giving it a `tabindex` of 0
- remove focus of the previously active checkbox by giving it a `tabindex` of -1
- move focus to the checkbox container and give it a `tabindex` of 0 in the case that 10 checkboxes are pre-selected and the first checkbox is disabled

See examples [here](https://www.w3.org/TR/wai-aria-practices-1.1/examples/grid/LayoutGrids.html).

### Attributes

#### `lang`

Programatically setting the language of a document and unique sections allows enhanced user software to translate content, correctly pronounce text, and provide definitions. See [lang](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/lang).

- Specify the general language of a page.
- Specify language changes within a page.

```html
<!-- DO -->
<!DOCTYPE html>
<html dir="ltr" lang="en-US">
  <head>
    <meta charset="UTF-8" />
    <title>Sample Document</title>
  </head>
  <body>
    <p>This paragraph is in English</p>
    <p lang="en-GB">This paragraph is defined as British English</p>
    <p lang="fr">Ce paragraphe est défini en français</p>
  </body>
</html>
```

#### `tabindex`

The `tabindex` attribute indicates an element, via keyboard navigation, can be allowed or prevented from being sequentially focusable or focused in a particular order. See [tabindex](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex).

- Write the document with elements in a logical sequence to avoid using `tabindex`.
- Do not apply `tabindex` to elements that are naturally focusable, such as buttons, links, inputs, etc.
- Avoid using positive integers since they alter focus order, making navigation confusing.
- It is unnecessary to add `tabindex` to pure HTML tables that don't have any interactivity.

| Value                          | Description                                                                                                                                                                                                                                  |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| null                           | The user agent will decide whether or not the element should be sequentially focusable or click focusable.                                                                                                                                   |
| -1 (or other negative integer) | Indicates that the element is not focusable by sequential keyboard navigation, but could be focused with JS or by clicking with the mouse. A negative value could be useful for content that is off screen, but appears on a specific event. |
| 0                              | This will cause the element to be both sequentially focusable and click focusable.                                                                                                                                                           |
| positive integer               | Behaves the same as 0, however, elements with higher values will come later. Not recommended as it changes the order of focusable elements and can provide users with confusing experience.                                                  |

### Best Practices

- When using `target="_blank"`. add `rel="noreferrer noopener"` to avoid exploitation of the `window.opener` API. See [Anchor element - security and privacy (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a#security_and_privacy) and [Target="\_blank" - the most underestimated vulnerability ever](https://www.jitbit.com/alexblog/256-targetblank--the-most-underestimated-vulnerability-ever).
- Add `role="list"` to `<ul>`'s affected by `list-style: none;`. This is a robust fix in reaction to a decision made by Apple, where any unordered list with `list-style: none;` applied to its list items is not treated as an unordered list semantically.
- Frequently asked questions (<abbr title="Frequently Asked Questions">FAQ</abbr>) should be used with `<abbr>` and is plural, so it doesn't require an "s" (FAQs).

### Correct Reading Sequence

Documents should have a logical reading flow. Whenever there is content with a meaningful sequence, that sequence should be refected in the DOM. It is recommended to separate content from layout, such that content sequence makes sense without styling and CSS handles positioning and layout. See [positioning content based on structural markup (W3C)](https://www.w3.org/WAI/WCAG21/Techniques/css/C6) and [meaningful sequence (WCAG)](https://www.w3.org/TR/WCAG21/#meaningful-sequence).

```html
<!-- DO -->
<form>
  <fieldset>
    <legend>How should we address you?</legend>
    <label for="name">Your nickname</label>
    <input autocomplete="nickname" id="name" name="nickname" type="text" />
  </fieldset>
  <fieldset>
    <!-- Note that this is not the first item in the form -->
    <legend>Next, pick a color for your profile</legend>
    <label for="color">Favorite color</label>
    <select id="color" name="colors">
      <option value="clear">clear</option>
      <option value="red">red</option>
      <option value="blue">blue</option>
      <option value="green">green</option>
    </select>
  </fieldset>
  <!-- The submit button should be the last item in the form -->
  <button type="submit">Submit</button>
</form>
```

### Headings

Headings describe the structure of content and its organization. Clear and descriptive headings make it easier to find information and understand the relationships between sections. Those using assistive technology may read them in an automatically-generated list and can jump to those sections.

- Use a `<header>` containing a `<h1>` to concisely summarize the page's topic or purpose. Use only one `<h1>` per page.
- Other headings `<h2>` to `<h6>` should describe the topic or purpose of a section. Don't skip heading levels.
- In terms of headings, treat modals as if they are a new page.
- Refer to these resources:
  - [WebAIM semantic structure headings](https://webaim.org/techniques/semanticstructure/#headings)
  - [Headings and labels - Level AA](https://www.w3.org/TR/WCAG21/#headings-and-labels)
  - [Info and relationships - Level A](https://www.w3.org/TR/WCAG21/#info-and-relationships)
  - [Page titled - Level A](https://www.w3.org/TR/WCAG21/#page-titled)

### Navigation

It is important everyone can navigate through website easily. They should feel confident in locating global controls, finding relevant pages, and quickly reaching main content.

- Style and location of common controls is consistent. Applies to navigations, settings, controls, and interactive elements. Order can only be changed if user-initiated. See [Consistent Navigation - Level AA](https://www.w3.org/TR/WCAG21/#consistent-navigation).
- Repeated multi-page content can be bypassed. A component or section repeated across pages (i.e. global header) should be skippable. See [details element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details), ["skip navigation" links](https://webaim.org/techniques/skipnav), disclosure widgets, and [Bypass Blocks - Level A](https://www.w3.org/TR/WCAG21/#bypass-blocks).
- Ensure global header is (mostly) the same on all pages. Items can be removed if not relevant on a page, but relative order should stay the same. Styling likely will change to indicate the current page, but it should still fit the established theme and serve a purpose.
- Be semantic. Semantic sectioning and headings (`<article>`, `<aside>`, `<footer>`, `<header>`, `<main>`, `<nav>`, `<section>`) allow everyone to quickly understand page structure by scanning, and those using assistive technology can jump to those sections.
- Include a search feature. Search is more appropriate if product size and complexity would make a sitemap too large. See [input element, type search](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/search). Between global navigation and search, all non-procedural pages can be reached in at least 2 ways ([Multiple Ways - Level AA](https://www.w3.org/TR/WCAG21/#multiple-ways)).
- Utilize links and show/hide features. "Skip to \[section\]" links allow everyone to get to specified sections. If appropriate, a section can be placed in a [disclosure widget](https://www.w3.org/TR/wai-aria-practices-1.1/#disclosure) or something similar.

### Semantics

It's important to ensure that information and relationships implied by visual or auditory formatting are preserved when presentation format changes. For example, when someone has a custom stylesheet and activates their browser's "reading mode" or engages with a screen reader. To achieve this, use semantic markup.

```html
<!-- DO -->
<header></header>
<time></time>

<!-- DON'T -->
<div class="header"></div>
<span>31 October 2017</span>
```

- [Semantic HTML5 Elements Explained](https://www.freecodecamp.org/news/semantic-html5-elements)
- [What On Earth Is Semantic Markup?](https://html.com/semantic-markup)
- [Semantics in HTML (MDN)](https://developer.mozilla.org/en-US/docs/Glossary/Semantics#semantics_in_html)
- [Semantic Structure <abbr title="Web Accessibility In Mind">(WebAIM)</abbr>](https://webaim.org/techniques/semanticstructure)
- [Info and Relationships - Level A](https://www.w3.org/TR/WCAG21/#info-and-relationships)

### Validation

To ensure that user agents, including assistive technologies, can parse content accurately and without crashing, conform to HTML specifications.

- `id` attribute values on a page are unique
- elements must have complete start and end tags (except self-closing elements), be correctly nested, and not contain duplicate attributes

## UI Components

### Consistent Functionality

Predictability allows us to more easily skim, search through, and understand content, requiring less guidance. When designing a component, consider its function and how that function will come to be known - it should be immediately identifiable. The cues should be unique enough that the component could be placed on any of our web pages and still be recognized. See [Consistent Identification - Level AA](https://www.w3.org/TR/WCAG21/#consistent-identification).

**DO** examples:

- Hyperlink
  - coded as `<a>`
  - contain text
  - have a different color than body copy
  - be underlined
  - change color when hovered or focused
- Activate a modal
  - coded as a `<button>`
  - contain concise text
  - have an outline, with whitespace between text and outline
  - change its background color when hovered or focused
- Download a document
  - have an icon representing documents
  - have alternative text beginning with the word "Download" followed by a short form of the document title

**DON'T** examples:

- Search for a term
  - some pages have the word "Search" as their submit button
  - other pages have the word "Find" on their submit button
- Refresh a chart icon
  - the icon for one chart has an `aria-label` value of "Refresh"
  - the icon for another chart has an `aria-label` value of "Renew chart"

### Identify Input purpose

Some people may not have a strong memory of their personal information, have difficulty typing, or require more details for their assistive technology to express an input's purpose. See [Identify Input Purpose - Level AA](https://www.w3.org/TR/WCAG21/#identify-input-purpose).

- Ensure every `<input>` has a `type` attribute. See [this](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input#input_types).
- Assume that `<input>`, `<select>`, and `<textarea>` can be autocompleted. See [this](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete). If there is no relevant option, then this attribute can be omitted.

```html
<!-- DO -->
<form>
  <label for="name_input">What is your preferred name?</label>
  <input
    autocomplete="nickname"
    id="name_input"
    name="user_name"
    placeholder="Name"
    type="text"
  />
  <label for="address_input">What is your address?</label>
  <textarea
    autocomplete="street-address"
    id="address_input"
    name="user_address"
    placeholder="32 Vassar Street - Room 32-G524"
  ></textarea>
  <label for="account_choice">Please choose an account</label>
  <select id="account_choice" name="user_account">
    <option value="checking">Checking</option>
    <option value="savings">Savings</option>
  </select>
</form>
```

### Name, Role, Value

Providing role, state, and value information on all UI components helps assistive technologies activate, set, gather information about, and keep up to date on the status of UI controls in the content. See [Name, Role, Value - Level A](https://www.w3.org/TR/WCAG21/#name-role-value).

- All user agents can read and edit the same metadata of an interface component, which may include role, name, state, property, value, notification of change
- Use semantic tags such as `<a>`, `<address>`, or `<button>`

```html
<!-- DO -->
<fieldset class="form-group">
  <legend>Example 1</legend>
  <label for="--disabled--">A general text field.</label>
  <input class="form-control" disabled id="--disabled--" type="text" />
</fieldset>

<fieldset>
  <legend>Example 2</legend>
  <label for="--password--">A general text field.</label>
  <input class="form-control" id="--password--" type="password" />
  <button class="btn btn-primary-outline" type="button">
    Forgot password?
  </button>
</fieldset>
```

### Tooltips best practices

- Only interactive elements should trigger tooltips
- Tooltips should directly describe the UI control that triggers them (i.e. do not create a control purely to trigger a tooltip)
- Use `aria-describedby` or `aria-labelledby` to associate the UI control with the tooltip. Avoid `aria-haspopup` and `aria-live`
- Do not use the `title` attribute to create a tooltip
- Do not put essential information in tooltips
- Provide a means to dismiss the tooltip with both keyboard and pointer
- Alow the mouse to easily move over the tooltip without dismissing it
- Do not use a timeout to hide the tooltip

## Color Contrast

People with low vision often have difficulty perceiving text and graphics with insufficient contrast. This can be exacerbated if the person has a color vision deficiency. Providing a minimum luminance contrast ratio can make these items more distinguishable, even if the person does not see the full range of colors.

- No requirement for decorative elements, inactive elements, or logotypes.
- Adjacent colors have a contrast ratio of at least 3 to 1 for large-scale text, interactive elements, or graphical elements with a distinct shape. See [Non-text Contrast](https://www.w3.org/TR/WCAG21/#non-text-contrast). See [Contrast (minimum)](https://www.w3.org/TR/WCAG21/#contrast-minimum).
- Adjacent colors have a contrast ratio of at least 4.5 to 1 for general text, general links, or graphical elements without a distinct shape.
- Tools: [WebAIM contrast checker](https://webaim.org/resources/contrastchecker/), [Who Can Use](https://whocanuse.com)

## Typography

### Size

People with visual disabilities may increase text size for easier reading. Scaling all content may be beneficial, but text is most critical. See [Resize text - Level AA](https://www.w3.org/TR/WCAG21/#resize-text).

- No consideration needed for decorative images of text
- Text can be resized upp to 200% (without loss of content or functionality) for text that we expect the user to read
- Use relative units (`em`) to ensure our content is well received within a variety of window sizes, zoom levels, and browser font settings. Set the font size of a page to `1rem`.
- [Guide to CSS units for relative spacing](https://dev.to/5t3ph/guide-to-css-units-for-relational-spacing-1mj5)
- [Font-relative lengths](https://www.w3.org/TR/css-values-4/#font-relative-lengths)
- [Relative length units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Relative_length_units)
- [Using named font sizes](https://www.w3.org/WAI/WCAG21/Techniques/css/C13.html)
- [Using percent for font sizes](https://www.w3.org/WAI/WCAG21/Techniques/css/C12.html)
- [Using em units for font sizes](https://www.w3.org/WAI/WCAG21/Techniques/css/C14.html)
- [Providing controls on web page that allow users to incrementally change size of all text on page up to 200 percent](https://www.w3.org/WAI/WCAG21/Techniques/general/G178.html)

### Spacing

Pages should be readable when typographic spacing is increased, allowing people with low vision, dyslexia, or cognitive disabilities to better discern characers and increase reading speed. See [Text Spacing - Level AA](https://www.w3.org/TR/WCAG21/#text-spacing)

- Copy should be visible and readable when:
  - Spacing following paragraphs is at least 2 times the font size
  - Line height is at least 1.5 times the font size
  - Word spacing is at least 0.16 times the font size
  - Letter spacing is at least 0.12 times the font size
- For best results, use relative units:
  - Set the font size of `<html>` to 100%
  - Set the font size of `<body>` in `rem`
  - Use `em` for text spacing such as padding, margin, etc.
  - Try to avoid setting heights, but if you need to, use relative units
- [Specifying the size of text containers using em units](https://www.w3.org/WAI/WCAG21/Techniques/css/C28.html)
- [Scaling form elements which contain text](https://www.w3.org/WAI/WCAG21/Techniques/css/C17.html)

## Glossary

| Term                           | Description                                                                                                                                                                                                         |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `<h1>` to `<h6>`               | Headings for sections.                                                                                                                                                                                              |
| `<header>`                     | A group of introductory or navigation aids. Not sectioning content.                                                                                                                                                 |
| `<input>`                      | Interactive controls for web-based forms in order to accept data from the user.                                                                                                                                     |
| `<select>`                     | A control that provides a menu of options.                                                                                                                                                                          |
| `<textarea>`                   | A multi-line, plain-text editing control, useful for allowing users to enter a sizeable amount of free-form text.                                                                                                   |
| Assistive technologies         | Hardwore and/or sofware that acts as a user agent, or along with a user agent, to provide functionality meeting the requirements of users with disabilities.                                                        |
| `autocomplete`                 | An HTML attribute that provides a hint for a form's autofill feature.                                                                                                                                               |
| Click focusable                | An element made focusable by clicking on it. Elements with a `tabindex` of 0 or more are considered click focusable by the user agent.                                                                              |
| DOM                            | An API for valid HTML and well-formed <abbr title="Extensible Markup Language">XML</abbr>. It defines the logical structure of documents and the way a document is accessed and manipulated.                        |
| Element (decorative)           | Content that does not provide necessary information, such as a subtle background pattern or an image that has visible text describing it.                                                                           |
| Element (graphical, semantic)  | Content that may not technically be text but is still expected to be read and understood, such as icons or lines on a graph.                                                                                        |
| Element (graphical, decorative | Any image that only functions as a style enhancer rather than providing information to the user.                                                                                                                    |
| Element(inactive)              | Interactive elements in a disabled state.                                                                                                                                                                           |
| Element (interactive)          | HTML elements that control a distinct function, such as buttons, checkboxes, date pickers, text fields, toggles, etc.                                                                                               |
| Focusable area                 | Refers to regions of the interface that can be focused by keyboard input, usually by the `tab` key.                                                                                                                 |
| Global-level                   | Code that affects or is within most pages.                                                                                                                                                                          |
| Interface component            | A part of the content that is perceived by users as a single control for a distinct function.                                                                                                                       |
| Large-scale text               | Text whose font size is at least 18px or at least 14px and bolded.                                                                                                                                                  |
| Link                           | HTML anchor element.                                                                                                                                                                                                |
| Logotype                       | A symbol or design that identifies a brand. While they don't have a contrast requirement, `alt` text should still be provided.                                                                                      |
| `name`                         | Text by which software can identify a component within web content to the user.                                                                                                                                     |
| Non-procedural                 | Pages that are not steps in a specific process.                                                                                                                                                                     |
| `role`                         | Text or number by which software can identify the function of a component within web content.                                                                                                                       |
| Semantic markup                | Selecting the correct HTML elements in order to convey the meaning of each part in a document. This is coupled with maintaining complete separation between the markup and the visual presentation of the elements. |
| Sequentially focusable         | Orders some or all of the focusable areas in the document relative to each other. A user can navigate a page sequentially using a keyboard by advancing focus from one element to the next.                         |
| Text / general text            | Text that will be read / that we expect to read.                                                                                                                                                                    |
| `type`                         | An HTML attribute that controls an input's data type.                                                                                                                                                               |
| User Agent                     | Any software that retrieves and presents web content, such as a web browser, screen magnifier, screen reader, or media player.                                                                                      |

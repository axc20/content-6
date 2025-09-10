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

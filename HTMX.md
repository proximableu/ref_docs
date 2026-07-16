# HTMX LLM Rules & Syntax Reference

When writing HTMX templates, adhere to these absolute rules of the Hypermedia-driven Architecture (HDA):

## Core Directives
1. ALWAYS respond from the server with clean HTML fragments (partials), NEVER JSON.
2. Favor declarative HTML attributes over Custom JavaScript wherever possible.
3. Use semantic HTML elements (buttons, forms, anchors) and progressive enhancement.

## Core Attribute Cheat Sheet
- hx-get="<url>": Issues a GET request to the target URL on the element's natural event.
- hx-post="<url>": Issues a POST request (excellent for submitting form datasets).
- hx-target="<CSS-selector>": Specifies the DOM element where the response HTML will be injected.
- hx-swap="<option>": Controls how the content is inserted. Options:
  - innerHTML (default): Replaces the children of the target.
  - outerHTML: Replaces the target element itself.
  - beforebegin / afterbegin / beforeend / afterend: Places relative to the target.
  - delete: Deletes the target element regardless of the response.
- hx-trigger="<event>": Modifies what triggers the request.
  - Modifiers: `changed` (only fire if input value changed), `delay:500ms` (debounces typing), `throttle:1s` (limits click spam).
  - From Selector: `hx-trigger="keyup[key=='Enter'] from:body"` (for keyboard shortcuts).
- hx-indicator="<CSS-selector>": Shows a loading spinner or thinking element by adding the `.htmx-request` class during transit.

## Out-of-Band (OOB) Swaps
To update multiple parts of a page from a single backend request, use the `hx-swap-oob="true"` attribute in your response fragment:
Example response:
<div id="target-main">Primary content update</div>
<div id="alert-box" hx-swap-oob="true" class="alert alert-success">Success!</div>
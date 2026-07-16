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

# HTMX Reference for LLM Code Generation

HTMX allows you to access AJAX, CSS Transitions, WebSockets, and Server Sent Events directly in HTML, using attributes. When generating HTMX code, use these core elements:

## Core Attributes
- hx-get="<url>": Issues a GET request to the specified URL.
- hx-post="<url>": Issues a POST request (sends parent form data if inside a <form>).
- hx-put="<url>": Issues a PUT request.
- hx-delete="<url>": Issues a DELETE request.
- hx-target="<CSS-selector>": Specifies the target element to swap the returned HTML into.
- hx-swap="<swap-strategy>": Controls how the response HTML is injected.
  - innerHTML (default): Replaces the content inside the target.
  - outerHTML: Replaces the entire target element.
  - beforebegin / afterbegin / beforeend / afterend: Places relative to target.
  - delete: Deletes the target element from the DOM.
  - none: Does not swap any content.

## Triggering Requests (hx-trigger)
By default, requests are triggered by the "natural" event of an element (change for select/input, submit for forms, click for others). Use `hx-trigger` to customize:
- hx-trigger="click": Standard click.
- hx-trigger="keyup delay:500ms changed": Debounces keyup inputs by 500ms; fires only if value changed.
- hx-trigger="load": Fires once when the element loads.
- hx-trigger="revealed": Fires when the element is scrolled into view (lazy loading).
- hx-trigger="every 5s": Polls the endpoint every 5 seconds.
- hx-trigger="custom-event from:body": Listens for a custom JS event dispatched to the document body.

## Out-of-Band (OOB) Swaps
To update multiple independent areas of a page from a single AJAX response, return the primary content normally, and append additional elements with `hx-swap-oob="true"` matched by ID.
Example response fragment:
<div id="main-content">New primary view</div>
<div id="status-message" hx-swap-oob="true" class="alert alert-success">Successfully saved!</div>

## Request Indicators
To show a spinner or "thinking" state during a request:
- Add class="htmx-indicator" to your spinner element (sets opacity to 0 by default).
- Add hx-indicator="#spinner-id" to the triggering element. HTMX will automatically add the `.htmx-request` class to the spinner during transit, making it visible.
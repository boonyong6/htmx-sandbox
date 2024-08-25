# introduction

- Supports AJAX, CSS Transitions, WebSockets and Server Sent Events using **attributes**.

# htmx in a Nutshell

- You can use the `data-` prefix with htmx:
  ```html
  <a data-hx-post="/click">Click Me!</a>
  ```

# AJAX

- The **core** of htmx is to issue AJAX requests.

## Triggering Requests - `hx-trigger`

- **(Default)** Triggered by the "natural" event:
  HTML Element | Event
  -------------|------
  `input`, `textarea` & `select` | `change`
  `form` | `submit`
  Everything else | `click`
- Use `hx-trigger` attribute to specify the event.

### Trigger Modifiers

- A trigger can have **modifiers** that change its behavior.
  Modifier | Description
  ---------|------------
  `once` | If you want a request to only happen once.
  `changed` | If the **value** of the element has changed.
  `delay:<time interval>`<br />(aka debounce) | If the event triggers again, the **countdown is reset**.<br />E.g. `delay:1s`, `delay:500ms`
  `throttle:<time interval>` | If a new event occurs before the time limit is hit the **event will be discarded**, so the request will **trigger at the end of the time period**.
  `from:<CSS Selector>` | Listen for the event **on a different element**.<br />**Note:** CSS selector is **not re-evaluated** if the page changes.<br />E.g. Can be used for things like **keyboard shortcuts**.
- Common UX pattern - [\*Active Search](https://htmx.org/examples/active-search/):
  ```html
  <input
    type="text"
    name="q"
    hx-get="/trigger_delay"
    hx-trigger="keyup changed delay:500ms"
    hx-target="#search-results"
    placeholder="Search..."
  />
  <div id="search-results"></div>
  ```
- **Multiple triggers** can be specified, **separated by commas**.
  ```html
  <div hx-get="/news" hx-trigger="load, click delay:1s"></div>
  ```

### Trigger Filters

- By using **`[]` after the event name**, enclosing a **JavaScript expression**.
  ```html
  <div hx-get="/clicked" hx-trigger="click[ctrlKey]">Control Click Me</div>
  ```
- Properties like `ctrlKey` will be resolved against the event first, then against the global scope.

### Special Events

Special Event | Description
--------------|------------
`load` | Fires once when first loaded.
`revealed` | Fires once when first **scrolls** into the viewport.
`intersect` | Fires once when first **intersects** the viewport.<br />Supports two additional options:<br />1. `root:<CSS Selector>` - The root element for intersection.<br />2. `threshold:<float>` - Between **0.0** and **1.0**, indicating what amount of intersection.

### Polling

- Use `every` syntax to poll the given URL.
  ```html
  <div hx-get="/news" hx-trigger="every 2s"></div>
  ```
- Respond with the HTTP response code `286` to **stop** polling.

### Load Polling

- Use a `load` trigger with a `delay`. (Assuming the `/messages` keeps returning a div set up this way.)
  ```html
  <div hx-get="/messages" hx-trigger="load delay:1s" hx-swap="outerHTML"></div>
  ```
- Useful where a poll has an **end**, e.g. [progress bar](https://htmx.org/examples/progress-bar/).

## Request Indicators - `hx-indicator`

- aka Loading spinners. [SVG spinners](http://samherbert.net/svg-loaders/).
- By using `htmx-indicator` class. The **opacity** of any element is 0 by default.
  ```html
  <!-- A spinner next to the button text. -->
  <button hx-get="/click">
    Click Me!
    <img class="htmx-indicator" src="/spinner.gif" />
  </button>
  ```
- When htmx issues a request, it put a `htmx-request` class onto the **requesting element** (default) or specified element (`hx-indicator`).
- To create your own **CSS transition**:
  ```css
  .htmx-indicator {
    display: none;
  }
  .htmx-request .htmx-indicator {
    display: inline;
  }
  .htmx-request.htmx-indicator {
    display: inline;
  }
  ```
- Use the `hx-indicator` attribute with a **CSS selector** to put a `htmx-request` class onto the specified element.
  ```html
  <div>
    <button hx-get="/click" hx-indicator="#indicator" hx-disabled-elt="this">
      Click Me!
    </button>
    <img id="indicator" class="htmx-indicator" src="/spinner.gif" />
  </div>
  ```
- Use the `hx-disabled-elt` attribute to disable elements for the duration of a request.

## Targets - `hx-target`

- Use `hx-target` attribute to load the response into a specified element.

### Extended CSS Selectors

- Most attributes that take a CSS selector, support an **"extended" CSS** syntax:
  Syntax | Description
  -------|------------
  `this` | The current element.
  `closest <CSS selector>` | Find the [closest](https://developer.mozilla.org/docs/Web/API/Element/closest) ancestor element or itself.
  `next <CSS selector>` | Find the next element.
  `previous <CSS selector>` | Find the previous element.
  `find <CSS selector>` | Find the first child descendent element.

## Swapping - `hx-swap`

- `hx-swap` attribute values:
  Name | Description
  -----|------------
  `innerHTML` | **Default**, puts the content **inside** the target.
  `outerHTML` | **Replaces** with the returned content.
  `afterbegin` | Prepends the content **before the first child** inside the target.
  `beforebegin` | Prepends the content **before the target**.
  `beforeend` | Appends the content **after the last child** inside the target.
  `afterend` | Appends the content **after the target**.
  `delete` | Deletes the target regardless of the response.
  `none` | Do nothing.

### Morph Swaps

- Supported via [**extensions**](https://htmx.org/docs/#morphing).
- To **merge** new content **by mutating existing nodes in-place**
- At the cost of more CPU.
- **Use case:** To preserve things like **focus**, video state, etc.

### View Transitions

- Built on the **experimental** *View Transitions API*.
- It is not available in all browsers.
- To experiment:
  - Set the `htmx.config.globalViewTransitions` variable to `true` to use transitions **for all swaps**.
  - Use `transition:true` option in `hx-swap` attribute.
  - You may catch the `htmx:beforeTransition` event and call `preventDefault()` to cancel the transition.
- [Example](https://htmx.org/examples/animations/#view-transitions)

### Swap Options

- By **default** htmx will swap in the **title tag** (`<title>`) found anywhere in the new content. To turn this off:
  ```html
  <button hx-post="/like" hx-swap="outerHTML ignoreTitle:true">Like</button>
  ```
- Available modifiers:
  Option | Description
  -------|------------
  `transition` | `true` or `false`, whether to use the view transition API.
  `swap` | **Swap delay** to use (e.g. `100ms`).
  `settle` | **Settle delay** to use (e.g `100ms`).
  `ignoreTitle` | Title found in the new content will be ignored.
  `scroll` | `top` or `bottom`, scroll the target to its top or bottom.
  `show` | `top` or `bottom`, scroll the target's top or bottom **into view**.
- Specified modifiers are **colon-separated**.
- [Documentation](https://htmx.org/attributes/hx-swap/)

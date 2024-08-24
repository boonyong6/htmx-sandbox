# introduction

- Supports AJAX, CSS Transitions, WebSockets and Server Sent Events using **attributes**.

# htmx in a Nutshell

- You can use the `data-` prefix with htmx:
  ```html
  <a data-hx-post="/click">Click Me!</a>
  ```

# AJAX

- The **core** of htmx is to issue AJAX requests.

## Triggering Requests

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
- Common UX pattern - [Active Search](https://htmx.org/examples/active-search/):
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
  <div hx-get="/clicked" hx-trigger="click[ctrlKey]">
    Control Click Me
  </div>
  ```
- Properties like `ctrlKey` will be resolved against the  event first, then against the global scope.

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
  <div hx-get="/messages"
    hx-trigger="load delay:1s"
    hx-swap="outerHTML"
  >
  </div>
  ```
- Useful where a poll has an **end**, e.g. [progress bar](https://htmx.org/examples/progress-bar/).

## Request Indicators

Continue...
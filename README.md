# introduction

- Supports AJAX, CSS Transitions, WebSockets and Server Sent Events using **attributes**.

# htmx in a Nutshell

- Can use the `data-` prefix with htmx:
  ```html
  <a data-hx-post="/click">Click Me!</a>
  ```

# AJAX

- The **core** of htmx is to issue AJAX requests.

## Triggering Requests

- **(Default)** Triggered by the "natural" event:
  Element | Event
  --------|------
  `input`, `textarea` & `select` | `change`
  `form` | `submit`
  Everything else | `click`
- Use `hx-trigger` attribute to specify the event.

## Trigger Modifier

- Continue...
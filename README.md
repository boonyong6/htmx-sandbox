# Introduction

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
- Use the `hx-trigger` attribute to specify the event.

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

- Use the `every` syntax to poll the given URL.
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

- Use the `hx-target` attribute to load the response into a specified element.

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
  - Use the `transition:true` option in `hx-swap` attribute.
  - You may catch the `htmx:beforeTransition` event and call `preventDefault()` to cancel the transition.
- [Animation Examples - View Transitions](https://htmx.org/examples/animations/#view-transitions)

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
- [Documentation - `hx-swap`](https://htmx.org/attributes/hx-swap/)

## Synchronization - `hx-sync`

- To coordinate requests between two elements. E.g.: 
  - To make a request from one element to substitute the request of another element.
  - To wait until the other element's request has finished.
- Example - `[1]` Watch for requests on the form and abort the input's request if a form request is present or starts:
  ```html
  <!-- Declarative way -->
  <form hx-post="/store">
    <input id="title" name="title" type="text"
      hx-post="/validate"
      hx-trigger="change"
      hx-sync="closest form:abort"> <!-- [1] -->
    <button type="submit">Submit</button>
  </form>
  ```
- Example - Send the `htmx:abort` event to an element to cancel any in-flight/in-progress requests:
  ```html
  <!-- Programmatic way -->
  <button id="request-button" hx-post="/example">
    Issue Request
  </button>
  <button onclick="htmx.trigger('#request-button', 'htmx:abort')">
    Cancel Request
  </button>
  ```
- [Documentation - `hx-sync`](https://htmx.org/attributes/hx-sync/)

## CSS Transitions (without JavaScript)

- When a new element with the **same `id`** is swapped in.
  ```html
  <!-- Original -->
  <div id="div1">Original Content</div>

  <!-- New -->
  <div id="div1" class="red">New Content</div>
  ```
- `CSS` example:
  ```css
  .red {
    color: red;
    transition: all ease-in 1s;
  }
  ```
- [Animation Examples](https://htmx.org/examples/animations/)

### Details

- How CSS transitions works without `JavaScript` in `htmx`:
  1. Before swap, find existing elements that match the `id`.
  2. If matched, copy old element's attributes onto the new element.
  3. Swap in the new content with the old attribute values.
  4. Swap in new attribute values after a `settle` delay.

## Out of Band Swaps - `hx-swap-oob`

- Use the `id` attribute to swap content directly into the DOM.
- In the **response** html:
  ```html
  <div id="message" hx-swap-oob="true">Swap me directly!</div>
  Additional Content <!-- This would be swapped into the target. -->
  ```
- **Use case:** To "piggy-back" updates on other requests.

### Selecting Content To Swap

Attribute | Description
----------|------------
`hx-select` | Specify a **CSS selector**.<br />E.g. `hx-select=".intro"`
`hx-select-oob` | Specify a list of **element IDs**.<br />E.g. `hx-select-oob="#alert:afterbegin, #info"`

### Preserving Content During A Swap - `hx-preserve`

- Value: `true` or `false`.

## Parameters

Attribute | Description
----------|------------
[`hx-include`](https://htmx.org/attributes/hx-include/) | To include values of other elements.
[`hx-params`](https://htmx.org/attributes/hx-params/) | To filter out parameters.

Event | Description
------|------------
[`htmx:configRequest`](https://htmx.org/events/#htmx:configRequest) | To programmatically modify the parameters.<br />E.g. To add **Authorization** request header.

### File Upload

- Set the `hx-encoding` attribute to `multipart/form-data` (the usual encoding is `application/x-www-form-urlencoded`).
- `htmx:xhr:progress` event is triggered periodically during upload, which you can hook into to show the progress.
- E.g. [progress bars](https://htmx.org/examples/file-upload/) and [error handling](https://htmx.org/examples/file-upload-input/).
- [More advanced form patterns](https://htmx.org/examples/)

### Extra Values

- Use the `hx-vals` attribute to include extra values in a request.
  ```html
  <div hx-get="/example" hx-vals='{"myVal": "My Value"}'>JSON literal</div>

  <!-- Use javascript: or js: to enable JavaScript evaluation. Beware of XSS. -->
  <div hx-get="/example" hx-vals='js:{myVal: calculateValue()}'>Dynamic value</div>
  ```

## Confirming Requests

- Use the `hx-confirm` attribute to confirm an action using a **simple javascript dialog**.
  ```html
  <button hx-delete="/account" hx-confirm="Are you sure?">
    Delete Account
  </button>
  ```
- [More sophisticated confirmation dialog](https://htmx.org/examples/confirm/)

### Alternative: Using Events - `htmx:confirm`

```js
document.body.addEventListener('htmx:confirm', function(evt) {
  if (evt.target.matches("[confirm-with-sweet-alert='true']")) {
    evt.preventDefault();
    // sweet alert - https://sweetalert.js.org/guides/
    swal({
      title: "Are you sure?",
      text: "Are you sure you are sure?",
      icon: "warning",
      buttons: true,
      dangerMode: true,
    }).then((confirmed) => {
      if (confirmed) {
        evt.detail.issueRequest();
      }
    });
  }
});
```

# Attribute Inheritance

- Most attributes are inherited (apply to any children elements).
  ```html
  <div hx-confirm="Are you sure?">
    <button hx-delete="/account">
      Delete My Account
    </button>
    <button hx-put="/account">
      Update My Account
    </button>
    <!-- To undo the inheritance. -->
    <button hx-confirm="unset" hx-get="/">
      Cancel
    </button>
  </div>
  ```
- Set the [`hx-disinherit`](https://htmx.org/attributes/hx-disinherit/) attribute on a **parent node** to **disable** inheritance.
- Set the `htmx.config.disableInheritance` variable to `true` to disable inheritance entirely/globally. Use the `hx-inherit` attribute to specify inheritance explicitly.
  ```html
  <!-- Configure htmx -->
  <meta name="htmx-config" content='{"disableInheritance": true}'>

  <div hx-target="#tab-container" hx-inherit="hx-target">
    <a hx-boost="true" href="/tab1">Tab 1</a>
    <a hx-boost="true" href="/tab2">Tab 2</a>
    <a hx-boost="true" href="/tab3">Tab 3</a>
  </div>
  ```

# Boosting - `hx-boost`

- Convert `<a>` and `<form>` into AJAX requests that, by default, target the `<body>`.
  ```html
  <div hx-boost="true">
    <a href="/blog">Blog</a>
  </div>
  ```

## Progressive Enhancement

- `<a>` and `<form>` will fallback to the original behavior if the JavaScript is not enabled.
- The server side can examine the `HX-Request` header to identify a htmx-driven request.

# Web Sockets & SSE

- Supported via extensions
  - [SSE extension](https://github.com/bigskysoftware/htmx-extensions/blob/main/src/sse/README.md)
  - [WebSocket extension](https://github.com/bigskysoftware/htmx-extensions/blob/main/src/ws/README.md)

# History Support - [browser history API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)

- To push the **request URL** into the **browser navigation bar** and add the current state of the page to the **browser's history** (aka history snapshot).
  ```html
  <a hx-get="/blog" hx-push-url="true">Blog</a>
  ```
- **NOTE:** If you push a URL into the history, you **must** be able to navigate to that URL and get a full page back.

## Undoing DOM Mutation By 3rd Party Libraries

- You will need to clean up the DOM before a snapshot is taken, since some of the 3rd party libraries will be reinitialized when the history content is loaded.

## Disabling History Snapshots

- **Use case:** To prevent sensitive data entering the `localStorage` cache (History navigation will still work).

# Requests & Responses

- Return `204 - No Content` response code, and htmx will ignore the content of the response.

Event | Description
------|------------
`htmx:responseError` | Fired when the server returns a `4xx` or `5xx` error response.
`htmx:sendError` | Fired when connection error.

## Configuring Response Handling

- Configure the `htmx.config.responseHandling` array to change the default response handling (status code).
  ```js
  responseHandling: [
    // 204 - No Content by default does nothing, but is not an error.
    {code:"204", swap: false}, 
    // 2xx & 3xx responses are non-errors and are swapped.
    {code:"[23]..", swap: true},
    // 4xx & 5xx responses are not swapped and are errors.
    {code:"[45]..", swap: false, error: true}, 
    // Catch all for any other response code.
    {code:"...", swap: false} 
  ]

  // Configuration fields: code, swap, error, ignoreTitle, select, target, swapOverride 
  ```
- Use the [Response Targets](https://github.com/bigskysoftware/htmx-extensions/blob/main/src/response-targets/README.md) extensions to configure the behavior of response codes declaratively via **attributes**.

## CORS

- **Access-Control headers** to configure on the server in the CORS context:
  - `Access-Control-Allow-Headers`
  - `Access-Control-Expose-Headers`
- [**All** request and response headers that htmx implements.](https://htmx.org/reference/#request_headers)

## Request Headers

- Some noteworthy headers:
  Header | Description
  -------|------------
  `HX-Trigger-Name` | Triggered element `name` if it exists.
  `HX-Trigger` | Triggered element `id` if it exists.

## Response Headers

- Some noteworthy headers:
  Header | Description
  -------|------------
  `HX-Location` | Do a **redirect** that does not do a full page reload.
  [`HX-Trigger[-After-(Settle\|Swap)]`](https://htmx.org/headers/hx-trigger/) | To trigger client-side events [after the (settle\|swap) step]

# Validation

- Events to hook to implement **custom validation** and **error handling**.
  Event | Description
  ------|------------
  `htmx:validation:validate` | Fired **before** an element's `checkValidity()`.<br />**Use case:** To add custom validation.
  `htmx:validation:failed` | Fired when `checkValidity()` returns false (invalid input).
  `htmx:validation:halted` | Fired when a request is not issued due to validation errors (`event.detail.errors`).
- Add `hx-validate="true"` to `<input>`, `<textarea>` or `<select>` enables validation before sending requests.

# Extensions

- Use the `hx-ext` attribute to use extensions.

# Events & Logging

- [Events mechanism](https://htmx.org/reference/#events) doubles as the logging system.
- `htmx:load` event is fired every time an element is loaded into the DOM.

## Use Case 1: Initialize A 3rd Party Library With Events

- `htmx.onLoad()` - A helper method for `htmx:load` event.
  ```js
  htmx.onLoad(function(target) {
    myJavascriptLib.init(target);
  });
  ```

## Use Case 2: Configure a Request With Events

- Handle the `htmx:configRequest` event to modify an AJAX request.
  ```js
  document.body.addEventListener('htmx:configRequest', function(evt) {
    // Add a new parameter into the request.
    evt.detail.parameters['auth_token'] = getAuthToken();
    // Add a new header into the request.
    evt.detail.headers['Authentication-Token'] = getAuthToken();
  })
  ```

## Use Case 3: Modifying Swapping Behavior With Events

- Handle the `htmx:beforeSwap` event to modify the swap behavior.
  ```js
  document.body.addEventListener('htmx:beforeSwap', function (evt) {
    if (evt.detail.xhr.status === 404) {
      alert('Error: Could Not Find Resource');
    } else if (evt.detail.xhr.status === 422) {
      evt.detail.shouldSwap = true;
      evt.detail.isError = false;
    } else if (evt.detail.xhr.status === 418) {
      evt.detail.shouldSwap = true;
      evt.detail.target = htmx.find('#teapot');
    }
  });
  ```

## Event Naming

- Camel Case - `htmx:afterSwap`
- Kebab Case - `htmx:after-swap`

## Logging

- To log `every` event.
  ```js
  htmx.logger = function (elt, eventName, eventDetail) {
    if (console) {
      console.log(eventName, elt, eventDetail);
    }
  }
  ```

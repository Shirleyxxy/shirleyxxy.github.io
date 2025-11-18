---
title: "Fallstack Part 0.b: Fundamentals of Web Apps"
---

## Fundamentals of Web Apps

### HTTP GET

The server and the web browser communicate with each other using the HTTP protocol. 
- Inspect --> `Network` tab --> `Headers`
    - `General`: request URL, request method, status code
    - `Response Headers`: content length (bytes), content type (e.g. `text/html; charset=utf-8`)

![Sequence diagram for browser-server communication](/assets/images/fso-part0-sequence-diagram.png)

The sequence diagram visualizes how the browser and server are communicating over the time. The time flows in the diagram from top to bottom, so the diagram starts with the first request that the browser sends to server, followed by the response.

First, the browser sends an HTTP GET request to the server to fetch the HTML code of the page. **The img tag in the HTML prompts the browser to fetch the image kuva.png.** The browser renders the HTML page and the image to the screen.

Even though it is difficult to notice, the HTML page begins to render before the image has been fetched from the server.


### Running application logic in the browser
- `clear()` to empty the console
- When you go to the `notes` page, the browser does 4 HTTP requests:
    - HTML `document`
    - CSS `stylesheet`
    - `main.js` JavaScript `script`
    - `data.json` raw data (`xhr`, i.e. `XMLHttpRequest`)


### Event handlers and callback functions
Event handler functions are called callback functions. The application code does not invoke the functions itself, but the runtime environment - the browser - invokes the function at an appropriate time when the event has occurred.


### Document Object Model (DOM)




### Readings
- [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [GET request method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/GET)
- [Status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
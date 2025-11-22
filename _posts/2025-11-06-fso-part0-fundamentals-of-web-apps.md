---
title: "Fallstack Part 0.b: Fundamentals of Web Apps"
date: 2025-11-22
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
The functioning of the browser is based on the idea of depicting HTML elements as a tree. Document Object Model, or DOM, is an Application Programming Interface (API) that enables programmatic modification of the element trees corresponding to web pages.
The topmost node of the DOM tree of an HTML document is called the `document` object. We can perform various operations on a webpage using the DOM-API. You can access the document object by typing `document` into the Console tab.

Manipulating the document object from console:

```javascript
list = document.getElementsByTagName('ul')[0]
newElement = document.createElement('li')
newElement.textContent = 'testing the page manipulation from console'
list.appendChild(newElement)
```
Even though the page updates on your browser, the changes are not permanent. If the page is reloaded, the new note will disappear, because the changes were not pushed to the server.


### Readings
- [HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [GET request method](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods/GET)
- [Status code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
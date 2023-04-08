## JavaScript

### General Info

- Universal browser langauge
- Not "Java" but similar syntax
- Event-based structure
- Most modern apps written in JavaScript
- Not just for UI widgets

### What its good for

- Read/Write to/from Document Object Model (DOM)
- Read cookies/headers
- Detect keystrokes & mouse clicks
- Take screenshots
- Fetch more content over HTTP (think AJAX)
- Open web sockets
- and more

### HTTP Traffic

#### Request

- Originates from browser
- Variety of requests
  - Type of request (GET/POST/etc)
  - Requested file path
  - User-Agent
  - Cookies
  - Etc
- May include a body

#### Response

- Responds to the browser's given request
- Status code
- Includes a variety of headers
  - Date
  - Set-Cookie
  - Server
  - Etc
- Often includes a body

#### Keeping Users Logged in

- Session cookies - can be edited by user
- Other browser managed auth
- Application managed auth

#### Client Persistence APIs

- Cookies
- LocalStorage
- SessionStorage
- IndexedDB


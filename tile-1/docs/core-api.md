# Core HTTP API

The core HTTP API provides the main request function and convenience methods for making HTTP requests with callbacks.

## Capabilities

### Main Request Function

Makes an HTTP request with a callback-based interface.

```javascript { .api }
/**
 * Makes an HTTP request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options (if uri is a string)
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance (extends Stream)
 */
function request(uri, options, callback);
```

**Parameters:**

- `uri` - Can be:
  - String: URL to request
  - Object: Options object (see Configuration)
- `options` - Optional configuration object with request settings
- `callback` - Optional callback function with signature: `(error, response, body)`
  - `error` - Error object if request failed, null otherwise
  - `response` - HTTP response object (IncomingMessage with additional properties)
  - `body` - Response body as string or Buffer

**Returns:** Request instance (Stream) that can be piped or listened to for events

**Usage Examples:**

```javascript
const request = require('request');

// Simple GET with callback
request('http://www.example.com', function (error, response, body) {
  if (!error && response.statusCode == 200) {
    console.log(body);
  }
});

// With options object
request({
  uri: 'http://api.example.com/data',
  method: 'GET',
  headers: {
    'User-Agent': 'my-app'
  }
}, function (error, response, body) {
  console.log('Status:', response.statusCode);
  console.log('Body:', body);
});

// Options as first parameter
request({
  uri: 'http://www.example.com',
  method: 'POST',
  json: { key: 'value' }
}, function (error, response, body) {
  console.log('Response:', body);
});
```

### GET Requests

Makes an HTTP GET request.

```javascript { .api }
/**
 * Makes an HTTP GET request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance
 */
function request.get(uri, options, callback);
```

**Usage Examples:**

```javascript
// Simple GET
request.get('http://www.example.com', function (error, response, body) {
  console.log(body);
});

// GET with query parameters
request.get({
  uri: 'http://api.example.com/users',
  qs: { page: 1, limit: 10 }
}, function (error, response, body) {
  console.log(body);
});

// GET with headers
request.get({
  uri: 'http://api.example.com/data',
  headers: {
    'Authorization': 'Bearer token123',
    'Accept': 'application/json'
  }
}, function (error, response, body) {
  console.log(body);
});
```

### POST Requests

Makes an HTTP POST request.

```javascript { .api }
/**
 * Makes an HTTP POST request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance
 */
function request.post(uri, options, callback);
```

**Usage Examples:**

```javascript
// POST with JSON body
request.post({
  uri: 'http://api.example.com/users',
  json: {
    name: 'John Doe',
    email: 'john@example.com'
  }
}, function (error, response, body) {
  console.log('Created:', body);
});

// POST with form data
request.post('http://service.com/upload', {
  form: { key: 'value', name: 'file' }
}, function (error, response, body) {
  console.log(body);
});

// POST with body string
request.post({
  uri: 'http://api.example.com/data',
  body: 'raw body string',
  headers: {
    'Content-Type': 'text/plain'
  }
}, function (error, response, body) {
  console.log(body);
});
```

### PUT Requests

Makes an HTTP PUT request.

```javascript { .api }
/**
 * Makes an HTTP PUT request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance
 */
function request.put(uri, options, callback);
```

**Usage Examples:**

```javascript
// PUT with JSON body
request.put({
  uri: 'http://api.example.com/users/123',
  json: {
    name: 'Jane Doe',
    email: 'jane@example.com'
  }
}, function (error, response, body) {
  console.log('Updated:', body);
});

// PUT to upload file
const fs = require('fs');
fs.createReadStream('file.json').pipe(
  request.put('http://mysite.com/obj.json')
);
```

### PATCH Requests

Makes an HTTP PATCH request.

```javascript { .api }
/**
 * Makes an HTTP PATCH request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance
 */
function request.patch(uri, options, callback);
```

**Usage Examples:**

```javascript
// PATCH to partially update resource
request.patch({
  uri: 'http://api.example.com/users/123',
  json: {
    email: 'newemail@example.com'
  }
}, function (error, response, body) {
  console.log('Patched:', body);
});
```

### DELETE Requests

Makes an HTTP DELETE request.

```javascript { .api }
/**
 * Makes an HTTP DELETE request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance
 */
function request.del(uri, options, callback);
function request.delete(uri, options, callback);
```

**Note:** Both `request.del` and `request.delete` are aliases for the same function.

**Usage Examples:**

```javascript
// DELETE request
request.del('http://api.example.com/users/123', function (error, response, body) {
  console.log('Deleted:', response.statusCode);
});

// DELETE with authorization
request.delete({
  uri: 'http://api.example.com/items/456',
  headers: {
    'Authorization': 'Bearer token123'
  }
}, function (error, response, body) {
  console.log('Status:', response.statusCode);
});
```

### HEAD Requests

Makes an HTTP HEAD request (retrieves headers only, no body).

```javascript { .api }
/**
 * Makes an HTTP HEAD request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance
 */
function request.head(uri, options, callback);
```

**Note:** HEAD requests MUST NOT include a request body.

**Usage Examples:**

```javascript
// Check if resource exists
request.head('http://example.com/file.pdf', function (error, response) {
  if (!error && response.statusCode == 200) {
    console.log('File exists');
    console.log('Content-Type:', response.headers['content-type']);
    console.log('Content-Length:', response.headers['content-length']);
  }
});

// Check last modified date
request.head('http://api.example.com/data', function (error, response) {
  console.log('Last-Modified:', response.headers['last-modified']);
  console.log('ETag:', response.headers['etag']);
});
```

## Request Instance

All request functions return a Request instance which:

- Extends Node.js Stream (readable and writable)
- Can be used with callbacks (as shown above)
- Can be used with streams (see Streaming documentation)
- Emits events ('response', 'data', 'end', 'error', 'complete')

## Callback Behavior

The callback receives three arguments:

1. **error**: Error object if request failed, `null` if successful
   - Network errors (connection refused, timeout)
   - DNS resolution errors
   - SSL/TLS errors (if strictSSL is true)
   - HTTP status codes >= 400 are NOT treated as errors

2. **response**: HTTP response object (http.IncomingMessage)
   - `response.statusCode` - HTTP status code
   - `response.headers` - Response headers
   - `response.caseless` - Case-insensitive header access
   - `response.request` - The originating request object
   - `response.elapsedTime` - Request duration (if time option enabled)
   - `response.toJSON()` - JSON representation

3. **body**: Response body
   - String (if encoding is specified, default: 'utf8')
   - Buffer (if encoding is null)
   - Parsed JSON (if json option is true)

## Error-Only Responses

To check only for errors without buffering the response body, omit the callback and listen for the 'error' event:

```javascript
request('http://example.com')
  .on('error', function(err) {
    console.error('Error:', err);
  })
  .on('response', function(response) {
    console.log('Status:', response.statusCode);
    // Response body is not buffered
  });
```

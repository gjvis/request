# Request

Request is a simplified HTTP client for Node.js that provides an intuitive API for making HTTP/HTTPS requests. It supports streaming, forms, authentication, proxies, redirects, cookies, and comprehensive HTTP features with both callback-based and stream-based interfaces.

## Package Information

- **Package Name**: request
- **Package Type**: npm
- **Language**: JavaScript (Node.js)
- **Installation**: `npm install request`

## Core Imports

```javascript
const request = require('request');
```

Access HTTP methods via the request object:

```javascript
const request = require('request');

// Use methods: request.get, request.post, request.put, etc.
request.get('http://example.com', callback);
request.post('http://example.com', callback);
```

## Basic Usage

```javascript
const request = require('request');

// Simple GET request with callback
request('http://www.example.com', function (error, response, body) {
  if (!error && response.statusCode == 200) {
    console.log(body);
  }
});

// POST request with form data
request.post('http://service.com/upload', {
  form: { key: 'value' }
}, function(err, response, body) {
  console.log(body);
});

// Streaming response to file
const fs = require('fs');
request('http://example.com/file.png')
  .pipe(fs.createWriteStream('file.png'));
```

## Architecture

Request is built around several key components:

- **Function API**: Main `request()` function and HTTP method shortcuts (get, post, put, etc.)
- **Request Class**: Stream-based class (extends Node.js Stream) that handles all HTTP operations
- **Factory Functions**: `defaults()` and `forever()` for creating configured request instances
- **Cookie Management**: Cookie jar system for handling HTTP cookies across requests
- **Authentication System**: Support for multiple auth methods (Basic, Digest, Bearer, OAuth, AWS, Hawk, HTTP Signature)
- **Form Handling**: Built-in support for URL-encoded and multipart form data
- **Streaming Interface**: Full Node.js stream compatibility for request and response piping

## Capabilities

### Core HTTP API

The main request function and HTTP method convenience functions for making requests with callbacks.

```javascript { .api }
/**
 * Makes an HTTP request
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback with signature (error, response, body)
 * @returns {Request} Request instance (Stream)
 */
function request(uri, options, callback);

/**
 * Makes an HTTP GET request
 */
function request.get(uri, options, callback);

/**
 * Makes an HTTP POST request
 */
function request.post(uri, options, callback);

/**
 * Makes an HTTP PUT request
 */
function request.put(uri, options, callback);

/**
 * Makes an HTTP PATCH request
 */
function request.patch(uri, options, callback);

/**
 * Makes an HTTP DELETE request
 */
function request.del(uri, options, callback);
function request.delete(uri, options, callback);

/**
 * Makes an HTTP HEAD request
 */
function request.head(uri, options, callback);
```

[Core HTTP API](./core-api.md)

### Streaming

Stream-based interface for piping requests and responses, enabling efficient data transfer without buffering.

```javascript { .api }
/**
 * Request class extends Node.js Stream
 */
class Request extends Stream {
  /**
   * Pipes the response to a destination stream
   * @param {Stream} dest - Destination stream
   * @param {object} [opts] - Pipe options
   * @returns {Stream} Destination stream
   */
  pipe(dest, opts);

  /**
   * Writes data to the request body
   */
  write(...args);

  /**
   * Ends the request, optionally writing final chunk
   * @param {string|Buffer} [chunk] - Final data to write
   */
  end(chunk);

  /**
   * Pauses reading from the response stream
   */
  pause();

  /**
   * Resumes reading from the response stream
   */
  resume();

  /**
   * Aborts the ongoing request
   */
  abort();

  /**
   * Destroys the request stream
   */
  destroy();
}
```

[Streaming](./streaming.md)

### Forms

Support for URL-encoded forms and multipart/form-data uploads.

```javascript { .api }
/**
 * Sets URL-encoded form data
 * @param {object|FormData} form - Form data
 * @returns {FormData} FormData instance when called without arguments
 */
Request.prototype.form(form);

/**
 * Sets multipart/form-data request body
 * @param {Array} multipart - Array of multipart sections
 */
Request.prototype.multipart(multipart);
```

[Forms](./forms.md)

### Authentication

Comprehensive authentication support including Basic, Digest, Bearer token, OAuth 1.0, AWS signatures, Hawk, and HTTP Signature.

```javascript { .api }
/**
 * Sets HTTP authentication
 * @param {string|object} user - Username or options object
 * @param {string} [pass] - Password
 * @param {boolean} [sendImmediately] - Send credentials without waiting for 401
 * @param {string} [bearer] - Bearer token string
 */
Request.prototype.auth(user, pass, sendImmediately, bearer);

/**
 * Signs request with AWS signature
 * @param {object} opts - AWS signing options
 * @param {Date} [now] - Optional timestamp
 */
Request.prototype.aws(opts, now);

/**
 * Signs request with OAuth 1.0
 * @param {object} _oauth - OAuth configuration object
 */
Request.prototype.oauth(_oauth);

/**
 * Signs request with Hawk authentication
 * @param {object} opts - Hawk options
 */
Request.prototype.hawk(opts);

/**
 * Signs request using HTTP Signature
 * @param {object} opts - HTTP signature options
 */
Request.prototype.httpSignature(opts);
```

[Authentication](./authentication.md)

### Configuration and Options

Factory functions for creating configured request instances, and comprehensive request options.

```javascript { .api }
/**
 * Creates a request wrapper with default options
 * @param {object|function} options - Default options or custom requester function
 * @param {function} [requester] - Optional custom request function
 * @returns {function} Request function with defaults applied
 */
function request.defaults(options, requester);

/**
 * Creates a request function with forever agent (connection pooling)
 * @param {object} [agentOptions] - Options for the forever agent
 * @param {object} [optionsArg] - Additional request options
 * @returns {function} Request function with forever agent
 */
function request.forever(agentOptions, optionsArg);
```

[Configuration and Options](./configuration.md)

### Cookie Management

Cookie jar system for managing HTTP cookies across requests.

```javascript { .api }
/**
 * Creates a new cookie jar
 * @param {object} [store] - Optional custom cookie store
 * @returns {RequestJar} Cookie jar instance
 */
function request.jar(store);

/**
 * Parses a cookie string
 * @param {string} str - Cookie string to parse
 * @returns {Cookie} Parsed Cookie object
 */
function request.cookie(str);

/**
 * Cookie jar class for managing cookies
 */
class RequestJar {
  /**
   * Sets a cookie in the jar
   * @param {string|Cookie} cookieOrStr - Cookie string or Cookie object
   * @param {string} uri - URI for the cookie
   * @param {object} [options] - Cookie options
   */
  setCookie(cookieOrStr, uri, options);

  /**
   * Gets cookies for a URI as a string
   * @param {string} uri - URI to get cookies for
   * @returns {string} Cookie string
   */
  getCookieString(uri);

  /**
   * Gets cookies for a URI as an array
   * @param {string} uri - URI to get cookies for
   * @returns {Array} Array of Cookie objects
   */
  getCookies(uri);
}
```

[Cookie Management](./cookies.md)

### Advanced Exports

Additional exports for advanced usage and type checking.

```javascript { .api }
/**
 * The Request class constructor (exported for instanceof checks and advanced usage)
 */
const Request = request.Request;

/**
 * Internal parameter normalization function (exported for advanced usage)
 * @param {string|object} uri - URL string or options object
 * @param {object} [options] - Request options
 * @param {function} [callback] - Callback function
 * @returns {object} Normalized parameters object
 */
function request.initParams(uri, options, callback);
```

**Usage Examples:**

```javascript
const request = require('request');

// Check if an object is a Request instance
const req = request('http://example.com');
console.log(req instanceof request.Request); // true

// Access Request class directly
const Request = request.Request;

// Use initParams for parameter normalization (advanced)
const params = request.initParams('http://example.com', { method: 'GET' });
```

## Events

The Request instance emits several events:

- `'response'` - Emitted when response is received (response object)
- `'data'` - Emitted when response data chunk is available (chunk)
- `'end'` - Emitted when response ends
- `'error'` - Emitted on errors (error object)
- `'complete'` - Emitted when request completes successfully (response, body)
- `'pipe'` - Standard stream pipe event
- `'drain'` - Standard stream drain event
- `'close'` - Emitted when connection closes

## Response Object

The response object is Node.js `http.IncomingMessage` with additional properties:

```javascript { .api }
interface Response extends http.IncomingMessage {
  /** HTTP status code */
  statusCode: number;

  /** Response headers */
  headers: object;

  /** Response body (when callback is used) */
  body: string | Buffer;

  /** The originating request object */
  request: Request;

  /** Case-insensitive header access */
  caseless: object;

  /** Request duration in milliseconds (if time option enabled) */
  elapsedTime?: number;

  /** JSON representation of response */
  toJSON(): object;
}
```

## Error Handling

Errors are passed to the callback or emitted via the 'error' event:

```javascript
// Callback-based error handling
request('http://example.com', function(error, response, body) {
  if (error) {
    console.error('Request failed:', error);
    return;
  }
  // Process response
});

// Stream-based error handling
request('http://example.com')
  .on('error', function(err) {
    console.error('Request failed:', err);
  })
  .pipe(destination);
```

Common error scenarios:
- Network errors (connection refused, timeout, DNS resolution)
- SSL/TLS certificate errors (when `strictSSL: true`)
- HTTP errors (status codes >= 400 are NOT treated as errors by default)
- Invalid options or configuration

## Types

```javascript { .api }
/**
 * Request options object
 */
interface RequestOptions {
  /** Request URL */
  uri?: string;
  url?: string;

  /** HTTP method */
  method?: string;

  /** Base URL for relative requests */
  baseUrl?: string;

  /** HTTP headers */
  headers?: object;

  /** Query string object */
  qs?: object;

  /** Request body */
  body?: string | Buffer | Stream;

  /** URL-encoded form data */
  form?: object;

  /** Multipart form data */
  formData?: object;

  /** JSON body (also sets Content-Type) */
  json?: any;

  /** Authentication credentials */
  auth?: {
    user?: string;
    username?: string;
    pass?: string;
    password?: string;
    sendImmediately?: boolean;
    bearer?: string;
  };

  /** OAuth 1.0 options */
  oauth?: object;

  /** Follow redirects */
  followRedirect?: boolean | Function;

  /** Maximum redirects */
  maxRedirects?: number;

  /** Response encoding (null for binary) */
  encoding?: string | null;

  /** Enable gzip compression */
  gzip?: boolean;

  /** Cookie jar */
  jar?: RequestJar | boolean;

  /** Request timeout in milliseconds */
  timeout?: number;

  /** Proxy URL */
  proxy?: string;

  /** Require valid SSL certificates */
  strictSSL?: boolean;

  /** Connection pool */
  pool?: object;

  /** Local network interface */
  localAddress?: string;

  /** Unix domain socket path */
  socketPath?: string;

  /** Set Host header explicitly */
  setHost?: boolean;

  /** Disable automatic cookie handling */
  _disableCookies?: boolean;

  /** Measure request duration */
  time?: boolean;

  /** HTTP Archive format request */
  har?: object;

  /** Response callback */
  callback?: Function;
}
```

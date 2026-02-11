# HTTP Methods and Options

Comprehensive HTTP request methods and configuration options for the request package.

## Capabilities

### Main Request Function

The primary function for making HTTP requests with flexible parameter signatures.

```javascript { .api }
/**
 * Make an HTTP request
 * @param uri - Target URL (string or options object)
 * @param options - Request configuration
 * @param callback - Callback function(error, response, body)
 * @returns Request instance
 */
function request(uri: string, options?: object, callback?: function): Request;
function request(uri: string, callback?: function): Request;
function request(options: object, callback?: function): Request;
function request(options: object): Request;
```

**Usage Examples:**

```javascript
// URL only
request('http://www.google.com');

// URL with callback
request('http://www.google.com', function (error, response, body) {
  console.log('statusCode:', response.statusCode);
  console.log('body:', body);
});

// URL with options and callback
request('http://www.google.com', {
  headers: {'User-Agent': 'my-app'}
}, function (error, response, body) {
  // ...
});

// Options object
request({
  uri: 'http://www.google.com',
  method: 'POST',
  json: {key: 'value'}
}, function (error, response, body) {
  // ...
});
```

### HTTP Verb Methods

Convenience methods that default to specific HTTP methods.

```javascript { .api }
/**
 * Make a GET request
 * @param uri - Target URL
 * @param options - Request configuration
 * @param callback - Callback function
 * @returns Request instance
 */
function request.get(uri: string, options?: object, callback?: function): Request;

/**
 * Make a POST request
 * @param uri - Target URL
 * @param options - Request configuration
 * @param callback - Callback function
 * @returns Request instance
 */
function request.post(uri: string, options?: object, callback?: function): Request;

/**
 * Make a PUT request
 * @param uri - Target URL
 * @param options - Request configuration
 * @param callback - Callback function
 * @returns Request instance
 */
function request.put(uri: string, options?: object, callback?: function): Request;

/**
 * Make a PATCH request
 * @param uri - Target URL
 * @param options - Request configuration
 * @param callback - Callback function
 * @returns Request instance
 */
function request.patch(uri: string, options?: object, callback?: function): Request;

/**
 * Make a HEAD request
 * @param uri - Target URL
 * @param options - Request configuration
 * @param callback - Callback function
 * @returns Request instance
 */
function request.head(uri: string, options?: object, callback?: function): Request;

/**
 * Make a DELETE request (alias)
 * @param uri - Target URL
 * @param options - Request configuration
 * @param callback - Callback function
 * @returns Request instance
 */
function request.del(uri: string, options?: object, callback?: function): Request;

/**
 * Make a DELETE request
 * @param uri - Target URL
 * @param options - Request configuration
 * @param callback - Callback function
 * @returns Request instance
 */
function request.delete(uri: string, options?: object, callback?: function): Request;
```

**Usage Examples:**

```javascript
// Simple GET
request.get('http://www.google.com');

// POST with form data
request.post('http://service.com/upload', {
  form: {key: 'value'}
}, callback);

// PUT with JSON
request.put({
  url: 'http://service.com/resource/123',
  json: {name: 'Updated Name'}
}, callback);

// PATCH
request.patch('http://service.com/resource/123', {
  json: {status: 'active'}
}, callback);

// HEAD request
request.head('http://www.google.com', function (error, response) {
  console.log('headers:', response.headers);
});

// DELETE
request.delete('http://service.com/resource/123', callback);
```

### URL and Header Options

Configure the request URL, method, and headers.

```javascript { .api }
/**
 * URL configuration options
 */
interface URLOptions {
  /** Target URL (required if not provided as first parameter) */
  uri?: string;
  /** Alias for uri */
  url?: string;
  /** Base URL for relative URIs */
  baseUrl?: string;
  /** HTTP method (default: 'GET') */
  method?: string;
  /** Custom HTTP headers */
  headers?: object;
}
```

**Usage Examples:**

```javascript
// Custom headers
request({
  url: 'https://api.github.com/repos/request/request',
  headers: {
    'User-Agent': 'request'
  }
}, callback);

// Base URL
request({
  baseUrl: 'https://api.example.com',
  uri: '/users/123'
  // Full URL will be: https://api.example.com/users/123
}, callback);

// Explicit method
request({
  uri: 'http://service.com/resource',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  }
}, callback);
```

### Query String Options

Configure query string parameters and parsing.

```javascript { .api }
/**
 * Query string configuration
 */
interface QueryStringOptions {
  /** Query string parameters as object */
  qs?: object;
  /** Options for qs.parse() */
  qsParseOptions?: object;
  /** Options for qs.stringify() */
  qsStringifyOptions?: object;
  /** Use querystring module instead of qs (default: false) */
  useQuerystring?: boolean;
}
```

**Usage Examples:**

```javascript
// Add query parameters
request({
  uri: 'http://service.com/search',
  qs: {
    q: 'search term',
    page: 1,
    limit: 10
  }
  // URL becomes: http://service.com/search?q=search%20term&page=1&limit=10
}, callback);

// Custom stringify options
request({
  uri: 'http://service.com/api',
  qs: {
    filters: ['active', 'verified']
  },
  qsStringifyOptions: {
    arrayFormat: 'repeat'  // filters=active&filters=verified
  }
}, callback);

// Use querystring module
request({
  uri: 'http://service.com/api',
  qs: {
    tags: ['foo', 'bar']
  },
  useQuerystring: true  // tags=foo&tags=bar instead of tags[0]=foo&tags[1]=bar
}, callback);
```

### Request Body Options

Configure the request body with various formats.

```javascript { .api }
/**
 * Request body configuration
 */
interface BodyOptions {
  /** Raw body (string, Buffer, Stream, or Array) */
  body?: string | Buffer | Stream | Array<any>;
  /** Enable JSON mode and optionally set body */
  json?: boolean | object;
  /** Custom JSON reviver for parsing response */
  jsonReviver?: function;
  /** Custom JSON replacer for stringifying body */
  jsonReplacer?: function;
}
```

**Usage Examples:**

```javascript
// String body
request.post({
  uri: 'http://service.com/api',
  body: 'Hello World',
  headers: {
    'Content-Type': 'text/plain'
  }
}, callback);

// Buffer body
const buf = Buffer.from('binary data');
request.post({
  uri: 'http://service.com/upload',
  body: buf
}, callback);

// JSON object
request.post({
  uri: 'http://service.com/api',
  json: {
    name: 'John Doe',
    age: 30
  }
  // Sets Content-Type: application/json and stringifies body
}, callback);

// Enable JSON parsing only
request.get({
  uri: 'http://service.com/api/data',
  json: true
  // Response body will be automatically parsed as JSON
}, callback);

// Stream body
const fs = require('fs');
request.put({
  uri: 'http://service.com/upload',
  body: fs.createReadStream('file.json')
}, callback);
```

### HTTP Agent and Pooling

Configure connection pooling and HTTP agents.

```javascript { .api }
/**
 * HTTP agent configuration
 */
interface AgentOptions {
  /** HTTP agent instance or false to disable */
  agent?: object | false;
  /** Custom agent class */
  agentClass?: function;
  /** Options for agent creation */
  agentOptions?: object;
  /** Use forever-agent for keep-alive connections */
  forever?: boolean;
  /** Connection pool object */
  pool?: object | false;
}
```

**Usage Examples:**

```javascript
const http = require('http');

// Custom agent
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 50
});

request({
  uri: 'http://service.com/api',
  agent: agent
}, callback);

// Agent options
request({
  uri: 'http://service.com/api',
  agentOptions: {
    keepAlive: true,
    maxSockets: 100
  }
}, callback);

// Forever agent (keep-alive)
request({
  uri: 'http://service.com/api',
  forever: true
}, callback);

// Custom pool
const pool = {maxSockets: Infinity};
request({
  uri: 'http://service.com/api',
  pool: pool
}, callback);

// Disable pooling
request({
  uri: 'http://service.com/api',
  pool: false
}, callback);
```

### Other Request Options

```javascript { .api }
/**
 * Additional request options
 */
interface OtherOptions {
  /** Local address to bind for network connections */
  localAddress?: string;
  /** Enable request timing (sets response.elapsedTime) */
  time?: boolean;
  /** HAR 1.2 Request Object */
  har?: object;
  /** Alternative to callback parameter */
  callback?: function;
  /** Custom HTTP/HTTPS modules */
  httpModules?: object;
}
```

**Usage Examples:**

```javascript
// Bind to specific local address
request({
  uri: 'http://service.com/api',
  localAddress: '192.168.1.100'
}, callback);

// Enable timing
request({
  uri: 'http://service.com/api',
  time: true
}, function (error, response, body) {
  console.log('Request took:', response.elapsedTime, 'ms');
});

// HAR format
request({
  uri: 'http://www.google.com',  // will be ignored
  har: {
    url: 'http://www.mockbin.com/har',
    method: 'POST',
    headers: [
      {name: 'content-type', value: 'application/x-www-form-urlencoded'}
    ],
    postData: {
      mimeType: 'application/x-www-form-urlencoded',
      params: [
        {name: 'foo', value: 'bar'},
        {name: 'hello', value: 'world'}
      ]
    }
  }
}, callback);
```

### Unix Domain Sockets

Make requests to Unix domain sockets using a special URL scheme.

```javascript { .api }
/**
 * Unix domain socket URL format:
 * http://unix:SOCKET:PATH
 * where SOCKET is absolute path to socket file
 */
```

**Usage Example:**

```javascript
// Request to Unix socket
request.get('http://unix:/absolute/path/to/unix.socket:/request/path');
```

### Chainable Configuration Methods

Request instances support chainable methods for configuring requests fluently.

```javascript { .api }
/**
 * Add or modify query string parameters
 * @param q - Query string parameters object
 * @param clobber - If true, replace existing qs, otherwise extend (default: false)
 * @returns this
 */
interface Request {
  qs(q: object, clobber?: boolean): Request;
}

/**
 * Enable JSON mode or set JSON body
 * @param val - true to parse response, false to disable, or object to set as body
 * @returns this
 */
interface Request {
  json(val: boolean | object): Request;
}
```

**Usage Examples:**

```javascript
// Chainable query string
request.get('http://api.example.com/search')
  .qs({q: 'search term', page: 1})
  .on('response', function(response) {
    console.log('Search complete');
  });

// Chainable JSON
request.post('http://api.example.com/users')
  .json({name: 'John', age: 30})
  .on('response', function(response) {
    console.log('User created');
  });

// Chain multiple configurations
request.get('http://api.example.com/data')
  .qs({filter: 'active'})
  .json(true)
  .on('complete', function(response, body) {
    console.log('Data:', body);
  });
```

## Types

### Request Instance

```javascript { .api }
/**
 * Request instance returned by all request methods
 * Extends Node.js Stream
 */
interface Request extends Stream {
  // Stream methods
  pipe(dest: Stream, opts?: object): Stream;
  write(chunk: string | Buffer, encoding?: string, callback?: function): boolean;
  end(chunk?: string | Buffer): void;
  pause(): void;
  resume(): void;
  abort(): void;

  // Configuration methods (chainable)
  qs(q: object, clobber?: boolean): Request;
  form(form?: object | string): Request | FormData;
  multipart(multipart: Array<object> | object): Request;
  json(val: boolean | object): Request;
  jar(jar: boolean | CookieJar): Request;
  auth(user: string, pass: string, sendImmediately?: boolean, bearer?: string): Request;
  oauth(oauth: object): Request;
  hawk(opts: object): Request;
  aws(opts: object, now?: boolean): Request;
  httpSignature(opts: object): Request;

  // Header methods
  getHeader(name: string, headers?: object): string | undefined;
  setHeader(name: string, value: string): void;
  hasHeader(name: string): boolean;
  removeHeader(name: string): void;

  // Properties
  uri: object;
  method: string;
  headers: object;
  body: any;
  response?: Response;
}
```

### Response Object

```javascript { .api }
/**
 * HTTP response object (extends http.IncomingMessage)
 */
interface Response {
  statusCode: number;
  statusMessage: string;
  headers: object;
  body?: string | Buffer | object;
  request: Request;
  elapsedTime?: number;
  toJSON(): ResponseJSON;
}

interface ResponseJSON {
  statusCode: number;
  body: any;
  headers: object;
  request: RequestJSON;
}

interface RequestJSON {
  uri: object;
  method: string;
  headers: object;
}
```

### Callback Function

```javascript { .api }
/**
 * Request callback function
 * @param error - Error object or null
 * @param response - HTTP response object
 * @param body - Response body (string, Buffer, or parsed JSON)
 */
type RequestCallback = (
  error: Error | null,
  response: Response,
  body: string | Buffer | object
) => void;
```

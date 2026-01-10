# Configuration and Options

Request provides extensive configuration options and factory functions for creating preconfigured request instances.

## Capabilities

### Defaults Factory

Creates a request wrapper with default options applied to every request.

```javascript { .api }
/**
 * Creates a request wrapper with default options
 * @param {object|function} options - Default options object or custom requester function
 * @param {function} [requester] - Optional custom request function
 * @returns {function} Request function with defaults applied (includes all HTTP verb methods)
 */
function request.defaults(options, requester);
```

**Usage Examples:**

```javascript
const request = require('request');

// Create request instance with default headers
const githubRequest = request.defaults({
  baseUrl: 'https://api.github.com',
  headers: {
    'User-Agent': 'my-app',
    'Accept': 'application/vnd.github.v3+json'
  },
  json: true
});

// All requests use the defaults
githubRequest.get('/users/octocat', function(err, response, body) {
  console.log('User:', body.name);
});

githubRequest.get('/repos/nodejs/node', function(err, response, body) {
  console.log('Repo:', body.full_name);
});

// Create authenticated request instance
const authenticatedRequest = request.defaults({
  headers: {
    'Authorization': 'Bearer token123'
  }
});

authenticatedRequest.get('http://api.example.com/protected', function(err, res, body) {
  console.log(body);
});

// Create request with timeout default
const timeoutRequest = request.defaults({
  timeout: 5000, // 5 seconds
  headers: {
    'User-Agent': 'my-bot'
  }
});

timeoutRequest('http://slow-api.example.com', function(err, res, body) {
  if (err) {
    console.error('Request timed out or failed');
    return;
  }
  console.log(body);
});
```

**Custom Requester Function:**

```javascript
// Custom requester for request transformation
function customRequester(options, callback) {
  // Transform options before request
  options.headers = options.headers || {};
  options.headers['X-Custom-Header'] = 'value';

  console.log('Making request to:', options.uri);

  // Make the actual request
  return request(options, callback);
}

const customRequest = request.defaults({}, customRequester);

customRequest('http://example.com', function(err, response, body) {
  console.log('Custom request completed');
});
```

**Nested Defaults:**

```javascript
// Create base request with common options
const baseRequest = request.defaults({
  headers: {
    'User-Agent': 'my-app'
  },
  timeout: 10000
});

// Create more specific request from base
const apiRequest = baseRequest.defaults({
  baseUrl: 'http://api.example.com',
  json: true
});

apiRequest.get('/users', function(err, res, body) {
  console.log('Users:', body);
});
```

### Forever Agent Factory

Creates a request function with a forever agent for connection pooling and reuse.

```javascript { .api }
/**
 * Creates a request function with forever agent (connection pooling)
 * @param {object} [agentOptions] - Options for the forever agent
 * @param {object} [optionsArg] - Additional request options
 * @returns {function} Request function with forever agent
 */
function request.forever(agentOptions, optionsArg);
```

**Usage Examples:**

```javascript
const request = require('request');

// Create request with forever agent
const foreverRequest = request.forever();

// Connections are reused across requests
foreverRequest('http://api.example.com/endpoint1', function(err, res, body) {
  console.log('Request 1');
});

foreverRequest('http://api.example.com/endpoint2', function(err, res, body) {
  console.log('Request 2 (reused connection)');
});

// With agent options
const customForeverRequest = request.forever({
  maxSockets: 50,
  minSockets: 10
}, {
  timeout: 5000
});

customForeverRequest('http://api.example.com/data', function(err, res, body) {
  console.log(body);
});
```

## Complete Request Options

Comprehensive list of all available request options:

```javascript { .api }
interface RequestOptions {
  // URL and Method
  /** Request URL */
  uri?: string;
  url?: string;

  /** HTTP method (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS) */
  method?: string;

  /** Base URL for relative requests */
  baseUrl?: string;

  // Headers
  /** HTTP headers object */
  headers?: object;

  // Request Body
  /** Raw request body (string, Buffer, or Stream) */
  body?: string | Buffer | Stream;

  /** URL-encoded form data */
  form?: object;

  /** Multipart form data */
  formData?: object;

  /** Multipart sections array */
  multipart?: Array<object>;

  /** JSON body (automatically stringified, sets Content-Type) */
  json?: any | boolean;

  // Query String
  /** Query string parameters object */
  qs?: object;

  /** Use querystring module instead of qs */
  useQuerystring?: boolean;

  /** Options for qs.stringify */
  qsStringifyOptions?: object;

  /** Options for qs.parse */
  qsParseOptions?: object;

  // Authentication
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

  /** AWS signature options */
  aws?: object;

  /** Hawk authentication options */
  hawk?: object;

  /** HTTP Signature options */
  httpSignature?: object;

  // Redirects
  /** Follow redirects (boolean or custom function) */
  followRedirect?: boolean | Function;

  /** Follow non-GET redirects */
  followAllRedirects?: boolean;

  /** Maximum number of redirects (default: 10) */
  maxRedirects?: number;

  /** Remove Referer header on redirect */
  removeRefererHeader?: boolean;

  // Response Handling
  /** Response encoding (string for encoding, null for binary Buffer) */
  encoding?: string | null;

  /** Enable automatic gzip decompression */
  gzip?: boolean;

  // Cookies
  /** Cookie jar instance or true for global jar */
  jar?: RequestJar | boolean;

  /** Disable automatic cookie handling */
  _disableCookies?: boolean;

  // Timeouts and Retries
  /** Request timeout in milliseconds */
  timeout?: number;

  // Connection and Network
  /** Local interface to bind to */
  localAddress?: string;

  /** Unix domain socket path (e.g., 'http://unix:/path/to/socket:/endpoint') */
  socketPath?: string;

  /** Proxy server URL */
  proxy?: string;

  /** Use HTTP tunneling for proxy */
  tunnel?: boolean;

  /** Require valid SSL certificates (default: true) */
  strictSSL?: boolean;

  /** Connection pool configuration */
  pool?: object;

  /** Use forever agent for connection pooling */
  forever?: boolean;

  /** Options for HTTP agent */
  agentOptions?: object;

  /** Custom agent class */
  agentClass?: Function;

  /** Custom agent instance */
  agent?: Agent;

  // Advanced Options
  /** Enable timing information */
  time?: boolean;

  /** HTTP Archive (HAR) format request definition */
  har?: object;

  /** Add CRLF before multipart boundary */
  preambleCRLF?: boolean;

  /** Add CRLF after multipart boundary */
  postambleCRLF?: boolean;

  /** Set Host header explicitly */
  setHost?: boolean;

  // Callback
  /** Response callback function */
  callback?: Function;
}
```

## Common Configuration Patterns

### Base URL Configuration

```javascript
// Create API client with base URL
const apiClient = request.defaults({
  baseUrl: 'https://api.example.com/v1',
  headers: {
    'User-Agent': 'my-app/1.0'
  },
  json: true
});

// Relative URLs are appended to baseUrl
apiClient.get('/users', callback); // https://api.example.com/v1/users
apiClient.post('/users', { json: { name: 'John' } }, callback);
```

### Timeout Configuration

```javascript
// Set timeout for all requests
const timeoutRequest = request.defaults({
  timeout: 10000 // 10 seconds
});

// Or per-request
request({
  uri: 'http://slow-api.example.com',
  timeout: 5000
}, callback);
```

### Proxy Configuration

```javascript
// HTTP proxy
request({
  uri: 'http://api.example.com/data',
  proxy: 'http://proxy-server:8080'
}, callback);

// HTTPS proxy
request({
  uri: 'https://api.example.com/data',
  proxy: 'https://secure-proxy:8443'
}, callback);

// Authenticated proxy
request({
  uri: 'http://api.example.com/data',
  proxy: 'http://username:password@proxy:8080'
}, callback);

// Proxy from environment variable
// Automatically uses HTTP_PROXY, HTTPS_PROXY, NO_PROXY environment variables
request('http://api.example.com/data', callback);
```

### SSL/TLS Configuration

```javascript
// Disable SSL certificate validation (not recommended for production)
request({
  uri: 'https://self-signed.example.com',
  strictSSL: false
}, callback);

// Custom agent with SSL options
const https = require('https');
const agent = new https.Agent({
  rejectUnauthorized: true,
  ca: fs.readFileSync('ca-cert.pem'),
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem')
});

request({
  uri: 'https://secure-api.example.com',
  agent: agent
}, callback);

// With agentOptions
request({
  uri: 'https://api.example.com',
  agentOptions: {
    ca: fs.readFileSync('ca-cert.pem'),
    rejectUnauthorized: true
  }
}, callback);
```

### Connection Pooling

```javascript
// Use forever agent for connection reuse
const request = require('request').forever();

// Make multiple requests (connections are reused)
for (let i = 0; i < 100; i++) {
  request('http://api.example.com/item/' + i, callback);
}

// Custom pool configuration
request({
  uri: 'http://api.example.com/data',
  pool: {
    maxSockets: 100
  }
}, callback);
```

### Redirect Configuration

```javascript
// Follow redirects (default behavior)
request({
  uri: 'http://example.com/redirect',
  followRedirect: true,
  maxRedirects: 10
}, callback);

// Don't follow redirects
request({
  uri: 'http://example.com/redirect',
  followRedirect: false
}, callback);

// Custom redirect handling
request({
  uri: 'http://example.com/redirect',
  followRedirect: function(response) {
    // Return true to follow, false to stop
    return response.statusCode >= 300 && response.statusCode < 400;
  }
}, callback);

// Follow all redirects (including POST redirects)
request({
  uri: 'http://example.com/redirect',
  followAllRedirects: true
}, callback);
```

### Encoding Configuration

```javascript
// UTF-8 encoding (default)
request('http://example.com/text', callback);

// Binary data (Buffer)
request({
  uri: 'http://example.com/image.png',
  encoding: null
}, function(err, response, body) {
  // body is a Buffer
  fs.writeFileSync('image.png', body);
});

// Specific encoding
request({
  uri: 'http://example.com/data',
  encoding: 'latin1'
}, callback);
```

### GZIP Compression

```javascript
// Enable automatic gzip decompression
request({
  uri: 'http://api.example.com/data',
  gzip: true
}, callback);

// Request will send 'Accept-Encoding: gzip, deflate'
// and automatically decompress the response
```

### Timing Information

```javascript
// Enable request timing
request({
  uri: 'http://api.example.com/data',
  time: true
}, function(err, response, body) {
  if (!err) {
    console.log('Request took:', response.elapsedTime, 'ms');
    console.log('Response time breakdown:', response.timings);
  }
});
```

### HAR (HTTP Archive) Format

```javascript
// Make request from HAR format
request({
  har: {
    url: 'http://api.example.com/data',
    method: 'POST',
    headers: [
      { name: 'Content-Type', value: 'application/json' }
    ],
    postData: {
      mimeType: 'application/json',
      text: JSON.stringify({ key: 'value' })
    }
  }
}, callback);
```

## Environment Variables

Request automatically uses these environment variables:

- `HTTP_PROXY` / `http_proxy` - HTTP proxy URL
- `HTTPS_PROXY` / `https_proxy` - HTTPS proxy URL
- `NO_PROXY` / `no_proxy` - Comma-separated list of hosts to bypass proxy
- `NODE_TLS_REJECT_UNAUTHORIZED` - Set to '0' to disable SSL verification (not recommended)
- `NODE_DEBUG=request` - Enable debug logging

**Example:**

```bash
export HTTP_PROXY=http://proxy:8080
export NO_PROXY=localhost,127.0.0.1
export NODE_DEBUG=request
node app.js
```

## Debug Mode

Enable debug logging:

```javascript
// Enable globally
request.debug = true;

// Or via environment variable
// NODE_DEBUG=request node app.js

// Check if debug is enabled
if (request.debug) {
  console.log('Debug mode is on');
}
```

## Performance Optimization

### Connection Pooling

```javascript
// Reuse connections with forever agent
const request = require('request').forever({
  maxSockets: 50,
  minSockets: 10
});

// Or with pool option
const pooledRequest = request.defaults({
  pool: {
    maxSockets: 100
  }
});
```

### Keep-Alive

```javascript
// Enable keep-alive with custom agent
const http = require('http');
const keepAliveAgent = new http.Agent({
  keepAlive: true,
  maxSockets: 50
});

request({
  uri: 'http://api.example.com/data',
  agent: keepAliveAgent
}, callback);
```

### Timeouts

```javascript
// Set appropriate timeouts to avoid hanging
request({
  uri: 'http://api.example.com/data',
  timeout: 5000, // 5 seconds
  pool: { maxSockets: 100 }
}, callback);
```

## Unix Domain Sockets

```javascript
// Connect via Unix domain socket
request({
  uri: 'http://unix:/path/to/socket:/endpoint'
}, callback);

// Or with explicit socket path
request({
  uri: 'http://endpoint',
  socketPath: '/path/to/socket'
}, callback);
```

## Custom Agent

```javascript
const http = require('http');
const https = require('https');

// HTTP agent with custom options
const httpAgent = new http.Agent({
  keepAlive: true,
  maxSockets: 50,
  maxFreeSockets: 10,
  timeout: 30000
});

// HTTPS agent with custom options
const httpsAgent = new https.Agent({
  keepAlive: true,
  maxSockets: 50,
  rejectUnauthorized: true,
  ca: [fs.readFileSync('ca-cert.pem')]
});

// Use custom agents
request({
  uri: 'https://api.example.com/data',
  agent: httpsAgent
}, callback);

// Or with agentClass
request({
  uri: 'http://api.example.com/data',
  agentClass: http.Agent,
  agentOptions: {
    keepAlive: true,
    maxSockets: 50
  }
}, callback);
```

### Static Properties and Global Configuration

Global static properties for configuring request behavior across all instances.

```javascript { .api }
/**
 * Global debug flag for enabling debug logging
 * @type {boolean}
 */
request.Request.debug = false;

/**
 * Default headers to forward through proxy
 * @type {Array<string>}
 */
request.Request.defaultProxyHeaderWhiteList = [
  'accept', 'accept-charset', 'accept-encoding', 'accept-language',
  'accept-ranges', 'cache-control', 'content-encoding', 'content-language',
  'content-length', 'content-location', 'content-md5', 'content-range',
  'content-type', 'connection', 'date', 'expect', 'max-forwards',
  'pragma', 'referer', 'te', 'user-agent', 'via'
];

/**
 * Headers that are exclusive to proxy requests
 * @type {Array<string>}
 */
request.Request.defaultProxyHeaderExclusiveList = [
  'proxy-authorization'
];
```

**Usage Examples:**

```javascript
const request = require('request');

// Enable debug mode globally
request.Request.debug = true;

// Or use the convenience property
request.debug = true;

// All requests will now output debug information
request('http://example.com', callback);

// Customize proxy header whitelist
request.Request.defaultProxyHeaderWhiteList.push('x-custom-header');

// Check proxy exclusive headers
console.log(request.Request.defaultProxyHeaderExclusiveList);
```

### Utility Methods

Utility methods for advanced request handling and debugging.

```javascript { .api }
/**
 * Gets a header value with case-insensitive lookup
 * @param {string} name - Header name to lookup
 * @param {object} [headers] - Optional headers object to search (defaults to request headers)
 * @returns {string|undefined} Header value or undefined if not found
 */
Request.prototype.getHeader(name, headers);

/**
 * Enables Unix domain socket support for the request
 * Modifies the request to use Unix sockets based on URI
 */
Request.prototype.enableUnixSocket();

/**
 * Returns a JSON representation of the request
 * Useful for logging and debugging
 * @returns {object} JSON object containing uri, method, and headers
 */
Request.prototype.toJSON();

/**
 * Outputs debug information if debugging is enabled
 * Called internally but available for custom debugging
 */
Request.prototype.debug();
```

**Usage Examples:**

```javascript
const request = require('request');

// Get header with case-insensitive lookup
const req = request('http://example.com');
req.setHeader('Content-Type', 'application/json');
console.log(req.getHeader('content-type')); // 'application/json'
console.log(req.getHeader('CONTENT-TYPE')); // 'application/json'

// Unix domain sockets
request({
  uri: 'http://unix:/path/to/socket:/endpoint',
}, function(err, response, body) {
  console.log(body);
});

// Get JSON representation for logging
const req2 = request({
  uri: 'http://api.example.com/data',
  method: 'POST',
  headers: { 'Authorization': 'Bearer token' }
});

console.log(JSON.stringify(req2.toJSON(), null, 2));
// Output:
// {
//   "uri": "http://api.example.com/data",
//   "method": "POST",
//   "headers": { "Authorization": "Bearer token" }
// }

// Enable debug for specific request
request.Request.debug = true;
const req3 = request('http://example.com');
req3.debug(); // Outputs debug information
```

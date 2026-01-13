# Request Options

The request package accepts a comprehensive options object to configure all aspects of HTTP requests. Options can be passed to the main `request()` function or any HTTP method convenience function.

## Capabilities

### Complete Options Interface

```javascript { .api }
interface RequestOptions {
  // URI/URL Options
  uri: string | URL;
  url?: string | URL;
  baseUrl?: string;

  // HTTP Method
  method?: string;

  // Headers
  headers?: object;

  // Query String
  qs?: object;
  qsParseOptions?: object;
  qsStringifyOptions?: object;
  useQuerystring?: boolean;

  // Request Body
  body?: string | Buffer | Stream;
  form?: object;
  formData?: object;
  multipart?: array | object;
  json?: boolean | any;
  jsonReplacer?: function;
  jsonReviver?: function;

  // Multipart Specific
  preambleCRLF?: boolean;
  postambleCRLF?: boolean;

  // Authentication (see Authentication doc for details)
  auth?: object;
  oauth?: object;
  hawk?: object;
  aws?: object;
  httpSignature?: object;

  // SSL/TLS (see SSL/TLS doc for details)
  ca?: string | Buffer | string[];
  cert?: string | Buffer;
  key?: string | Buffer;
  pfx?: string | Buffer;
  passphrase?: string;
  strictSSL?: boolean;
  rejectUnauthorized?: boolean;
  ciphers?: string;
  secureProtocol?: string;
  secureOptions?: number;

  // Agent/Connection
  agent?: http.Agent | https.Agent;
  agentClass?: class;
  agentOptions?: object;
  forever?: boolean;
  pool?: object;
  timeout?: number;
  localAddress?: string;

  // Redirect Handling (see Redirects and Proxies doc for details)
  followRedirect?: boolean | function;
  followAllRedirects?: boolean;
  maxRedirects?: number;
  removeRefererHeader?: boolean;

  // Proxy (see Redirects and Proxies doc for details)
  proxy?: string | URL;
  tunnel?: boolean;
  proxyHeaderWhiteList?: string[];
  proxyHeaderExclusiveList?: string[];

  // Response Handling
  encoding?: string | null;
  gzip?: boolean;
  jar?: boolean | RequestJar;

  // Miscellaneous
  callback?: function;
  time?: boolean;
  har?: object;
  httpModules?: object;
  pipefilter?: function;
}
```

## URI/URL Options

### uri (required)

```javascript { .api }
uri: string | URL
```

The fully qualified URI or URL object for the HTTP request. This is the only required option.

**Usage Examples:**

```javascript
// String URI
request({ uri: 'http://example.com/path' }, callback);

// URL object
const url = require('url');
request({ uri: url.parse('http://example.com/path') }, callback);

// HTTPS
request({ uri: 'https://secure.example.com/api' }, callback);
```

### url

```javascript { .api }
url: string | URL
```

Alias for `uri`. Either `uri` or `url` can be used.

**Usage Example:**

```javascript
request({ url: 'http://example.com' }, callback);
```

### baseUrl

```javascript { .api }
baseUrl: string
```

Base URL for relative paths. The `uri` option should be a relative path when using `baseUrl`.

**Usage Examples:**

```javascript
// Combine baseUrl with relative uri
request({
  baseUrl: 'http://api.example.com',
  uri: '/users/123'
}, callback);
// Results in: http://api.example.com/users/123

// Handles trailing slashes correctly
request({
  baseUrl: 'http://api.example.com/',
  uri: '/users'
}, callback);
// Results in: http://api.example.com/users
```

## HTTP Method

### method

```javascript { .api }
method: string
```

HTTP method to use. Defaults to 'GET' if not specified.

**Supported Methods:** GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS, and any other valid HTTP method.

**Usage Example:**

```javascript
request({
  uri: 'http://api.example.com/users',
  method: 'POST',
  json: true,
  body: { name: 'Alice' }
}, callback);
```

## Headers

### headers

```javascript { .api }
headers: object
```

HTTP headers as key-value pairs. Header names are case-insensitive.

**Usage Examples:**

```javascript
request({
  uri: 'http://example.com',
  headers: {
    'User-Agent': 'my-app/1.0',
    'Accept': 'application/json',
    'X-Custom-Header': 'value'
  }
}, callback);

// Authorization header
request({
  uri: 'http://api.example.com',
  headers: {
    'Authorization': 'Bearer token123'
  }
}, callback);
```

## Query String Options

### qs

```javascript { .api }
qs: object
```

Query string parameters as an object. These are appended to the URI.

**Usage Examples:**

```javascript
request({
  uri: 'http://api.example.com/search',
  qs: {
    query: 'nodejs',
    page: 1,
    limit: 10
  }
}, callback);
// Results in: http://api.example.com/search?query=nodejs&page=1&limit=10

// Arrays
request({
  uri: 'http://api.example.com/items',
  qs: {
    ids: [1, 2, 3]
  }
}, callback);
```

### qsParseOptions

```javascript { .api }
qsParseOptions: object
```

Options passed to the `qs.parse()` method for parsing query strings.

### qsStringifyOptions

```javascript { .api }
qsStringifyOptions: object
```

Options passed to the `qs.stringify()` method for building query strings.

**Usage Example:**

```javascript
request({
  uri: 'http://api.example.com',
  qs: { filters: { category: 'books', price: { max: 50 } } },
  qsStringifyOptions: { encode: false }
}, callback);
```

### useQuerystring

```javascript { .api }
useQuerystring: boolean
```

Use Node.js `querystring` module instead of `qs` module for query string handling.

## Request Body Options

### body

```javascript { .api }
body: string | Buffer | Stream
```

Request body as a string, Buffer, or readable stream.

**Usage Examples:**

```javascript
// String body
request({
  uri: 'http://api.example.com/data',
  method: 'POST',
  body: 'raw string data'
}, callback);

// Buffer body
request({
  uri: 'http://api.example.com/upload',
  method: 'POST',
  body: Buffer.from('binary data'),
  headers: { 'Content-Type': 'application/octet-stream' }
}, callback);

// Stream body
const fs = require('fs');
request({
  uri: 'http://api.example.com/upload',
  method: 'POST',
  body: fs.createReadStream('file.txt')
}, callback);
```

### json

```javascript { .api }
json: boolean | any
```

If `true`, sets `Content-Type: application/json` header and parses response as JSON. If an object/value is provided, it will be stringified and sent as the request body.

**Usage Examples:**

```javascript
// Send JSON and expect JSON response
request({
  uri: 'http://api.example.com/users',
  method: 'POST',
  json: true,
  body: { name: 'Alice', email: 'alice@example.com' }
}, function(error, response, body) {
  // body is automatically parsed as JSON
  console.log(body.id);
});

// Shorthand: provide object directly
request({
  uri: 'http://api.example.com/users',
  method: 'POST',
  json: { name: 'Bob', email: 'bob@example.com' }
}, callback);
```

### jsonReplacer

```javascript { .api }
jsonReplacer: function
```

Replacer function passed to `JSON.stringify()` when serializing the request body.

**Usage Example:**

```javascript
request({
  uri: 'http://api.example.com/data',
  method: 'POST',
  json: true,
  body: { password: 'secret', data: 'value' },
  jsonReplacer: function(key, value) {
    // Exclude password from serialization
    if (key === 'password') return undefined;
    return value;
  }
}, callback);
```

### jsonReviver

```javascript { .api }
jsonReviver: function
```

Reviver function passed to `JSON.parse()` when parsing JSON response body.

**Usage Example:**

```javascript
request({
  uri: 'http://api.example.com/data',
  json: true,
  jsonReviver: function(key, value) {
    // Parse date strings to Date objects
    if (key === 'createdAt') return new Date(value);
    return value;
  }
}, callback);
```

## Agent and Connection Options

### agent

```javascript { .api }
agent: http.Agent | https.Agent
```

Custom HTTP or HTTPS agent instance. Allows control over connection pooling and socket options.

**Usage Example:**

```javascript
const http = require('http');
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 50
});

request({
  uri: 'http://example.com',
  agent: agent
}, callback);
```

### agentClass

```javascript { .api }
agentClass: class
```

Custom Agent class to instantiate for the request.

### agentOptions

```javascript { .api }
agentOptions: object
```

Options passed to the agent constructor.

**Usage Example:**

```javascript
request({
  uri: 'http://example.com',
  agentOptions: {
    keepAlive: true,
    keepAliveMsecs: 10000,
    maxSockets: 100,
    maxFreeSockets: 10
  }
}, callback);
```

### forever

```javascript { .api }
forever: boolean
```

Use forever-agent to maintain persistent connections. Useful for making multiple requests to the same host.

**Usage Example:**

```javascript
request({
  uri: 'http://api.example.com',
  forever: true
}, callback);
```

### pool

```javascript { .api }
pool: object
```

Custom agent pool with optional `maxSockets` property. By default, requests use a global pool.

**Usage Example:**

```javascript
const customPool = { maxSockets: 100 };

request({
  uri: 'http://example.com',
  pool: customPool
}, callback);

// Disable pooling
request({
  uri: 'http://example.com',
  pool: false
}, callback);
```

### timeout

```javascript { .api }
timeout: number
```

Request timeout in milliseconds. Applies to the entire request/response cycle.

**Usage Examples:**

```javascript
// 10 second timeout
request({
  uri: 'http://example.com',
  timeout: 10000
}, function(error, response, body) {
  if (error && error.code === 'ETIMEDOUT') {
    console.error('Request timed out');
  }
});

// Very short timeout for health checks
request({
  uri: 'http://api.example.com/health',
  timeout: 1000
}, callback);
```

### localAddress

```javascript { .api }
localAddress: string
```

Local network interface to bind for outgoing connections.

**Usage Example:**

```javascript
request({
  uri: 'http://example.com',
  localAddress: '192.168.1.100'
}, callback);
```

## Response Handling Options

### encoding

```javascript { .api }
encoding: string | null
```

Encoding for the response body. If `null`, body is returned as a Buffer.

**Common Values:** 'utf8', 'ascii', 'base64', 'hex', 'binary', null

**Usage Examples:**

```javascript
// Text response
request({
  uri: 'http://example.com',
  encoding: 'utf8'
}, function(error, response, body) {
  // body is a string
  console.log(typeof body); // 'string'
});

// Binary response
request({
  uri: 'http://example.com/image.png',
  encoding: null
}, function(error, response, body) {
  // body is a Buffer
  console.log(Buffer.isBuffer(body)); // true
  fs.writeFileSync('image.png', body);
});
```

### gzip

```javascript { .api }
gzip: boolean
```

Automatically decompress gzip and deflate encoded responses. Adds 'accept-encoding: gzip, deflate' header.

**Usage Example:**

```javascript
request({
  uri: 'http://example.com',
  gzip: true
}, function(error, response, body) {
  // body is automatically decompressed
  console.log(body);
});
```

## Timing Option

### time

```javascript { .api }
time: boolean
```

Measure request duration. Adds `elapsedTime` property (in milliseconds) to the response object.

**Usage Example:**

```javascript
request({
  uri: 'http://api.example.com',
  time: true
}, function(error, response, body) {
  console.log('Request took', response.elapsedTime, 'ms');
});
```

## HAR Options

### har

```javascript { .api }
har: object
```

HAR 1.2 Request Object to configure the request. Overrides other options.

**HAR Request Structure:**

```javascript
{
  url: string,
  method: string,
  headers?: Array<{ name: string, value: string }>,
  queryString?: Array<{ name: string, value: string }>,
  cookies?: Array<{ name: string, value: string }>,
  postData?: {
    mimeType: string,
    params?: Array<{ name: string, value: string }>,
    text?: string
  }
}
```

**Usage Example:**

```javascript
request({
  har: {
    url: 'http://api.example.com/users',
    method: 'POST',
    headers: [
      { name: 'Content-Type', value: 'application/json' }
    ],
    postData: {
      mimeType: 'application/json',
      text: JSON.stringify({ name: 'Alice' })
    }
  }
}, callback);
```

## Advanced Options

### httpModules

```javascript { .api }
httpModules: object
```

Custom http/https modules to use instead of Node.js built-in modules.

**Usage Example:**

```javascript
const customHttp = require('custom-http-module');

request({
  uri: 'http://example.com',
  httpModules: {
    http: customHttp
  }
}, callback);
```

### pipefilter

```javascript { .api }
pipefilter: function
```

Function to filter/modify headers when piping requests.

**Signature:**

```javascript
function pipefilter(response, dest) {
  // Modify headers
  // Return modified response or headers object
}
```

## Unix Domain Sockets

Unix domain sockets are supported using a special URI format:

**Format:** `http://unix:SOCKET_PATH:REQUEST_PATH`

**Usage Example:**

```javascript
request({
  uri: 'http://unix:/var/run/docker.sock:/containers/json'
}, function(error, response, body) {
  console.log('Docker containers:', body);
});
```

## Options Usage Patterns

### Minimal Request

```javascript
request({ uri: 'http://example.com' }, callback);
```

### Common API Request

```javascript
request({
  uri: 'http://api.example.com/users',
  method: 'POST',
  json: true,
  body: { name: 'Alice' },
  headers: {
    'Authorization': 'Bearer token123'
  },
  timeout: 5000
}, callback);
```

### File Upload

```javascript
const fs = require('fs');

request({
  uri: 'http://service.com/upload',
  method: 'POST',
  formData: {
    file: fs.createReadStream('document.pdf'),
    description: 'Important document'
  }
}, callback);
```

### Streaming Download

```javascript
request({
  uri: 'http://example.com/large-file.zip',
  encoding: null,
  gzip: true
})
.on('response', function(response) {
  console.log('Content-Length:', response.headers['content-length']);
})
.pipe(fs.createWriteStream('file.zip'));
```

# Request - Simplified HTTP Client

Request is a simplified HTTP client library for Node.js that provides an intuitive API for making HTTP/HTTPS requests. It offers comprehensive support for various data formats, authentication methods, streaming, cookies, redirects, proxies, and SSL/TLS configuration. The library is designed for ease of use while maintaining flexibility for advanced use cases.

## Package Information

- **Package Name**: request
- **Package Type**: npm
- **Language**: JavaScript (Node.js)
- **Installation**: `npm install request`

## Core Imports

```javascript
const request = require('request');
```

Individual components can also be accessed:

```javascript
const request = require('request');
// Access Request class
const Request = request.Request;
// Access jar and cookie functions
const jar = request.jar;
const cookie = request.cookie;
```

## Basic Usage

### Simple GET Request

```javascript
const request = require('request');

request('http://www.example.com', function (error, response, body) {
  if (!error && response.statusCode === 200) {
    console.log(body); // HTML content
  }
});
```

### POST Request with JSON

```javascript
request.post({
  url: 'http://api.example.com/users',
  json: true,
  body: { name: 'John', email: 'john@example.com' }
}, function (error, response, body) {
  console.log('User created:', body);
});
```

### Streaming Response to File

```javascript
const fs = require('fs');
request('http://example.com/file.pdf').pipe(fs.createWriteStream('file.pdf'));
```

## Architecture

The request package is built around several key components:

- **Main Request Function**: The primary `request()` function that accepts a URI and options, returning a Request instance
- **Request Class**: Extends Node.js Stream, making it both readable and writable for bidirectional data flow
- **HTTP Method Helpers**: Convenience functions for common HTTP verbs (GET, POST, PUT, PATCH, DELETE, HEAD)
- **Authentication System**: Modular authentication supporting Basic, Digest, Bearer, OAuth 1.0, AWS, Hawk, and HTTP Signature
- **Cookie Management**: Full cookie jar support with automatic cookie handling
- **Streaming Interface**: Native stream support for efficient handling of large requests/responses
- **Event Emitter**: Rich event system for granular control over request lifecycle
- **Configuration System**: Defaults mechanism for creating pre-configured request instances

## Capabilities

### HTTP Method Convenience Functions

Shorthand functions for common HTTP methods with automatic method configuration.

```javascript { .api }
function request(uri: string | object, options?: object, callback?: function): Request;
function request.get(uri: string | object, options?: object, callback?: function): Request;
function request.head(uri: string | object, options?: object, callback?: function): Request;
function request.post(uri: string | object, options?: object, callback?: function): Request;
function request.put(uri: string | object, options?: object, callback?: function): Request;
function request.patch(uri: string | object, options?: object, callback?: function): Request;
function request.del(uri: string | object, options?: object, callback?: function): Request;
function request.delete(uri: string | object, options?: object, callback?: function): Request;
```

[HTTP Methods](./http-methods.md)

### Request Options

Comprehensive configuration options for controlling all aspects of HTTP requests including URI, method, headers, timeouts, and more.

```javascript { .api }
interface RequestOptions {
  uri: string | URL;
  method?: string;
  headers?: object;
  qs?: object;
  body?: string | Buffer | Stream;
  form?: object;
  formData?: object;
  json?: boolean | any;
  timeout?: number;
  encoding?: string | null;
  gzip?: boolean;
  followRedirect?: boolean | function;
  maxRedirects?: number;
  // ... and 50+ more options
}
```

[Request Options](./request-options.md)

### Authentication

Multiple authentication methods including Basic, Digest, Bearer tokens, OAuth 1.0, AWS signatures, Hawk, and HTTP Signature Scheme.

```javascript { .api }
interface AuthOptions {
  user?: string;
  username?: string;
  pass?: string;
  password?: string;
  sendImmediately?: boolean;
  bearer?: string;
}

function Request.prototype.auth(
  user: string,
  pass: string,
  sendImmediately?: boolean,
  bearer?: string
): Request;

function Request.prototype.oauth(params: OAuthParams): Request;
function Request.prototype.aws(options: AWSOptions, now?: boolean): Request;
function Request.prototype.hawk(options: HawkOptions): Request;
function Request.prototype.httpSignature(options: HttpSignatureOptions): Request;
```

[Authentication](./authentication.md)

### Body Handling

Support for URL-encoded forms, multipart form data, JSON payloads, and streaming request bodies.

```javascript { .api }
function Request.prototype.form(data?: object): FormData | Request;
function Request.prototype.json(val: any): Request;
function Request.prototype.multipart(data: array | object): Request;
```

[Body Handling](./body-handling.md)

### Cookie Management

Full cookie jar support with automatic cookie parsing, storage, and transmission.

```javascript { .api }
function request.jar(store?: CookieStore): RequestJar;
function request.cookie(str: string): Cookie;

interface RequestJar {
  setCookie(cookieOrStr: string | Cookie, uri: string, options?: object): void;
  getCookieString(uri: string): string;
  getCookies(uri: string): Cookie[];
}
```

[Cookies](./cookies.md)

### Redirects and Proxies

Automatic redirect following with configurable behavior and HTTP/HTTPS proxy support with tunneling.

```javascript { .api }
interface RedirectOptions {
  followRedirect?: boolean | function;
  followAllRedirects?: boolean;
  maxRedirects?: number;
  removeRefererHeader?: boolean;
}

interface ProxyOptions {
  proxy?: string | URL;
  tunnel?: boolean;
  proxyHeaderWhiteList?: string[];
  proxyHeaderExclusiveList?: string[];
}
```

[Redirects and Proxies](./redirects-proxies.md)

### SSL/TLS Configuration

Complete SSL/TLS configuration including certificates, keys, validation control, and cipher selection.

```javascript { .api }
interface SSLOptions {
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
}
```

[SSL/TLS Configuration](./ssl-tls.md)

### Events and Streaming

Rich event system for lifecycle control and full Node.js Stream compatibility for efficient data transfer.

```javascript { .api }
// Request Events
request.on('request', (req: http.ClientRequest) => void);
request.on('response', (resp: http.IncomingMessage) => void);
request.on('data', (chunk: Buffer) => void);
request.on('end', () => void);
request.on('complete', (resp: http.IncomingMessage, body: string | Buffer) => void);
request.on('error', (err: Error) => void);

// Stream Methods
function Request.prototype.pipe(dest: Stream, opts?: object): Stream;
function Request.prototype.write(chunk: any): boolean;
function Request.prototype.end(chunk?: any): void;
function Request.prototype.pause(): void;
function Request.prototype.resume(): void;
```

[Events and Streaming](./events-streaming.md)

### Advanced Configuration

Create pre-configured request instances with defaults, persistent connections with forever-agent, HAR 1.2 format support, and custom agent configuration.

```javascript { .api }
function request.defaults(options: object, requester?: function): RequestAPI;
function request.forever(agentOptions?: object, options?: object): RequestAPI;

interface HAROptions {
  har: {
    url: string;
    method: string;
    headers?: Array<{name: string, value: string}>;
    postData?: object;
  };
}
```

[Advanced Configuration](./advanced.md)

## Callback Signature

All request functions accept an optional callback with the following signature:

```javascript { .api }
function callback(
  error: Error | null,
  response: http.IncomingMessage & { body: string | Buffer },
  body: string | Buffer | any
): void;
```

**Parameters:**
- `error`: Error object if request failed, null on success
- `response`: HTTP response object with added `body` property
  - `statusCode`: HTTP status code (number)
  - `headers`: Response headers (object)
  - `body`: Response body (string, Buffer, or parsed JSON)
  - `elapsedTime`: Request duration in ms (if `time: true` option set)
- `body`: Convenience parameter containing response body

## Types

### Request Instance

```javascript { .api }
class Request extends Stream {
  // Core Methods
  init(options: object): void;
  start(): void;
  abort(): void;
  toJSON(): object;

  // Header Methods
  getHeader(name: string, headers?: object): any;
  setHeader(name: string, value: any): void;
  hasHeader(name: string): boolean;
  removeHeader(name: string): void;

  // Query String
  qs(obj: object, clobber?: boolean): Request;

  // Body Methods
  form(data?: object): FormData | Request;
  json(val: any): Request;
  multipart(data: array | object): Request;

  // Authentication Methods
  auth(user: string, pass: string, sendImmediately?: boolean, bearer?: string): Request;
  oauth(params: object): Request;
  aws(options: object, now?: boolean): Request;
  hawk(options: object): Request;
  httpSignature(options: object): Request;

  // Cookie Methods
  jar(jar?: RequestJar): RequestJar | Request;

  // Stream Methods
  pipe(dest: Stream, opts?: object): Stream;
  write(chunk: any): boolean;
  end(chunk?: any): void;
  pause(): void;
  resume(): void;
  destroy(): void;

  // Network Methods
  enableUnixSocket(): void;
  getNewAgent(): http.Agent | https.Agent;
}
```

### RequestAPI

Returned by `request.defaults()` and `request.forever()`:

```javascript { .api }
interface RequestAPI {
  (uri: string | object, options?: object, callback?: function): Request;
  get(uri: string | object, options?: object, callback?: function): Request;
  head(uri: string | object, options?: object, callback?: function): Request;
  post(uri: string | object, options?: object, callback?: function): Request;
  put(uri: string | object, options?: object, callback?: function): Request;
  patch(uri: string | object, options?: object, callback?: function): Request;
  del(uri: string | object, options?: object, callback?: function): Request;
  delete(uri: string | object, options?: object, callback?: function): Request;
  jar(store?: CookieStore): RequestJar;
  cookie(str: string): Cookie;
  defaults(options: object, requester?: function): RequestAPI;
}
```

## Error Handling

Request errors are passed to the callback's first parameter or emitted via the 'error' event:

```javascript
request('http://example.com', function (error, response, body) {
  if (error) {
    console.error('Request failed:', error.message);
    console.error('Error code:', error.code);
    return;
  }
  // Process response
});

// Or using events
request('http://example.com')
  .on('error', function(err) {
    console.error('Request error:', err);
  })
  .pipe(outputStream);
```

**Common Error Codes:**
- `ETIMEDOUT`: Request timeout
- `ESOCKETTIMEDOUT`: Socket timeout
- `ECONNRESET`: Connection reset
- `ENOTFOUND`: DNS lookup failed
- `ECONNREFUSED`: Connection refused
- SSL/TLS errors: Certificate validation failures

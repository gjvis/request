# Request

Request is a simplified HTTP client for Node.js that provides comprehensive functionality for making HTTP/HTTPS requests with minimal configuration. It supports streaming, multiple authentication methods, forms, cookies, proxies, redirects, and extensive customization options.

## Package Information

- **Package Name**: request
- **Package Type**: npm
- **Language**: JavaScript
- **Installation**: `npm install request`

## Core Imports

```javascript
const request = require('request');
```

## Basic Usage

```javascript
const request = require('request');

// Simple GET request with callback
request('http://www.google.com', function (error, response, body) {
  if (!error && response.statusCode == 200) {
    console.log(body);
  }
});

// POST request with form data
request.post('http://service.com/upload', {
  form: {key: 'value'}
}, function(err, httpResponse, body) {
  console.log('Server responded with:', body);
});

// Streaming response to file
request('http://google.com/doodle.png')
  .pipe(fs.createWriteStream('doodle.png'));
```

## Capabilities

### HTTP Methods

Convenience methods for making requests with specific HTTP verbs.

```javascript { .api }
function request(uri: string, options?: object, callback?: function): Request;
function request(options: object, callback?: function): Request;

function request.get(uri: string, options?: object, callback?: function): Request;
function request.post(uri: string, options?: object, callback?: function): Request;
function request.put(uri: string, options?: object, callback?: function): Request;
function request.patch(uri: string, options?: object, callback?: function): Request;
function request.head(uri: string, options?: object, callback?: function): Request;
function request.del(uri: string, options?: object, callback?: function): Request;
function request.delete(uri: string, options?: object, callback?: function): Request;
```

All methods accept:
- `uri` (string) - Target URL
- `options` (object, optional) - Request configuration
- `callback` (function, optional) - `function(error, response, body)`

Returns a `Request` instance that extends Node.js Stream.

[HTTP Methods and Options](./http-methods.md)

### Forms and Multipart

Support for URL-encoded forms and multipart form data including file uploads.

```javascript { .api }
// Form option
interface FormOption {
  form?: object | string;  // application/x-www-form-urlencoded
  formData?: object;        // multipart/form-data
  multipart?: Array<MultipartPart> | MultipartConfig;
}

interface MultipartPart {
  'content-type'?: string;
  body: string | Buffer | Stream;
}

interface MultipartConfig {
  chunked?: boolean;
  data: Array<MultipartPart>;
}
```

[Forms and File Uploads](./forms.md)

### Authentication

Multiple authentication methods including Basic, Bearer, Digest, OAuth 1.0, AWS, Hawk, and HTTP Signature.

```javascript { .api }
// Basic/Bearer authentication
interface AuthOption {
  user?: string;
  username?: string;
  pass?: string;
  password?: string;
  sendImmediately?: boolean;
  bearer?: string | function;
}

// OAuth 1.0
interface OAuthOption {
  consumer_key: string;
  consumer_secret: string;
  token?: string;
  token_secret?: string;
  signature_method?: string;  // 'HMAC-SHA1', 'RSA-SHA1', 'PLAINTEXT'
  transport_method?: string;  // 'header', 'query', 'body'
  body_hash?: boolean | string;
}

// AWS Signature
interface AWSOption {
  key: string;
  secret: string;
  bucket?: string;
  sign_version?: number;  // 2 or 4
}
```

[Authentication Methods](./authentication.md)

### Cookies

Cookie jar management for handling cookies across requests.

```javascript { .api }
function request.jar(store?: CookieStore): CookieJar;
function request.cookie(str: string): Cookie;

interface CookieJar {
  setCookie(cookie: Cookie | string, uri: string, options?: object): void;
  getCookieString(uri: string): string;
  getCookies(uri: string): Array<Cookie>;
}
```

[Cookie Management](./cookies.md)

### Streaming

Request instances are streams that support piping for both request and response bodies.

```javascript { .api }
interface Request extends Stream {
  pipe(dest: Stream, opts?: object): Stream;
  write(chunk: string | Buffer, encoding?: string, callback?: function): boolean;
  end(chunk?: string | Buffer): void;
  pause(): void;
  resume(): void;
  abort(): void;
}
```

Events emitted:
- `'response'` - Response received (http.IncomingMessage)
- `'data'` - Response data chunk
- `'end'` - Response complete
- `'error'` - Error occurred
- `'complete'` - Request/response cycle complete

[Streaming and Events](./streaming.md)

### Redirects and HTTP Control

Control redirect behavior, timeouts, retries, and response encoding.

```javascript { .api }
interface RedirectOptions {
  followRedirect?: boolean | function;
  followAllRedirects?: boolean;
  maxRedirects?: number;
  removeRefererHeader?: boolean;
}

interface ControlOptions {
  timeout?: number;
  encoding?: string | null;
  gzip?: boolean;
  time?: boolean;
}
```

[Redirects and HTTP Control](./http-control.md)

### Proxies

HTTP and HTTPS proxy support with environment variable configuration and tunneling.

```javascript { .api }
interface ProxyOptions {
  proxy?: string;
  tunnel?: boolean;
  proxyHeaderWhiteList?: Array<string>;
  proxyHeaderExclusiveList?: Array<string>;
}
```

Environment variables:
- `HTTP_PROXY` / `http_proxy`
- `HTTPS_PROXY` / `https_proxy`
- `NO_PROXY` / `no_proxy`

[Proxy Configuration](./proxies.md)

### SSL/TLS Configuration

Comprehensive SSL/TLS options for client certificates, custom CAs, and protocol selection.

```javascript { .api }
interface SSLOptions {
  strictSSL?: boolean;
  rejectUnauthorized?: boolean;
  ca?: string | Buffer | Array<string | Buffer>;
  cert?: string | Buffer;
  key?: string | Buffer;
  pfx?: string | Buffer;
  passphrase?: string;
  ciphers?: string;
  secureProtocol?: string;
  secureOptions?: number;
}
```

[SSL/TLS Options](./ssl-tls.md)

### Request Defaults and Configuration

Create request instances with pre-configured default options.

```javascript { .api }
function request.defaults(options: object, requester?: function): RequestWithDefaults;
function request.forever(agentOptions?: object, options?: object): RequestWithDefaults;

interface RequestWithDefaults {
  (uri: string, options?: object, callback?: function): Request;
  get: function;
  post: function;
  put: function;
  patch: function;
  head: function;
  del: function;
  delete: function;
  jar: function;
  cookie: function;
  defaults: function;
}
```

[Configuration and Defaults](./configuration.md)

## Types

### Request Options

```javascript { .api }
interface RequestOptions {
  // URL
  uri?: string;
  url?: string;
  baseUrl?: string;
  method?: string;
  headers?: object;

  // Query string
  qs?: object;
  qsParseOptions?: object;
  qsStringifyOptions?: object;
  useQuerystring?: boolean;

  // Body
  body?: string | Buffer | Stream | Array<any>;
  json?: boolean | object;
  jsonReviver?: function;
  jsonReplacer?: function;
  form?: object | string;
  formData?: object;
  multipart?: Array<object> | object;
  preambleCRLF?: boolean;
  postambleCRLF?: boolean;

  // Authentication
  auth?: AuthOption;
  oauth?: OAuthOption;
  hawk?: object;
  aws?: AWSOption;
  httpSignature?: object;

  // Cookies
  jar?: boolean | CookieJar;

  // Redirects
  followRedirect?: boolean | function;
  followAllRedirects?: boolean;
  maxRedirects?: number;
  removeRefererHeader?: boolean;

  // Response
  encoding?: string | null;
  gzip?: boolean;

  // Agent & Pool
  agent?: object | false;
  agentClass?: function;
  agentOptions?: object;
  forever?: boolean;
  pool?: object | false;

  // Timeout
  timeout?: number;

  // Proxy
  proxy?: string;
  tunnel?: boolean;
  proxyHeaderWhiteList?: Array<string>;
  proxyHeaderExclusiveList?: Array<string>;

  // SSL/TLS
  strictSSL?: boolean;
  rejectUnauthorized?: boolean;
  ca?: string | Buffer | Array<string | Buffer>;
  cert?: string | Buffer;
  key?: string | Buffer;
  pfx?: string | Buffer;
  passphrase?: string;
  ciphers?: string;
  secureProtocol?: string;
  secureOptions?: number;

  // Other
  localAddress?: string;
  time?: boolean;
  har?: object;
  callback?: function;
  httpModules?: object;
}
```

### Response Object

```javascript { .api }
interface Response extends http.IncomingMessage {
  statusCode: number;
  headers: object;
  body?: string | Buffer | object;
  request: Request;
  elapsedTime?: number;
  toJSON(): object;
}
```

### Callback Function

```javascript { .api }
type RequestCallback = (error: Error | null, response: Response, body: string | Buffer | object) => void;
```

### Debug Mode

```javascript { .api }
/**
 * Enable debug logging for all requests
 * Can also be enabled via NODE_DEBUG=request environment variable
 */
request.debug: boolean;
```

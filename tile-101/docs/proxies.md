# Proxy Configuration

HTTP and HTTPS proxy support with environment variable configuration, tunneling, and fine-grained header control.

## Capabilities

### Basic Proxy Configuration

Configure HTTP/HTTPS proxy for requests.

```javascript { .api }
/**
 * Proxy options
 */
interface ProxyOptions {
  /** Proxy URL (supports authentication: http://user:pass@host:port) */
  proxy?: string;

  /** Control tunneling behavior */
  tunnel?: boolean;

  /** Headers to whitelist for tunneling proxy */
  proxyHeaderWhiteList?: Array<string>;

  /** Headers exclusive to proxy (not sent to destination) */
  proxyHeaderExclusiveList?: Array<string>;
}
```

**Usage Examples:**

```javascript
// Basic proxy
request({
  uri: 'http://example.com',
  proxy: 'http://proxy-server.com:8080'
}, callback);

// Proxy with authentication
request({
  uri: 'http://example.com',
  proxy: 'http://username:password@proxy-server.com:8080'
}, callback);

// HTTPS proxy
request({
  uri: 'https://example.com',
  proxy: 'https://proxy-server.com:8080'
}, callback);
```

### Proxy Tunneling

Control HTTP CONNECT tunneling for HTTPS requests through proxies.

```javascript { .api }
/**
 * Tunnel option controls CONNECT tunneling behavior
 * - undefined (default): true for HTTPS, false for HTTP
 * - true: always tunnel (CONNECT to proxy)
 * - false: never tunnel (standard proxied request)
 */
interface TunnelOption {
  tunnel?: boolean;
}
```

**Usage Examples:**

```javascript
// Default behavior (tunnel for HTTPS)
request({
  uri: 'https://example.com',
  proxy: 'http://proxy-server.com:8080'
  // tunnel: undefined (defaults to true for HTTPS)
}, callback);

// Force tunneling for HTTP
request({
  uri: 'http://example.com',
  proxy: 'http://proxy-server.com:8080',
  tunnel: true
}, callback);

// Disable tunneling for HTTPS
request({
  uri: 'https://example.com',
  proxy: 'http://proxy-server.com:8080',
  tunnel: false
  // Warning: allows proxy to see traffic to/from destination
}, callback);
```

**How Tunneling Works:**

When tunneling is enabled (default for HTTPS), request makes a CONNECT request to the proxy:

```
HTTP/1.1 CONNECT endpoint-server.com:443
Host: proxy-server.com
User-Agent: <your user agent>
```

Proxy establishes TCP connection and returns:

```
HTTP/1.1 200 OK
```

Then client communicates directly with endpoint through the tunnel.

When tunneling is disabled, request makes a standard proxied HTTP request:

```
HTTP/1.1 GET http://endpoint-server.com/path
Host: proxy-server.com
Other-Headers: all go here

request body or whatever
```

### Proxy Header Control

Control which headers are sent to proxy vs destination server.

```javascript { .api }
/**
 * Proxy header configuration
 */
interface ProxyHeaderOptions {
  /** Headers to share with tunneling proxy */
  proxyHeaderWhiteList?: Array<string>;

  /** Headers sent only to proxy, not destination */
  proxyHeaderExclusiveList?: Array<string>;
}
```

**Default Header Whitelist:**

```
accept
accept-charset
accept-encoding
accept-language
accept-ranges
cache-control
content-encoding
content-language
content-length
content-location
content-md5
content-range
content-type
connection
date
expect
max-forwards
pragma
proxy-authorization
referer
te
transfer-encoding
user-agent
via
```

**Usage Examples:**

```javascript
// Custom header whitelist
request({
  uri: 'https://example.com',
  proxy: 'http://proxy-server.com:8080',
  proxyHeaderWhiteList: ['accept', 'user-agent', 'x-custom-header']
}, callback);

// Headers exclusive to proxy
request({
  uri: 'https://example.com',
  proxy: 'http://proxy-server.com:8080',
  proxyHeaderExclusiveList: ['x-proxy-only-header']
}, callback);
```

**Notes:**
- `proxy-authorization` header is **never** sent to endpoint server in tunneling mode
- Headers in `proxyHeaderExclusiveList` are sent only to proxy, not destination
- Custom whitelist replaces default whitelist

### Environment Variable Configuration

Request respects standard proxy environment variables.

```javascript { .api }
/**
 * Environment variables for proxy configuration:
 * - HTTP_PROXY / http_proxy: Proxy for HTTP requests
 * - HTTPS_PROXY / https_proxy: Proxy for HTTPS requests
 * - NO_PROXY / no_proxy: Hosts to exclude from proxying
 */
```

**Usage Examples:**

```javascript
// Set environment variables (before running app)
// HTTP_PROXY=http://proxy-server.com:8080
// HTTPS_PROXY=https://proxy-server.com:8080
// NO_PROXY=localhost,127.0.0.1,example.com

// Requests automatically use environment proxy
request('http://www.google.com', callback);
// Uses HTTP_PROXY

request('https://www.google.com', callback);
// Uses HTTPS_PROXY

// Override environment proxy
request({
  uri: 'http://www.google.com',
  proxy: 'http://different-proxy.com:8080'
}, callback);

// Opt out of environment proxy
request({
  uri: 'http://www.google.com',
  proxy: false
}, callback);

// Disable proxy with null
request({
  uri: 'http://www.google.com',
  proxy: null
}, callback);
```

### NO_PROXY Configuration

Fine-grained control over which hosts bypass proxy.

**NO_PROXY Format:**

```javascript
/**
 * NO_PROXY values (comma-separated):
 * - google.com: Don't proxy HTTP/HTTPS to Google
 * - google.com:443: Don't proxy HTTPS to Google (but do proxy HTTP)
 * - google.com:443,yahoo.com:80: Don't proxy HTTPS to Google or HTTP to Yahoo
 * - *: Ignore all proxy environment variables
 */
```

**Examples:**

```bash
# Don't proxy to Google
export NO_PROXY=google.com

# Don't proxy HTTPS to Google (but do proxy HTTP)
export NO_PROXY=google.com:443

# Don't proxy HTTPS to Google or HTTP to Yahoo
export NO_PROXY=google.com:443,yahoo.com:80

# Ignore all environment proxy settings
export NO_PROXY=*

# Multiple hosts
export NO_PROXY=localhost,127.0.0.1,.local,example.com
```

### Proxy with Defaults

Set default proxy for all requests.

**Usage Examples:**

```javascript
// Default proxy for all requests
const request = require('request').defaults({
  proxy: 'http://proxy-server.com:8080'
});

request('http://www.google.com', callback);
request('http://www.example.com', callback);
// Both use the default proxy

// Override default proxy
request({
  uri: 'http://www.example.com',
  proxy: 'http://different-proxy.com:8080'
}, callback);

// Disable proxy for specific request
request({
  uri: 'http://www.example.com',
  proxy: false
}, callback);
```

### Proxy Authentication

Authenticate with proxy server.

**Usage Examples:**

```javascript
// Proxy auth in URL
request({
  uri: 'http://example.com',
  proxy: 'http://username:password@proxy-server.com:8080'
}, callback);

// Proxy auth with special characters
const username = 'user@example.com';
const password = 'p@ss:word';
const proxy = 'http://' +
  encodeURIComponent(username) + ':' +
  encodeURIComponent(password) +
  '@proxy-server.com:8080';

request({
  uri: 'http://example.com',
  proxy: proxy
}, callback);

// Proxy-Authorization header (automatic for basic auth)
request({
  uri: 'http://example.com',
  proxy: 'http://username:password@proxy-server.com:8080'
  // Proxy-Authorization header added automatically
}, callback);
```

## Types

### Proxy Option Types

```javascript { .api }
/**
 * Proxy configuration options
 */
interface ProxyOptions {
  /** Proxy URL or false/null to disable */
  proxy?: string | false | null;

  /** Tunnel control (undefined = auto, true = force, false = disable) */
  tunnel?: boolean;

  /** Headers to whitelist for tunneling proxy */
  proxyHeaderWhiteList?: Array<string>;

  /** Headers exclusive to proxy */
  proxyHeaderExclusiveList?: Array<string>;
}

/**
 * Proxy URL format
 */
type ProxyURL = string;  // http://[user:pass@]host:port

/**
 * Environment variables
 */
interface ProxyEnvironment {
  HTTP_PROXY?: string;
  http_proxy?: string;
  HTTPS_PROXY?: string;
  https_proxy?: string;
  NO_PROXY?: string;
  no_proxy?: string;
}
```

## Notes

### Proxy Behavior
- Environment variables used automatically unless overridden
- Both uppercase and lowercase environment variable names supported
- `proxy` option overrides environment variables
- `proxy: false` or `proxy: null` disables proxy completely
- Proxy auth credentials in URL are automatically handled

### Tunneling
- Default tunneling: `true` for HTTPS destinations, `false` for HTTP
- Tunneling creates secure end-to-end connection through proxy
- Non-tunneling allows proxy to see all traffic
- `tunnel: false` with HTTPS exposes traffic to proxy (use with caution)

### Headers
- `proxy-authorization` never sent to destination in tunnel mode
- Default whitelist includes most standard HTTP headers
- `proxyHeaderWhiteList` replaces default whitelist
- `proxyHeaderExclusiveList` adds proxy-only headers

### NO_PROXY
- Comma-separated list of hosts or host:port pairs
- `*` disables all environment proxy settings
- Supports domain matching (e.g., `.example.com` matches `subdomain.example.com`)
- Port-specific exclusions supported (e.g., `example.com:443`)

### Security
- Use HTTPS proxy when available for better security
- Proxy credentials in URL are not encrypted (use HTTPS proxy)
- Be cautious with `tunnel: false` for HTTPS - proxy can see decrypted traffic
- Proxy-Authorization header contains credentials (use secure connection to proxy)

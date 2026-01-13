# Redirects and Proxies

The request package provides automatic redirect following with fine-grained control over redirect behavior, and comprehensive proxy support including HTTP CONNECT tunneling for secure connections through proxies.

## Capabilities

### Redirect Configuration

Control how redirects are handled with configurable options.

```javascript { .api }
interface RedirectOptions {
  /**
   * Follow HTTP 3xx redirects
   * - true: Follow GET/HEAD redirects (default)
   * - false: Don't follow redirects
   * - function: Custom redirect logic
   */
  followRedirect?: boolean | ((response: http.IncomingMessage) => boolean);

  /**
   * Follow all redirects including POST/PUT/DELETE (default: false)
   */
  followAllRedirects?: boolean;

  /**
   * Maximum number of redirects to follow (default: 10)
   */
  maxRedirects?: number;

  /**
   * Remove referer header on redirect (default: false)
   */
  removeRefererHeader?: boolean;
}
```

**Usage Examples:**

```javascript
const request = require('request');

// Follow redirects (default behavior)
request('http://example.com/redirect', function(err, res, body) {
  console.log('Final URL:', res.request.uri.href);
});

// Disable redirect following
request({
  url: 'http://example.com/redirect',
  followRedirect: false
}, function(err, res, body) {
  console.log('Status:', res.statusCode);  // 301, 302, etc.
  console.log('Location:', res.headers.location);
});

// Follow all redirects including POST
request.post({
  url: 'http://example.com/form',
  followAllRedirects: true,
  form: { data: 'value' }
});

// Custom redirect logic
request({
  url: 'http://example.com',
  followRedirect: function(response) {
    // Only follow redirects to same domain
    const location = response.headers.location;
    return location && location.indexOf('example.com') !== -1;
  }
});

// Limit redirect count
request({
  url: 'http://example.com',
  maxRedirects: 5
});
```

### Redirect Behavior

The redirect handler follows specific rules based on HTTP standards.

```javascript { .api }
/**
 * Redirect handling class
 */
class Redirect {
  constructor(request: Request);

  /**
   * Process redirect options
   * @param options - Request options with redirect settings
   */
  onRequest(options: object): void;

  /**
   * Determine redirect target URL from response
   * @param response - HTTP response object
   * @returns Redirect URL or null
   */
  redirectTo(response: http.IncomingMessage): string | null;

  /**
   * Handle redirect response
   * @param response - HTTP response object
   * @returns true if redirecting, false if not
   */
  onResponse(response: http.IncomingMessage): boolean;

  // Properties
  followRedirect: boolean;
  followAllRedirects: boolean;
  maxRedirects: number;
  redirects: Array<{statusCode: number, redirectUri: string}>;
  redirectsFollowed: number;
  removeRefererHeader: boolean;
  allowRedirect: (response: http.IncomingMessage) => boolean;
}
```

**Redirect Behavior Rules:**

- **3xx Status Codes**: Standard redirect codes (300-399) with Location header
- **followRedirect = true**: Follows GET and HEAD redirects only (default)
- **followAllRedirects = true**: Follows all HTTP methods including POST, PUT, DELETE
- **Method Conversion**: Non-GET/HEAD requests are converted to GET on redirect (except 307)
- **Status 307**: Preserves original HTTP method on redirect
- **Status 401**: Re-attempts request with authentication if credentials available
- **Authorization Removal**: Removes auth header when redirecting to different hostname
- **Referer Header**: Automatically sets referer to previous URL (unless removeRefererHeader is true)
- **Body Removal**: Request body is removed on redirect (except 307 and 401)
- **Protocol Changes**: Handles transitions between HTTP and HTTPS

**Usage Examples:**

```javascript
// Track redirect history
request({
  url: 'http://example.com/start',
  followRedirect: true
}, function(err, res, body) {
  // Access redirect history
  if (res.request._redirect && res.request._redirect.redirects) {
    res.request._redirect.redirects.forEach(redirect => {
      console.log('Redirected:', redirect.statusCode, redirect.redirectUri);
    });
  }
});

// Handle redirect errors
request({
  url: 'http://example.com/circular',
  maxRedirects: 3
}, function(err, res, body) {
  if (err && err.message.includes('Exceeded maxRedirects')) {
    console.error('Redirect loop detected');
  }
});

// Preserve referer on redirects
request({
  url: 'http://example.com',
  followRedirect: true,
  removeRefererHeader: false  // default
});
```

### Proxy Configuration

Configure HTTP and HTTPS proxies for routing requests.

```javascript { .api }
interface ProxyOptions {
  /**
   * Proxy server URL with optional authentication
   * Format: http://[user:pass@]host:port
   */
  proxy?: string | URL;

  /**
   * Use HTTP CONNECT tunneling (default: true for HTTPS)
   */
  tunnel?: boolean;

  /**
   * Headers to send to proxy server (whitelist)
   */
  proxyHeaderWhiteList?: string[];

  /**
   * Headers to send ONLY to proxy (not to destination)
   */
  proxyHeaderExclusiveList?: string[];
}
```

**Usage Examples:**

```javascript
// Basic proxy
request({
  url: 'http://example.com',
  proxy: 'http://proxy.example.com:8080'
});

// Proxy with authentication
request({
  url: 'http://example.com',
  proxy: 'http://username:password@proxy.example.com:8080'
});

// Use environment variable proxy
// Request automatically detects HTTP_PROXY/HTTPS_PROXY/NO_PROXY
request('http://example.com');  // Uses process.env.HTTP_PROXY if set

// Disable tunneling
request({
  url: 'https://example.com',
  proxy: 'http://proxy.example.com:8080',
  tunnel: false
});

// Custom proxy headers
request({
  url: 'http://example.com',
  proxy: 'http://proxy.example.com:8080',
  proxyHeaderWhiteList: ['accept', 'user-agent', 'custom-header']
});
```

### Tunnel Configuration

Control HTTP CONNECT tunneling for proxied HTTPS requests.

```javascript { .api }
/**
 * Tunnel handling class
 */
class Tunnel {
  constructor(request: Request);

  /**
   * Check if tunneling is enabled for this request
   * @returns true if tunnel should be used
   */
  isEnabled(): boolean;

  /**
   * Setup tunnel agent for request
   * @param options - Request options with proxy settings
   * @returns true if tunnel was configured
   */
  setup(options: object): boolean;

  // Static properties
  static defaultProxyHeaderWhiteList: string[];
  static defaultProxyHeaderExclusiveList: string[];
}
```

**Default Proxy Header Whitelist:**

```javascript { .api }
const defaultProxyHeaderWhiteList = [
  'accept',
  'accept-charset',
  'accept-encoding',
  'accept-language',
  'accept-ranges',
  'cache-control',
  'content-encoding',
  'content-language',
  'content-location',
  'content-md5',
  'content-range',
  'content-type',
  'connection',
  'date',
  'expect',
  'max-forwards',
  'pragma',
  'referer',
  'te',
  'user-agent',
  'via'
];

const defaultProxyHeaderExclusiveList = [
  'proxy-authorization'
];
```

**Usage Examples:**

```javascript
// Tunnel HTTPS through HTTP proxy (automatic)
request({
  url: 'https://api.example.com',
  proxy: 'http://proxy.example.com:8080'
  // tunnel: true is implicit for HTTPS
});

// Custom proxy header filtering
request({
  url: 'https://example.com',
  proxy: 'http://proxy.example.com:8080',
  proxyHeaderWhiteList: ['accept', 'user-agent'],
  proxyHeaderExclusiveList: ['proxy-authorization', 'x-proxy-secret']
});
```

### Environment Variable Proxy Detection

Automatic proxy detection from environment variables.

```javascript { .api }
/**
 * Get proxy URL from environment variables
 * @param uri - Parsed URL object
 * @returns Proxy URL or null
 */
function getProxyFromURI(uri: URL): string | null;
```

**Environment Variables:**

- **HTTP_PROXY** / **http_proxy**: Proxy for HTTP requests
- **HTTPS_PROXY** / **https_proxy**: Proxy for HTTPS requests
- **NO_PROXY** / **no_proxy**: Comma-separated list of hosts to bypass proxy

**NO_PROXY Format:**
- `*` - Bypass proxy for all hosts
- `example.com` - Bypass for domain and subdomains
- `example.com:8080` - Bypass for specific host and port
- `.example.com` - Bypass for subdomains only
- `192.168.1.1,localhost,*.local` - Multiple patterns

**Usage Examples:**

```bash
# Shell environment setup
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
export NO_PROXY=localhost,127.0.0.1,.internal.company.com
```

```javascript
// Request automatically uses environment proxy
request('http://example.com');  // Uses HTTP_PROXY

request('https://api.example.com');  // Uses HTTPS_PROXY

request('http://localhost:3000');  // Bypasses proxy (in NO_PROXY)

// Override environment proxy
request({
  url: 'http://example.com',
  proxy: null  // Disable proxy even if env var is set
});
```

## Usage Patterns

### Following Redirects with History

Track redirect chain and final destination:

```javascript
request({
  url: 'http://bit.ly/short-url',
  followRedirect: true
}, function(err, res, body) {
  // Final URL after all redirects
  console.log('Final URL:', res.request.uri.href);

  // Redirect count
  if (res.request._redirect) {
    console.log('Redirects:', res.request._redirect.redirectsFollowed);
  }
});
```

### Conditional Redirects

Custom logic for following redirects:

```javascript
request({
  url: 'http://example.com',
  followRedirect: function(response) {
    // Don't follow redirects to external sites
    const location = response.headers.location;
    const currentHost = this.uri.hostname;

    if (!location) return false;

    const redirectUrl = url.parse(url.resolve(this.uri.href, location));
    return redirectUrl.hostname === currentHost;
  }
});
```

### Corporate Proxy Setup

Configure for corporate proxy environment:

```javascript
const request = require('request').defaults({
  proxy: 'http://username:password@corporate-proxy.com:8080',
  tunnel: true,
  strictSSL: false  // If corporate proxy uses self-signed cert
});

// All requests use corporate proxy
request('http://external-api.com');
```

### Per-Request Proxy Override

Use different proxies for different requests:

```javascript
const normalRequest = request.defaults({
  proxy: 'http://proxy1.example.com:8080'
});

// Use default proxy
normalRequest('http://api1.example.com');

// Override with different proxy
normalRequest({
  url: 'http://api2.example.com',
  proxy: 'http://proxy2.example.com:8080'
});

// Disable proxy for specific request
normalRequest({
  url: 'http://internal.example.com',
  proxy: null
});
```

### SOCKS Proxy Support

Use SOCKS proxy with custom agent:

```javascript
const SocksProxyAgent = require('socks-proxy-agent');

const agent = new SocksProxyAgent('socks://proxy.example.com:1080');

request({
  url: 'http://example.com',
  agent: agent
});
```

## Notes

### Redirect Notes

- Default behavior follows GET/HEAD redirects automatically
- POST/PUT/DELETE requests are not followed by default (use `followAllRedirects`)
- Maximum redirect limit prevents infinite loops (default: 10)
- Authorization headers are removed when redirecting to different hostnames
- Request body is removed on redirect (except 307 and 401 status codes)
- The `redirect` event is emitted each time a redirect occurs
- Circular redirects are detected and result in an error

### Proxy Notes

- Environment variables are checked automatically if no proxy is specified
- HTTPS requests through HTTP proxies use CONNECT tunneling by default
- Proxy authentication is supported via URL format: `http://user:pass@host:port`
- The `tunnel` option controls whether HTTP CONNECT is used
- Custom proxy headers allow sending additional data to proxy servers
- NO_PROXY environment variable supports wildcards and port specifications
- Proxy settings do not affect Unix domain socket requests

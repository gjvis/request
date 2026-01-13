# Cookie Management

The request package provides comprehensive cookie management through a cookie jar system that automatically handles cookie storage, parsing, and transmission across requests. Built on the tough-cookie library, it supports RFC 6265 cookie standards with automatic domain matching, path handling, and expiration.

## Capabilities

### Creating Cookie Jars

Create cookie jar instances for managing cookies across requests.

```javascript { .api }
/**
 * Create a new cookie jar
 * @param store - Optional custom cookie store (must implement tough-cookie store interface)
 * @returns RequestJar instance
 */
function request.jar(store?: CookieStore): RequestJar;
```

**Usage Examples:**

```javascript
const request = require('request');

// Create a default cookie jar
const jar = request.jar();

// Use jar with request
request({ url: 'http://example.com', jar: jar }, function(err, res, body) {
  // Cookies from response are automatically stored in jar
});

// Create jar with custom store
const FileCookieStore = require('tough-cookie-filestore');
const customStore = new FileCookieStore('./cookies.json');
const persistentJar = request.jar(customStore);
```

### Parsing Cookies

Parse cookie strings into Cookie objects.

```javascript { .api }
/**
 * Parse a cookie string into a Cookie object
 * @param str - Cookie string to parse
 * @returns Cookie object from tough-cookie library
 * @throws {Error} If parameter is not a string
 */
function request.cookie(str: string): Cookie;
```

**Usage Examples:**

```javascript
// Parse a cookie string
const cookie = request.cookie('key=value; Domain=example.com; Path=/');

// Access cookie properties
console.log(cookie.key);      // 'key'
console.log(cookie.value);    // 'value'
console.log(cookie.domain);   // 'example.com'
console.log(cookie.path);     // '/'

// Set parsed cookie in jar
const jar = request.jar();
jar.setCookie(cookie, 'http://example.com');
```

### RequestJar Methods

The RequestJar class provides methods for managing cookies within the jar.

```javascript { .api }
/**
 * Set a cookie in the jar for a specific URI
 * @param cookieOrStr - Cookie object or cookie string
 * @param uri - URI to associate with the cookie
 * @param options - Optional settings for cookie storage
 * @returns The stored Cookie object
 */
function RequestJar.prototype.setCookie(
  cookieOrStr: string | Cookie,
  uri: string,
  options?: object
): Cookie;

/**
 * Get cookies for a URI as a cookie header string
 * @param uri - URI to get cookies for
 * @returns Cookie header string (e.g., "name1=value1; name2=value2")
 */
function RequestJar.prototype.getCookieString(uri: string): string;

/**
 * Get cookies for a URI as an array of Cookie objects
 * @param uri - URI to get cookies for
 * @returns Array of Cookie objects matching the URI
 */
function RequestJar.prototype.getCookies(uri: string): Cookie[];
```

**Usage Examples:**

```javascript
const jar = request.jar();
const uri = 'http://example.com/path';

// Set cookies
jar.setCookie('session=abc123; Path=/', uri);
jar.setCookie('user=john; Domain=example.com', uri);

// Get cookie string for request
const cookieHeader = jar.getCookieString(uri);
// Returns: "session=abc123; user=john"

// Get cookie objects
const cookies = jar.getCookies(uri);
cookies.forEach(cookie => {
  console.log(cookie.key, '=', cookie.value);
});
```

### Using Jars with Requests

Configure cookie jar handling in request options.

```javascript { .api }
interface CookieOptions {
  // Enable default cookie jar (shared across all requests)
  jar?: true;

  // Use custom cookie jar
  jar?: RequestJar;

  // Disable cookie handling
  jar?: false;
}

/**
 * Set or get the cookie jar for a request instance
 * @param jar - Cookie jar to use (omit to get current jar)
 * @returns Request instance (when setting) or current jar (when getting)
 */
function Request.prototype.jar(jar?: RequestJar): Request | RequestJar;
```

**Usage Examples:**

```javascript
// Use default global jar (enabled by default for all requests)
request({ url: 'http://example.com', jar: true });

// Use custom jar
const jar = request.jar();
request({ url: 'http://example.com', jar: jar });

// Disable cookies for a request
request({ url: 'http://example.com', jar: false });

// Set jar on request instance
const req = request('http://example.com');
req.jar(jar);

// Get current jar from request instance
const currentJar = req.jar();
```

## Cookie Types

### Cookie Object

Cookie objects from the tough-cookie library have these properties:

```javascript { .api }
interface Cookie {
  key: string;           // Cookie name
  value: string;         // Cookie value
  domain: string;        // Domain the cookie belongs to
  path: string;          // Path the cookie is valid for
  secure: boolean;       // Requires HTTPS
  httpOnly: boolean;     // Not accessible via JavaScript
  expires: Date;         // Expiration date (or Infinity for session cookies)
  maxAge: number;        // Max age in seconds
  sameSite: string;      // SameSite attribute (Strict, Lax, None)

  // Methods
  toString(): string;    // Convert to cookie string
  validate(): boolean;   // Validate cookie properties
}
```

### CookieStore Interface

Custom stores must implement the tough-cookie store interface:

```javascript { .api }
interface CookieStore {
  findCookie(domain: string, path: string, key: string, callback: (err: Error, cookie: Cookie) => void): void;
  findCookies(domain: string, path: string, callback: (err: Error, cookies: Cookie[]) => void): void;
  putCookie(cookie: Cookie, callback: (err: Error) => void): void;
  updateCookie(oldCookie: Cookie, newCookie: Cookie, callback: (err: Error) => void): void;
  removeCookie(domain: string, path: string, key: string, callback: (err: Error) => void): void;
  removeCookies(domain: string, path: string, callback: (err: Error) => void): void;
}
```

## Usage Patterns

### Session Management

Maintain session cookies across multiple requests:

```javascript
const request = require('request');
const jar = request.jar();

// Login request stores session cookie
request.post({
  url: 'http://api.example.com/login',
  jar: jar,
  form: { username: 'user', password: 'pass' }
}, function(err, res, body) {

  // Subsequent requests use stored session cookie
  request.get({
    url: 'http://api.example.com/profile',
    jar: jar
  }, function(err, res, body) {
    console.log('Profile data:', body);
  });
});
```

### Per-Domain Jars

Use separate jars for different domains or contexts:

```javascript
const apiJar = request.jar();
const cdnJar = request.jar();

// API requests use apiJar
request({ url: 'http://api.example.com', jar: apiJar });

// CDN requests use cdnJar
request({ url: 'http://cdn.example.com', jar: cdnJar });
```

### Manual Cookie Management

Explicitly set and retrieve cookies:

```javascript
const jar = request.jar();
const uri = 'http://example.com';

// Manually set cookies
jar.setCookie('auth=token123', uri);
jar.setCookie('preference=dark-mode', uri);

// Retrieve cookies
const cookies = jar.getCookies(uri);
console.log('Stored cookies:', cookies.length);

// Get cookie header string
const headerValue = jar.getCookieString(uri);
console.log('Cookie header:', headerValue);
```

### Persistent Cookie Storage

Use a custom store for persistent cookie storage:

```javascript
const FileCookieStore = require('tough-cookie-filestore');

// Create jar with file-based storage
const store = new FileCookieStore('./cookies.json');
const jar = request.jar(store);

// Cookies are automatically saved to disk
request({
  url: 'http://example.com',
  jar: jar
}, function(err, res, body) {
  // Cookies persisted to cookies.json
});
```

### Sharing Jars Across Request Instances

Use defaults to share a jar across multiple request configurations:

```javascript
const jar = request.jar();

// Create request instance with shared jar
const api = request.defaults({
  jar: jar,
  baseUrl: 'http://api.example.com'
});

// All requests use the same jar
api.post('/login', { form: { user: 'john' } });
api.get('/profile');  // Uses cookies from login
api.get('/data');     // Uses cookies from login
```

### Inspecting Cookies

Examine cookies stored in a jar:

```javascript
const jar = request.jar();

request({
  url: 'http://example.com',
  jar: jar
}, function(err, res, body) {

  // Get all cookies for domain
  const cookies = jar.getCookies('http://example.com');

  cookies.forEach(cookie => {
    console.log('Name:', cookie.key);
    console.log('Value:', cookie.value);
    console.log('Domain:', cookie.domain);
    console.log('Path:', cookie.path);
    console.log('Expires:', cookie.expires);
    console.log('HttpOnly:', cookie.httpOnly);
    console.log('Secure:', cookie.secure);
  });
});
```

## Notes

- Cookie jars automatically handle domain matching, path matching, and expiration
- The default behavior is to use a global shared jar across all requests
- Setting `jar: false` disables automatic cookie handling
- Cookies are parsed in "loose mode" for better compatibility
- The tough-cookie library handles RFC 6265 compliance
- Cookies set via `Set-Cookie` headers are automatically stored in the jar
- Stored cookies are automatically sent with matching requests
- Session cookies (without explicit expiry) are kept until jar is discarded
- Custom stores enable persistent storage, encryption, or other advanced features

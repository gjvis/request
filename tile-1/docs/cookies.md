# Cookie Management

Request provides a comprehensive cookie jar system for managing HTTP cookies across multiple requests, built on the tough-cookie library.

## Capabilities

### Cookie Jar Factory

Creates a new cookie jar for storing and managing cookies.

```javascript { .api }
/**
 * Creates a new cookie jar
 * @param {object} [store] - Optional custom cookie store
 * @returns {RequestJar} Cookie jar instance
 */
function request.jar(store);
```

**Usage Examples:**

```javascript
const request = require('request');

// Create a new cookie jar
const jar = request.jar();

// Use jar with requests
request({
  uri: 'http://example.com/login',
  jar: jar,
  form: {
    username: 'john',
    password: 'secret'
  }
}, function(err, response, body) {
  console.log('Logged in, cookies saved to jar');
});

// Cookies automatically sent on subsequent requests
request({
  uri: 'http://example.com/profile',
  jar: jar
}, function(err, response, body) {
  console.log('Profile accessed with session cookie');
});
```

### Cookie Parser

Parses a cookie string into a Cookie object.

```javascript { .api }
/**
 * Parses a cookie string
 * @param {string} str - Cookie string to parse
 * @returns {Cookie} Parsed Cookie object (from tough-cookie library)
 */
function request.cookie(str);
```

**Usage Examples:**

```javascript
const request = require('request');

// Parse cookie string
const cookie = request.cookie('key=value; Path=/; HttpOnly');
console.log('Cookie:', cookie);

// Manually add cookie to jar
const jar = request.jar();
jar.setCookie(cookie, 'http://example.com');
```

## RequestJar Class

The cookie jar manages cookies for multiple domains and paths.

```javascript { .api }
/**
 * Cookie jar class for managing cookies
 */
class RequestJar {
  /**
   * Sets a cookie in the jar
   * @param {string|Cookie} cookieOrStr - Cookie string or Cookie object
   * @param {string} uri - URI for the cookie domain/path
   * @param {object} [options] - Cookie options
   * @returns {Cookie} The cookie that was set
   */
  setCookie(cookieOrStr, uri, options);

  /**
   * Gets cookies for a URI as a string
   * @param {string} uri - URI to get cookies for
   * @returns {string} Cookie string (format: "key1=value1; key2=value2")
   */
  getCookieString(uri);

  /**
   * Gets cookies for a URI as an array
   * @param {string} uri - URI to get cookies for
   * @returns {Array<Cookie>} Array of Cookie objects
   */
  getCookies(uri);
}
```

### Set Cookie

Add or update a cookie in the jar.

**Usage Examples:**

```javascript
const request = require('request');
const jar = request.jar();

// Set cookie from string
jar.setCookie('session=abc123; Path=/; HttpOnly', 'http://example.com');

// Set cookie from Cookie object
const cookie = request.cookie('token=xyz789; Path=/api');
jar.setCookie(cookie, 'http://example.com');

// Set cookie with options
jar.setCookie('pref=dark_mode; Path=/', 'http://example.com', {
  ignoreError: true
});

// Set cookie for subdomain
jar.setCookie('user=john; Domain=.example.com', 'http://api.example.com');
```

### Get Cookie String

Retrieve cookies for a URI as a formatted string.

**Usage Examples:**

```javascript
const jar = request.jar();

// Set some cookies
jar.setCookie('session=abc123', 'http://example.com');
jar.setCookie('token=xyz789', 'http://example.com');

// Get cookie string for URI
const cookieString = jar.getCookieString('http://example.com');
console.log(cookieString); // "session=abc123; token=xyz789"

// Cookie string for specific path
jar.setCookie('admin=true; Path=/admin', 'http://example.com');
const adminCookies = jar.getCookieString('http://example.com/admin');
console.log(adminCookies); // "session=abc123; token=xyz789; admin=true"
```

### Get Cookies

Retrieve cookies for a URI as an array of Cookie objects.

**Usage Examples:**

```javascript
const jar = request.jar();

// Set cookies
jar.setCookie('session=abc123', 'http://example.com');
jar.setCookie('token=xyz789', 'http://example.com');

// Get cookies as array
const cookies = jar.getCookies('http://example.com');
console.log('Number of cookies:', cookies.length);

cookies.forEach(function(cookie) {
  console.log('Key:', cookie.key);
  console.log('Value:', cookie.value);
  console.log('Domain:', cookie.domain);
  console.log('Path:', cookie.path);
  console.log('Expires:', cookie.expires);
  console.log('HttpOnly:', cookie.httpOnly);
  console.log('Secure:', cookie.secure);
});

// Get cookies for specific path
const pathCookies = jar.getCookies('http://example.com/admin');
console.log('Cookies for /admin:', pathCookies.length);
```

### Associate Cookie Jar with Request (Method Form)

Associates a cookie jar with a specific request instance.

```javascript { .api }
/**
 * Associates a cookie jar with this request
 * @param {RequestJar} jar - Cookie jar instance to use
 * @returns {Request} The request instance (for chaining)
 */
Request.prototype.jar(jar);
```

**Usage Examples:**

```javascript
const request = require('request');
const jar = request.jar();

// Associate jar using method form
const req = request('http://example.com');
req.jar(jar);

// Method chaining
request('http://example.com')
  .jar(jar)
  .on('response', function(response) {
    console.log('Response received');
  });

// Note: Typically you would use the jar option instead:
// request({ uri: 'http://example.com', jar: jar }, callback);
```

## Cookie Jar Usage Patterns

### Per-Request Cookie Jar

```javascript
const request = require('request');

// Create separate jars for different users/sessions
const userJar = request.jar();
const adminJar = request.jar();

// User session
request.post({
  uri: 'http://example.com/login',
  jar: userJar,
  form: { username: 'user', password: 'pass' }
}, function(err, response, body) {
  console.log('User logged in');
});

// Admin session
request.post({
  uri: 'http://example.com/admin/login',
  jar: adminJar,
  form: { username: 'admin', password: 'adminpass' }
}, function(err, response, body) {
  console.log('Admin logged in');
});

// Separate requests with different sessions
request.get({ uri: 'http://example.com/profile', jar: userJar }, callback);
request.get({ uri: 'http://example.com/admin/dashboard', jar: adminJar }, callback);
```

### Global Cookie Jar

```javascript
// Use global cookie jar for all requests
request({
  uri: 'http://example.com',
  jar: true // Use global jar
}, callback);

// Or disable cookies entirely
request({
  uri: 'http://example.com',
  jar: false // No cookie handling
}, callback);
```

### Cookie Jar with Defaults

```javascript
const request = require('request');

// Create request instance with default jar
const jar = request.jar();
const sessionRequest = request.defaults({
  jar: jar
});

// All requests use the same jar
sessionRequest('http://example.com/login', callback);
sessionRequest('http://example.com/profile', callback);
sessionRequest('http://example.com/settings', callback);
```

### Inspecting Cookies

```javascript
const jar = request.jar();

// Make request that sets cookies
request({
  uri: 'http://example.com/login',
  jar: jar,
  form: { username: 'user', password: 'pass' }
}, function(err, response, body) {
  // Inspect cookies after response
  const cookies = jar.getCookies('http://example.com');

  cookies.forEach(function(cookie) {
    console.log('Cookie:', cookie.key, '=', cookie.value);
    console.log('Expires:', cookie.expires);

    // Check if cookie is still valid
    if (cookie.expiryTime() < Date.now()) {
      console.log('Cookie expired');
    }
  });
});
```

### Manual Cookie Management

```javascript
const request = require('request');
const jar = request.jar();

// Manually set cookies before request
jar.setCookie('session=manual123; Path=/', 'http://example.com');
jar.setCookie('pref=value; Path=/', 'http://example.com');

// Make request with pre-populated jar
request({
  uri: 'http://example.com/api',
  jar: jar
}, function(err, response, body) {
  console.log('Request made with manual cookies');

  // Get updated cookies after response
  const cookieString = jar.getCookieString('http://example.com');
  console.log('Current cookies:', cookieString);
});
```

### Persistent Cookies

```javascript
const request = require('request');
const fs = require('fs');

// Save cookies to file
function saveCookies(jar, filename) {
  const cookies = jar.getCookies('http://example.com');
  const cookieStrings = cookies.map(c => c.toString());
  fs.writeFileSync(filename, JSON.stringify(cookieStrings));
}

// Load cookies from file
function loadCookies(jar, filename) {
  if (fs.existsSync(filename)) {
    const cookieStrings = JSON.parse(fs.readFileSync(filename));
    cookieStrings.forEach(function(cookieString) {
      jar.setCookie(cookieString, 'http://example.com');
    });
  }
}

// Usage
const jar = request.jar();
loadCookies(jar, 'cookies.json');

request({ uri: 'http://example.com', jar: jar }, function(err, res, body) {
  saveCookies(jar, 'cookies.json');
});
```

### Custom Cookie Store

```javascript
const tough = require('tough-cookie');

// Create custom cookie store
class CustomStore extends tough.Store {
  constructor() {
    super();
    this.idx = {};
  }

  findCookie(domain, path, key, callback) {
    // Custom implementation
    callback(null, this.idx[key]);
  }

  findCookies(domain, path, callback) {
    // Custom implementation
    callback(null, Object.values(this.idx));
  }

  putCookie(cookie, callback) {
    // Custom implementation
    this.idx[cookie.key] = cookie;
    callback(null);
  }

  updateCookie(oldCookie, newCookie, callback) {
    // Custom implementation
    this.putCookie(newCookie, callback);
  }

  removeCookie(domain, path, key, callback) {
    // Custom implementation
    delete this.idx[key];
    callback(null);
  }

  removeCookies(domain, path, callback) {
    // Custom implementation
    this.idx = {};
    callback(null);
  }

  getAllCookies(callback) {
    // Custom implementation
    callback(null, Object.values(this.idx));
  }
}

// Use custom store
const store = new CustomStore();
const jar = request.jar(store);

request({ uri: 'http://example.com', jar: jar }, callback);
```

## Cookie Options

When setting cookies, these properties are respected:

```javascript { .api }
interface CookieProperties {
  /** Cookie key/name */
  key: string;

  /** Cookie value */
  value: string;

  /** Expiration date (Date object or string) */
  expires?: Date | string;

  /** Max age in seconds */
  maxAge?: number;

  /** Cookie domain */
  domain?: string;

  /** Cookie path */
  path?: string;

  /** Secure flag (HTTPS only) */
  secure?: boolean;

  /** HttpOnly flag (not accessible via JavaScript) */
  httpOnly?: boolean;

  /** SameSite policy: 'strict', 'lax', or 'none' */
  sameSite?: string;
}
```

**Example:**

```javascript
const jar = request.jar();

// Cookie with all properties
jar.setCookie(
  'session=abc123; ' +
  'Domain=.example.com; ' +
  'Path=/; ' +
  'Expires=Wed, 09 Jun 2025 10:18:14 GMT; ' +
  'Secure; ' +
  'HttpOnly; ' +
  'SameSite=Strict',
  'http://example.com'
);
```

## Automatic Cookie Handling

By default, request automatically handles cookies:

```javascript
// Automatic cookie management (default)
request('http://example.com/login', function(err, response, body) {
  // Cookies from Set-Cookie headers are stored
});

request('http://example.com/profile', function(err, response, body) {
  // Cookies automatically sent if domain/path match
});

// Disable automatic cookie handling
request({
  uri: 'http://example.com',
  jar: false
}, callback);
```

## Cookie Synchronization

The tough-cookie library used by request handles cookie synchronization:

- Cookies are synchronized based on domain and path
- Expired cookies are automatically removed
- Cookie precedence follows RFC 6265 rules
- Secure cookies only sent over HTTPS
- HttpOnly cookies not accessible outside HTTP requests

**Example:**

```javascript
const jar = request.jar();

// Set cookie
jar.setCookie('session=abc123; Max-Age=3600', 'http://example.com');

// Cookie is available
console.log(jar.getCookieString('http://example.com')); // "session=abc123"

// After 3600 seconds (1 hour), cookie expires and is removed automatically
// jar.getCookieString('http://example.com'); // ""
```

## Important Notes

1. **Domain Matching**: Cookies are only sent to matching domains and subdomains
2. **Path Matching**: Cookies are only sent to matching paths and sub-paths
3. **Secure Cookies**: Cookies with Secure flag only sent over HTTPS
4. **HttpOnly**: HttpOnly cookies cannot be accessed via document.cookie
5. **SameSite**: Controls cookie behavior with cross-site requests
6. **Expiration**: Cookies with past expiration dates are immediately invalid

## Debugging Cookies

```javascript
const jar = request.jar();

// Enable request debugging
request.debug = true;

request({
  uri: 'http://example.com',
  jar: jar
}, function(err, response, body) {
  // Inspect cookies
  const cookies = jar.getCookies('http://example.com');

  console.log('Received cookies:', cookies.length);
  cookies.forEach(function(cookie) {
    console.log('  Key:', cookie.key);
    console.log('  Value:', cookie.value);
    console.log('  Domain:', cookie.domain);
    console.log('  Path:', cookie.path);
    console.log('  Secure:', cookie.secure);
    console.log('  HttpOnly:', cookie.httpOnly);
    console.log('  SameSite:', cookie.sameSite);
    console.log('  Expires:', cookie.expires);
  });
});
```

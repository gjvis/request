# Cookie Management

Cookie jar management for handling cookies across requests with support for custom cookie stores.

## Capabilities

### Cookie Jar Creation

Create and manage cookie jars for persistent cookie storage across requests.

```javascript { .api }
/**
 * Create a cookie jar instance
 * @param store - Optional custom cookie store (e.g., FileCookieStore)
 * @returns Cookie jar instance
 */
function request.jar(store?: CookieStore): CookieJar;

interface CookieJar {
  /**
   * Set a cookie for a URI
   * @param cookie - Cookie object or string
   * @param uri - URI to associate cookie with
   * @param options - Additional options
   */
  setCookie(cookie: Cookie | string, uri: string, options?: object): void;

  /**
   * Get cookies as string for URI
   * @param uri - URI to get cookies for
   * @returns Cookie string (e.g., "key1=value1; key2=value2")
   */
  getCookieString(uri: string): string;

  /**
   * Get cookies as array for URI
   * @param uri - URI to get cookies for
   * @returns Array of Cookie objects
   */
  getCookies(uri: string): Array<Cookie>;
}
```

**Usage Examples:**

```javascript
// Create cookie jar
const jar = request.jar();

// Use jar with requests (enables cookies)
request('http://www.google.com', {jar: jar}, function() {
  // Cookies from first request are stored in jar
  request('http://images.google.com', {jar: jar});
  // Cookies automatically sent with second request
});

// Custom cookie store (requires tough-cookie-filestore)
const FileCookieStore = require('tough-cookie-filestore');
const jar = request.jar(new FileCookieStore('cookies.json'));

request = request.defaults({jar: jar});
request('http://www.google.com', function() {
  request('http://images.google.com');
  // Cookies persisted to cookies.json
});
```

### Cookie Parsing

Parse cookie strings into cookie objects.

```javascript { .api }
/**
 * Parse a cookie string into a cookie object
 * @param str - Cookie string (e.g., 'key1=value1')
 * @returns Cookie object
 */
function request.cookie(str: string): Cookie;
```

**Usage Example:**

```javascript
// Parse cookie string
const cookie = request.cookie('key1=value1');
console.log(cookie);
```

### Using Cookies with Requests

Enable or disable cookies for specific requests.

```javascript { .api }
/**
 * Cookie options
 */
interface CookieOptions {
  /** Enable cookies with jar (true for global jar, CookieJar instance for custom jar, false to disable) */
  jar?: boolean | CookieJar;
}
```

**Usage Examples:**

```javascript
// Enable cookies with global jar
const requestWithCookies = request.defaults({jar: true});
requestWithCookies('http://www.google.com', function() {
  requestWithCookies('http://images.google.com');
  // Cookies shared across requests
});

// Disable cookies (default behavior)
request('http://www.google.com', {jar: false}, callback);

// Custom cookie jar
const jar = request.jar();
request('http://www.google.com', {jar: jar}, callback);

// Using .jar() method
request.get('http://www.google.com')
  .jar(jar)
  .on('response', function(response) {
    console.log('Response received with cookies');
  });
```

### Manual Cookie Management

Manually set and retrieve cookies from a cookie jar.

**Usage Examples:**

```javascript
// Create jar and set cookies manually
const jar = request.jar();
const cookie = request.cookie('key1=value1');
const url = 'http://www.google.com';

jar.setCookie(cookie, url);

request({url: url, jar: jar}, function() {
  // Cookie sent with request
  request('http://images.google.com', {jar: jar});
});

// Inspect cookie jar after request
const jar = request.jar();
request({url: 'http://www.google.com', jar: jar}, function() {
  const cookieString = jar.getCookieString('http://www.google.com');
  console.log('Cookies:', cookieString);
  // "key1=value1; key2=value2; ..."

  const cookies = jar.getCookies('http://www.google.com');
  console.log('Cookie array:', cookies);
  // [{key: 'key1', value: 'value1', domain: "www.google.com", ...}, ...]
});
```

### Cookie Jar with Request Defaults

Use cookie jar with default request configuration.

**Usage Examples:**

```javascript
// Create request with default cookie jar
const jar = request.jar();
const req = request.defaults({jar: jar});

req('http://www.google.com', function() {
  req('http://images.google.com', function() {
    // All requests share the same cookie jar
    const cookies = jar.getCookies('http://www.google.com');
    console.log('Stored cookies:', cookies);
  });
});

// Global jar enabled by default
const req = request.defaults({jar: true});
req('http://www.google.com', function() {
  req('http://images.google.com');
  // Cookies automatically managed
});
```

### Cookie Store Interface

Custom cookie stores must implement the tough-cookie CookieStore API.

```javascript { .api }
/**
 * Cookie store interface (from tough-cookie)
 * Must support synchronous operations
 */
interface CookieStore {
  findCookie(domain: string, path: string, key: string, callback: (err: Error | null, cookie: Cookie | null) => void): void;
  findCookies(domain: string, path: string, callback: (err: Error | null, cookies: Array<Cookie>) => void): void;
  putCookie(cookie: Cookie, callback: (err: Error | null) => void): void;
  updateCookie(oldCookie: Cookie, newCookie: Cookie, callback: (err: Error | null) => void): void;
  removeCookie(domain: string, path: string, key: string, callback: (err: Error | null) => void): void;
  removeCookies(domain: string, path: string, callback: (err: Error | null) => void): void;
}
```

**Usage Example:**

```javascript
// Using FileCookieStore (requires tough-cookie-filestore package)
const FileCookieStore = require('tough-cookie-filestore');

// Note: cookies.json file must exist before use
const jar = request.jar(new FileCookieStore('cookies.json'));
const req = request.defaults({jar: jar});

req('http://www.google.com', function() {
  req('http://images.google.com');
  // Cookies saved to cookies.json
});
```

### Cookie Jar Method on Request Instance

Set cookie jar on a Request instance.

```javascript { .api }
/**
 * Set cookie jar for request
 * @param jar - Cookie jar instance, true for global jar, or false to disable
 * @returns this
 */
interface Request {
  jar(jar: boolean | CookieJar): Request;
}
```

**Usage Example:**

```javascript
const jar = request.jar();

request.get('http://www.google.com')
  .jar(jar)
  .on('response', function(response) {
    // Cookies stored in jar
    const cookies = jar.getCookies('http://www.google.com');
    console.log('Cookies:', cookies);
  });
```

## Types

### Cookie Types

```javascript { .api }
/**
 * Cookie object (from tough-cookie)
 */
interface Cookie {
  key: string;
  value: string;
  domain: string;
  path: string;
  expires?: Date;
  maxAge?: number;
  secure?: boolean;
  httpOnly?: boolean;
  [key: string]: any;
}

/**
 * Cookie jar interface
 */
interface CookieJar {
  setCookie(cookie: Cookie | string, uri: string, options?: {ignoreError?: boolean}): void;
  getCookieString(uri: string): string;
  getCookies(uri: string): Array<Cookie>;
  _jar?: any;  // Internal tough-cookie jar
}

/**
 * Cookie store interface
 */
interface CookieStore {
  findCookie(domain: string, path: string, key: string, callback: (err: Error | null, cookie: Cookie | null) => void): void;
  findCookies(domain: string, path: string, callback: (err: Error | null, cookies: Array<Cookie>) => void): void;
  putCookie(cookie: Cookie, callback: (err: Error | null) => void): void;
  updateCookie(oldCookie: Cookie, newCookie: Cookie, callback: (err: Error | null) => void): void;
  removeCookie(domain: string, path: string, key: string, callback: (err: Error | null) => void): void;
  removeCookies(domain: string, path: string, callback: (err: Error | null) => void): void;
}
```

## Notes

- Cookies are **disabled by default** in request
- Must explicitly enable cookies by setting `jar: true` or providing a CookieJar instance
- Global cookie jar is used when `jar: true`
- Custom cookie stores must implement the synchronous tough-cookie CookieStore API
- Cookie jars automatically handle Set-Cookie headers from responses
- Cookies are domain and path-specific
- Custom cookie stores enable persistent storage (e.g., FileCookieStore for JSON files)

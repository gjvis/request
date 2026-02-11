# Redirects and HTTP Control

Control redirect behavior, timeouts, response processing, and other HTTP request characteristics.

## Capabilities

### Redirect Control

Configure how HTTP 3xx redirects are handled.

```javascript { .api }
/**
 * Redirect configuration options
 */
interface RedirectOptions {
  /** Follow HTTP 3xx redirects (default: true) */
  followRedirect?: boolean | ((response: Response) => boolean);

  /** Follow non-GET HTTP 3xx redirects (default: false) */
  followAllRedirects?: boolean;

  /** Maximum number of redirects to follow (default: 10) */
  maxRedirects?: number;

  /** Remove referer header when redirect happens (default: false) */
  removeRefererHeader?: boolean;
}
```

**Usage Examples:**

```javascript
// Disable redirects
request({
  uri: 'http://example.com/redirect',
  followRedirect: false
}, callback);

// Custom redirect logic
request({
  uri: 'http://example.com',
  followRedirect: function(response) {
    // Only follow redirects to same domain
    return response.headers.location.indexOf('example.com') !== -1;
  }
}, callback);

// Follow all redirects including POST/PUT
request({
  uri: 'http://example.com',
  method: 'POST',
  followAllRedirects: true
}, callback);

// Limit redirect count
request({
  uri: 'http://example.com',
  maxRedirects: 5
}, callback);

// Remove referer header on redirect
request({
  uri: 'http://example.com',
  removeRefererHeader: true
}, callback);
```

### Timeout Configuration

Set timeout for server response.

```javascript { .api }
/**
 * Timeout option
 */
interface TimeoutOption {
  /** Timeout in milliseconds for server response */
  timeout?: number;
}
```

**Usage Examples:**

```javascript
// Request timeout
request({
  uri: 'http://example.com',
  timeout: 5000  // 5 seconds
}, callback);

// Check timeout error
request.get('http://10.255.255.1', {timeout: 1500}, function(err) {
  if (err) {
    console.log(err.code === 'ETIMEDOUT');  // true
    console.log(err.connect === true);      // true if connection timeout
  }
});

// Two types of timeouts:
// - Connection timeout: timeout while establishing connection
// - Read timeout: timeout while waiting for server response
```

**Timeout Error Detection:**

```javascript
request.get('http://slow-server.com', {timeout: 3000}, function(err, response, body) {
  if (err) {
    // Check if timeout error
    if (err.code === 'ETIMEDOUT') {
      // Connection timeout vs read timeout
      if (err.connect === true) {
        console.error('Connection timeout');
      } else {
        console.error('Read timeout');
      }
    }
  }
});
```

### Response Encoding

Configure response body encoding.

```javascript { .api }
/**
 * Encoding option
 */
interface EncodingOption {
  /**
   * Response encoding
   * - undefined/default: 'utf8'
   * - null: Buffer
   * - string: specific encoding (e.g., 'utf16le', 'ascii', 'base64')
   */
  encoding?: string | null;
}
```

**Usage Examples:**

```javascript
// Default encoding (utf8 string)
request('http://example.com', function(err, response, body) {
  console.log(typeof body);  // 'string'
});

// Binary data (Buffer)
request({
  uri: 'http://example.com/image.png',
  encoding: null
}, function(err, response, body) {
  console.log(Buffer.isBuffer(body));  // true
  fs.writeFileSync('image.png', body);
});

// Custom encoding
request({
  uri: 'http://example.com',
  encoding: 'base64'
}, function(err, response, body) {
  console.log('Base64:', body);
});

// Binary file download
request({
  uri: 'http://example.com/file.zip',
  encoding: null
}, function(err, response, body) {
  fs.writeFileSync('file.zip', body);
});
```

### Compression Support

Enable gzip/deflate compression for responses.

```javascript { .api }
/**
 * Compression option
 */
interface CompressionOption {
  /** Accept and decode gzip/deflate compressed responses (default: false) */
  gzip?: boolean;
}
```

**Usage Examples:**

```javascript
// Enable gzip compression
request({
  method: 'GET',
  uri: 'http://www.google.com',
  gzip: true
}, function(error, response, body) {
  // Body is automatically decompressed
  console.log('Encoding:', response.headers['content-encoding'] || 'identity');
  console.log('Decoded data:', body);
});

// Gzip with streaming
request({
  uri: 'http://www.google.com',
  gzip: true
})
.on('data', function(data) {
  // Decompressed data chunks
  console.log('Decoded chunk:', data);
})
.on('response', function(response) {
  // Raw compressed response
  response.on('data', function(data) {
    console.log('Received', data.length, 'bytes of compressed data');
  });
});

// Gzip is opt-in for backwards compatibility
request('http://www.google.com', function(err, response, body) {
  // Body is NOT decompressed unless gzip: true
});
```

**Notes:**
- When `gzip: true`, adds `Accept-Encoding: gzip, deflate` header
- Response body in callback is automatically decompressed
- Response stream from 'response' event contains compressed data
- Request stream from 'data' event contains decompressed data

### Request Timing

Enable timing measurements for request/response cycle.

```javascript { .api }
/**
 * Timing option
 */
interface TimingOption {
  /** Enable request timing (sets response.elapsedTime) */
  time?: boolean;
}
```

**Usage Examples:**

```javascript
// Enable timing
request({
  uri: 'http://example.com',
  time: true
}, function(err, response, body) {
  console.log('Request took:', response.elapsedTime, 'ms');
});

// Timing includes all redirects
request({
  uri: 'http://example.com/redirect',
  time: true,
  followRedirect: true
}, function(err, response, body) {
  // elapsedTime includes entire redirect chain
  console.log('Total time:', response.elapsedTime, 'ms');
});
```

### JSON Mode

Automatically parse response as JSON and stringify request body.

```javascript { .api }
/**
 * JSON options
 */
interface JSONOptions {
  /** Enable JSON mode and optionally set body */
  json?: boolean | object;

  /** Custom JSON reviver for parsing response */
  jsonReviver?: (key: string, value: any) => any;

  /** Custom JSON replacer for stringifying body */
  jsonReplacer?: (key: string, value: any) => any;
}
```

**Usage Examples:**

```javascript
// Parse response as JSON
request({
  uri: 'http://api.example.com/data',
  json: true
}, function(err, response, body) {
  // body is automatically parsed JSON object
  console.log(body.name);
});

// Send JSON object
request({
  uri: 'http://api.example.com/users',
  method: 'POST',
  json: {
    name: 'John Doe',
    age: 30
  }
}, function(err, response, body) {
  // Request body is stringified
  // Content-Type: application/json header added
  // Response body is parsed
  console.log(body);
});

// JSON with custom reviver
request({
  uri: 'http://api.example.com/data',
  json: true,
  jsonReviver: function(key, value) {
    if (key === 'date') {
      return new Date(value);
    }
    return value;
  }
}, callback);

// JSON with custom replacer
request({
  uri: 'http://api.example.com/data',
  method: 'POST',
  json: {
    date: new Date(),
    value: 123
  },
  jsonReplacer: function(key, value) {
    if (value instanceof Date) {
      return value.toISOString();
    }
    return value;
  }
}, callback);

// Using .json() method
request.post('http://api.example.com/users')
  .json({name: 'John', age: 30})
  .on('response', function(response) {
    console.log('JSON sent');
  });
```

### Headers Configuration

Set custom HTTP headers.

```javascript { .api }
/**
 * Headers option
 */
interface HeadersOption {
  /** Custom HTTP headers */
  headers?: {
    [key: string]: string | number;
  };
}
```

**Usage Examples:**

```javascript
// Custom headers
request({
  uri: 'https://api.github.com/repos/request/request',
  headers: {
    'User-Agent': 'request',
    'X-Custom-Header': 'value'
  }
}, callback);

// Set headers with method
const req = request.get('http://example.com');
req.setHeader('User-Agent', 'my-app/1.0');
req.setHeader('Accept', 'application/json');

// Remove headers
req.removeHeader('User-Agent');

// Check headers
if (req.hasHeader('User-Agent')) {
  console.log('User-Agent:', req.getHeader('User-Agent'));
}
```

### Method Configuration

Explicitly set HTTP method.

```javascript { .api }
/**
 * Method option
 */
interface MethodOption {
  /** HTTP method (default: 'GET') */
  method?: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE' | 'HEAD' | 'OPTIONS' | string;
}
```

**Usage Example:**

```javascript
// Explicit method
request({
  uri: 'http://example.com',
  method: 'PATCH',
  json: {status: 'active'}
}, callback);

// Any HTTP method
request({
  uri: 'http://example.com',
  method: 'OPTIONS'
}, callback);
```

## Types

### HTTP Control Option Types

```javascript { .api }
/**
 * Redirect options
 */
interface RedirectOptions {
  followRedirect?: boolean | ((response: Response) => boolean);
  followAllRedirects?: boolean;
  maxRedirects?: number;
  removeRefererHeader?: boolean;
}

/**
 * Timeout and encoding options
 */
interface ControlOptions {
  timeout?: number;
  encoding?: string | null;
  gzip?: boolean;
  time?: boolean;
}

/**
 * JSON processing options
 */
interface JSONOptions {
  json?: boolean | object;
  jsonReviver?: (key: string, value: any) => any;
  jsonReplacer?: (key: string, value: any) => any;
}

/**
 * Timeout error
 */
interface TimeoutError extends Error {
  code: 'ETIMEDOUT' | 'ESOCKETTIMEDOUT';
  connect?: boolean;  // true for connection timeout, false/undefined for read timeout
}
```

## Notes

### Redirects
- `followRedirect` defaults to `true`
- `followAllRedirects` defaults to `false` (only GET redirects followed)
- `maxRedirects` defaults to `10`
- Authorization headers removed when redirecting to different host
- Referer header maintained by default unless `removeRefererHeader: true`

### Timeouts
- Timeout applies to response headers, not entire request/response
- OS TCP connection timeout may override timeout setting
- Two types: connection timeout (`err.connect === true`) and read timeout
- Timeout error has `err.code === 'ETIMEDOUT'`

### Encoding
- Default encoding is `'utf8'`
- Use `encoding: null` for binary data (returns Buffer)
- `encoding` option affects response body only

### Compression
- Compression is opt-in (`gzip: false` by default)
- Automatically adds `Accept-Encoding` header when enabled
- Supports gzip and deflate encodings
- Response body in callback is decompressed
- Response stream contains compressed data

### Timing
- Measures entire request/response cycle including redirects
- Time measured in milliseconds
- Available as `response.elapsedTime`

### JSON Mode
- Sets `Content-Type: application/json` header
- Automatically stringifies request body
- Automatically parses response body
- Also sets `Accept: application/json` header
- Custom reviver/replacer functions supported

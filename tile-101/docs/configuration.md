# Configuration and Defaults

Create request instances with pre-configured default options and connection pooling strategies.

## Capabilities

### Request Defaults

Create a wrapper around request with default options pre-applied.

```javascript { .api }
/**
 * Create request wrapper with default options
 * @param options - Default options to apply to all requests
 * @param requester - Optional custom requester function
 * @returns Request-like function with defaults applied
 */
function request.defaults(options: object, requester?: function): RequestWithDefaults;

interface RequestWithDefaults {
  /** Make request with default options */
  (uri: string, options?: object, callback?: function): Request;

  /** HTTP verb convenience methods with defaults */
  get: (uri: string, options?: object, callback?: function) => Request;
  post: (uri: string, options?: object, callback?: function) => Request;
  put: (uri: string, options?: object, callback?: function) => Request;
  patch: (uri: string, options?: object, callback?: function) => Request;
  head: (uri: string, options?: object, callback?: function) => Request;
  del: (uri: string, options?: object, callback?: function) => Request;
  delete: (uri: string, options?: object, callback?: function) => Request;

  /** Utility methods */
  jar: (store?: CookieStore) => CookieJar;
  cookie: (str: string) => Cookie;

  /** Create nested defaults */
  defaults: (options: object, requester?: function) => RequestWithDefaults;
}
```

**Usage Examples:**

```javascript
// Default headers
const baseRequest = request.defaults({
  headers: {'x-token': 'my-token'}
});

baseRequest('http://api.example.com/users', callback);
// All requests include x-token header

// Default base URL
const apiRequest = request.defaults({
  baseUrl: 'https://api.example.com',
  headers: {'Authorization': 'Bearer token123'}
});

apiRequest('/users', callback);
// GET https://api.example.com/users

apiRequest('/posts', callback);
// GET https://api.example.com/posts

// Default JSON mode
const jsonRequest = request.defaults({
  json: true,
  headers: {'User-Agent': 'my-app/1.0'}
});

jsonRequest.post('http://api.example.com/users', {
  body: {name: 'John Doe'}
}, callback);
// Automatically sends JSON and parses response

// Default timeout
const timeoutRequest = request.defaults({
  timeout: 5000
});

timeoutRequest('http://slow-server.com', callback);
// 5 second timeout on all requests
```

### Nested Defaults

Create defaults from existing defaults to build up configuration.

**Usage Examples:**

```javascript
// Base request with token
const baseRequest = request.defaults({
  headers: {'x-token': 'my-token'}
});

// Add more headers to base
const specialRequest = baseRequest.defaults({
  headers: {'special': 'special value'}
});

specialRequest('http://api.example.com/resource', callback);
// Includes both x-token and special headers

// Override inherited defaults
const noTokenRequest = specialRequest.defaults({
  headers: {'x-token': undefined}  // Remove x-token
});

// Complex nesting
const level1 = request.defaults({
  baseUrl: 'https://api.example.com',
  json: true
});

const level2 = level1.defaults({
  headers: {'Authorization': 'Bearer token123'}
});

const level3 = level2.defaults({
  timeout: 5000
});

level3.get('/users', callback);
// Has baseUrl, json, headers, and timeout
```

### Forever Agent (Keep-Alive)

Create request instance with persistent connections.

```javascript { .api }
/**
 * Create request with forever-agent (keep-alive)
 * @param agentOptions - Optional agent options
 * @param options - Optional additional request options
 * @returns Request with keep-alive enabled
 */
function request.forever(agentOptions?: object, options?: object): RequestWithDefaults;
```

**Usage Examples:**

```javascript
// Simple forever agent
const foreverRequest = request.forever();

foreverRequest('http://api.example.com/users', callback);
foreverRequest('http://api.example.com/posts', callback);
// Connections reused between requests

// With agent options
const foreverRequest = request.forever({
  maxSockets: 50
});

// With additional options
const foreverRequest = request.forever(
  {maxSockets: 100},
  {json: true, baseUrl: 'https://api.example.com'}
);

foreverRequest.get('/users', callback);
foreverRequest.post('/users', {body: {name: 'John'}}, callback);
```

**How Forever Agent Works:**

```javascript
// In Node.js 0.10 and earlier:
// Uses forever-agent package for keep-alive

// In Node.js 0.12+:
// Uses native http.Agent with keepAlive: true
const agent = new http.Agent({keepAlive: true});
```

### Connection Pooling

Configure connection pooling for requests.

```javascript { .api }
/**
 * Pool options
 */
interface PoolOptions {
  /** Connection pool object */
  pool?: {
    /** Maximum sockets per agent */
    maxSockets?: number;
  } | false;
}
```

**Usage Examples:**

```javascript
// Custom pool with max sockets
const pool = {maxSockets: Infinity};

request({
  uri: 'http://api.example.com/resource',
  pool: pool
}, callback);

// Pool in defaults
const pooledRequest = request.defaults({
  pool: {maxSockets: 100}
});

pooledRequest('http://api.example.com/users', callback);

// Disable pooling
request({
  uri: 'http://api.example.com/resource',
  pool: false
}, callback);

// Multiple requests sharing pool
const pool = {maxSockets: 50};

for (let i = 0; i < 100; i++) {
  request({
    uri: 'http://api.example.com/resource/' + i,
    pool: pool
  }, callback);
}
// Max 50 concurrent connections
```

**Notes:**
- Global pool used by default if not specified
- Pool manages agent instances
- Different agents for different SSL/TLS configurations
- `maxSockets` controls concurrent connections
- Create pool outside loop for proper socket limiting

### HTTP Agent Configuration

Customize HTTP agent for advanced scenarios.

```javascript { .api }
/**
 * HTTP agent options
 */
interface AgentConfiguration {
  /** Custom HTTP agent instance */
  agent?: http.Agent | https.Agent | false;

  /** Custom agent class */
  agentClass?: typeof http.Agent;

  /** Options for agent creation */
  agentOptions?: {
    keepAlive?: boolean;
    keepAliveMsecs?: number;
    maxSockets?: number;
    maxFreeSockets?: number;
    timeout?: number;
    [key: string]: any;
  };
}
```

**Usage Examples:**

```javascript
const http = require('http');
const https = require('https');

// Custom agent instance
const agent = new http.Agent({
  keepAlive: true,
  maxSockets: 50,
  maxFreeSockets: 10
});

request({
  uri: 'http://api.example.com/resource',
  agent: agent
}, callback);

// Custom agent class
class CustomAgent extends http.Agent {
  createConnection(options, callback) {
    // Custom connection logic
    return super.createConnection(options, callback);
  }
}

request({
  uri: 'http://api.example.com/resource',
  agentClass: CustomAgent
}, callback);

// Agent options
request({
  uri: 'http://api.example.com/resource',
  agentOptions: {
    keepAlive: true,
    maxSockets: 100,
    maxFreeSockets: 10,
    timeout: 30000
  }
}, callback);

// Disable agent
request({
  uri: 'http://api.example.com/resource',
  agent: false
}, callback);
```

### Default Configuration Examples

Common default configuration patterns.

**API Client with Base URL and Auth:**

```javascript
const apiClient = request.defaults({
  baseUrl: 'https://api.example.com',
  headers: {
    'Authorization': 'Bearer ' + API_TOKEN,
    'User-Agent': 'my-app/1.0'
  },
  json: true,
  timeout: 10000
});

apiClient.get('/users', callback);
apiClient.post('/users', {body: {name: 'John'}}, callback);
```

**Cookie-Enabled Client:**

```javascript
const jar = request.jar();
const cookieRequest = request.defaults({
  jar: jar,
  followRedirect: true
});

cookieRequest('http://example.com/login', {
  method: 'POST',
  form: {username: 'user', password: 'pass'}
}, function() {
  cookieRequest('http://example.com/profile', callback);
  // Cookies from login automatically sent
});
```

**Proxy-Enabled Client:**

```javascript
const proxyRequest = request.defaults({
  proxy: 'http://proxy-server.com:8080',
  tunnel: true
});

proxyRequest('http://example.com', callback);
```

**High-Performance Client:**

```javascript
const fastRequest = request.defaults({
  pool: {maxSockets: Infinity},
  forever: true,
  gzip: true,
  time: true
});

fastRequest('http://api.example.com/data', function(err, response, body) {
  console.log('Request time:', response.elapsedTime, 'ms');
});
```

**Multi-Service Client:**

```javascript
// Service A client
const serviceA = request.defaults({
  baseUrl: 'https://service-a.example.com',
  headers: {'X-Service': 'A'},
  json: true
});

// Service B client
const serviceB = request.defaults({
  baseUrl: 'https://service-b.example.com',
  headers: {'X-Service': 'B'},
  json: true
});

serviceA.get('/users', callbackA);
serviceB.get('/orders', callbackB);
```

### Configuration with Custom Requester

Use custom requester function for advanced control.

```javascript { .api }
/**
 * Custom requester function
 * @param options - Request options
 * @param callback - Callback function
 * @returns Request instance
 */
type Requester = (options: object, callback?: function) => Request;
```

**Usage Example:**

```javascript
// Custom requester with logging
function loggingRequester(options, callback) {
  console.log('Making request to:', options.uri);
  return request(options, function(err, response, body) {
    console.log('Response status:', response && response.statusCode);
    if (callback) callback(err, response, body);
  });
}

const loggedRequest = request.defaults({
  baseUrl: 'https://api.example.com'
}, loggingRequester);

loggedRequest.get('/users', callback);
// Logs: Making request to: /users
// Logs: Response status: 200
```

## Types

### Configuration Types

```javascript { .api }
/**
 * Request defaults interface
 */
interface RequestWithDefaults {
  (uri: string, options?: object, callback?: function): Request;
  (options: object, callback?: function): Request;

  get: (uri: string, options?: object, callback?: function) => Request;
  post: (uri: string, options?: object, callback?: function) => Request;
  put: (uri: string, options?: object, callback?: function) => Request;
  patch: (uri: string, options?: object, callback?: function) => Request;
  head: (uri: string, options?: object, callback?: function) => Request;
  del: (uri: string, options?: object, callback?: function) => Request;
  delete: (uri: string, options?: object, callback?: function) => Request;

  jar: (store?: CookieStore) => CookieJar;
  cookie: (str: string) => Cookie;
  defaults: (options: object, requester?: function) => RequestWithDefaults;
}

/**
 * Pool configuration
 */
interface PoolConfig {
  maxSockets?: number;
}

/**
 * Agent configuration
 */
interface AgentConfig {
  agent?: http.Agent | https.Agent | false;
  agentClass?: typeof http.Agent;
  agentOptions?: {
    keepAlive?: boolean;
    keepAliveMsecs?: number;
    maxSockets?: number;
    maxFreeSockets?: number;
    timeout?: number;
    [key: string]: any;
  };
  forever?: boolean;
  pool?: PoolConfig | false;
}
```

## Notes

### request.defaults()
- Does **not** modify global request API
- Returns a new wrapper with defaults applied
- Options in individual requests override defaults
- Can nest defaults for layered configuration
- All convenience methods available on returned wrapper

### request.forever()
- Enables connection keep-alive
- Uses ForeverAgent in Node.js 0.10 and earlier
- Uses native http.Agent({keepAlive: true}) in Node.js 0.12+
- Improves performance for multiple requests to same host
- Reduces connection overhead

### Connection Pooling
- Global pool used by default
- Agents pooled by protocol and SSL configuration
- Pool key includes: protocol, CA, cert, ciphers, etc.
- `maxSockets` controls concurrent connections per host
- `pool: false` disables pooling entirely
- Create pool object once and reuse for proper limiting

### HTTP Agent
- `agent: false` disables agent and pooling
- Custom agent class allows connection customization
- agentOptions passed to agent constructor
- Different agents for HTTP vs HTTPS
- Agent selection automatic based on URI protocol

### Default Options Priority
- Individual request options override defaults
- Nested defaults override parent defaults
- Later defaults override earlier defaults
- `undefined` in nested defaults removes parent default

### Performance Tips
- Use `forever: true` for multiple requests to same host
- Set `pool: {maxSockets: Infinity}` for high concurrency
- Enable `gzip: true` to reduce bandwidth
- Use `baseUrl` to avoid repeating host in every request
- Enable `time: true` to measure performance
- Reuse cookie jars for session persistence

### Debug Mode

Enable debug logging for all requests.

```javascript { .api }
/**
 * Static property to enable debug logging
 * Also accessible via request.debug setter
 * Can be enabled via NODE_DEBUG=request environment variable
 */
Request.debug: boolean;
```

**Usage Examples:**

```javascript
// Enable debug mode programmatically
require('request').debug = true;

// Or via environment variable before running app
// NODE_DEBUG=request node app.js

// Debug output includes:
// - Request method, URL, headers
// - Response status, headers
// - Redirect information
// - Error details
```

### Best Practices
- Create default instances for different services/APIs
- Use baseUrl with relative URIs
- Set common headers in defaults
- Enable JSON mode for JSON APIs
- Configure timeout in defaults for reliability
- Use cookie jars for session-based APIs
- Set User-Agent in defaults for identification

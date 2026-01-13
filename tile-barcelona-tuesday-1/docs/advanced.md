# Advanced Configuration

The request package provides advanced configuration options for creating pre-configured request instances, maintaining persistent connections, supporting HAR (HTTP Archive) format, and customizing HTTP agents for fine-grained control over connection behavior.

## Capabilities

### Creating Request Defaults

Create pre-configured request instances with default options.

```javascript { .api }
/**
 * Create a wrapper with default options
 * @param options - Default options to apply to all requests
 * @param requester - Optional custom request function
 * @returns RequestAPI with all HTTP method helpers
 */
function request.defaults(
  options: object,
  requester?: (options: object, callback?: function) => Request
): RequestAPI;

/**
 * RequestAPI interface returned by defaults()
 */
interface RequestAPI {
  // Main request function
  (uri: string | object, options?: object, callback?: function): Request;

  // HTTP method helpers
  get(uri: string | object, options?: object, callback?: function): Request;
  head(uri: string | object, options?: object, callback?: function): Request;
  post(uri: string | object, options?: object, callback?: function): Request;
  put(uri: string | object, options?: object, callback?: function): Request;
  patch(uri: string | object, options?: object, callback?: function): Request;
  del(uri: string | object, options?: object, callback?: function): Request;
  delete(uri: string | object, options?: object, callback?: function): Request;

  // Cookie helpers
  jar(store?: CookieStore): RequestJar;
  cookie(str: string): Cookie;

  // Nested defaults
  defaults(options: object, requester?: function): RequestAPI;
}
```

**Usage Examples:**

```javascript
const request = require('request');

// Create API client with base configuration
const api = request.defaults({
  baseUrl: 'http://api.example.com',
  headers: {
    'User-Agent': 'MyApp/1.0',
    'Accept': 'application/json'
  },
  json: true,
  timeout: 5000
});

// All requests use default options
api.get('/users', function(err, res, body) {
  console.log('Users:', body);
});

api.post('/users', { body: { name: 'John' } });

// Override defaults per request
api.get({
  url: '/slow-endpoint',
  timeout: 30000  // Override default timeout
});

// Create nested defaults
const authenticatedAPI = api.defaults({
  headers: {
    'Authorization': 'Bearer ' + token
  }
});

authenticatedAPI.get('/protected');

// Different configurations for different services
const serviceA = request.defaults({
  baseUrl: 'http://service-a.example.com',
  auth: { user: 'app', pass: 'secret' }
});

const serviceB = request.defaults({
  baseUrl: 'http://service-b.example.com',
  oauth: { consumer_key: 'key', consumer_secret: 'secret' }
});
```

### Persistent Connections

Create request instances with persistent HTTP connections using forever-agent.

```javascript { .api }
/**
 * Create wrapper using forever-agent for persistent connections
 * @param agentOptions - Options for forever-agent
 * @param optionsArg - Additional request options
 * @returns RequestAPI with persistent connection agent
 */
function request.forever(
  agentOptions?: object,
  optionsArg?: object
): RequestAPI;

/**
 * Forever agent options
 */
interface ForeverAgentOptions {
  /**
   * Maximum number of sockets to allow per origin (default: Infinity)
   */
  maxSockets?: number;

  /**
   * Maximum number of free sockets per origin (default: 256)
   */
  maxFreeSockets?: number;

  /**
   * Minimum number of free sockets to keep (default: 0)
   */
  minSockets?: number;

  /**
   * Keep alive timeout in milliseconds (default: 90000)
   */
  keepAliveTimeout?: number;
}
```

**Usage Examples:**

```javascript
// Create request instance with persistent connections
const persistentRequest = request.forever();

// Make multiple requests using same connection
for (let i = 0; i < 100; i++) {
  persistentRequest('http://api.example.com/data/' + i, function(err, res, body) {
    console.log('Response', i);
  });
}

// Configure agent options
const customPersistent = request.forever({
  maxSockets: 50,
  maxFreeSockets: 10,
  keepAliveTimeout: 60000
});

// Combine with other defaults
const persistentAPI = request.forever({
  maxSockets: 100
}, {
  baseUrl: 'http://api.example.com',
  headers: { 'User-Agent': 'MyApp/1.0' },
  pool: { maxSockets: 100 }
});

// High-throughput configuration
const highThroughput = request.forever({
  maxSockets: Infinity,
  keepAliveTimeout: 90000
}, {
  timeout: 10000,
  gzip: true
});
```

### HAR Format Support

Import requests from HAR (HTTP Archive) 1.2 format.

```javascript { .api }
/**
 * HAR request object configuration
 */
interface HAROptions {
  har: {
    /**
     * Request URL
     */
    url: string;

    /**
     * HTTP method
     */
    method: string;

    /**
     * Request headers
     */
    headers?: Array<{ name: string; value: string }>;

    /**
     * Query string parameters
     */
    queryString?: Array<{ name: string; value: string }>;

    /**
     * Cookies
     */
    cookies?: Array<{ name: string; value: string }>;

    /**
     * POST data
     */
    postData?: {
      mimeType: string;
      params?: Array<{ name: string; value: string }>;
      text?: string;
    };

    /**
     * HTTP version (default: HTTP/1.1)
     */
    httpVersion?: string;
  };
}
```

**Usage Examples:**

```javascript
// Simple HAR request
request({
  har: {
    url: 'http://api.example.com/users',
    method: 'GET',
    headers: [
      { name: 'Accept', value: 'application/json' },
      { name: 'User-Agent', value: 'MyApp/1.0' }
    ]
  }
}, function(err, res, body) {
  console.log('Response:', body);
});

// HAR POST with form data
request({
  har: {
    url: 'http://api.example.com/login',
    method: 'POST',
    headers: [
      { name: 'Content-Type', value: 'application/x-www-form-urlencoded' }
    ],
    postData: {
      mimeType: 'application/x-www-form-urlencoded',
      params: [
        { name: 'username', value: 'user' },
        { name: 'password', value: 'pass' }
      ]
    }
  }
});

// HAR with JSON body
request({
  har: {
    url: 'http://api.example.com/data',
    method: 'POST',
    headers: [
      { name: 'Content-Type', value: 'application/json' }
    ],
    postData: {
      mimeType: 'application/json',
      text: JSON.stringify({ key: 'value' })
    }
  }
});

// HAR with cookies and query string
request({
  har: {
    url: 'http://api.example.com/search',
    method: 'GET',
    queryString: [
      { name: 'q', value: 'search term' },
      { name: 'limit', value: '10' }
    ],
    cookies: [
      { name: 'session', value: 'abc123' },
      { name: 'preference', value: 'dark' }
    ]
  }
});

// Import from HAR file
const fs = require('fs');
const harFile = JSON.parse(fs.readFileSync('network.har', 'utf8'));
const harEntry = harFile.log.entries[0];

request({
  har: harEntry.request
}, function(err, res, body) {
  console.log('Replayed HAR request');
});
```

### Custom Agent Configuration

Configure custom HTTP/HTTPS agents for fine-grained control.

```javascript { .api }
/**
 * Agent configuration options
 */
interface AgentOptions {
  /**
   * Custom http.Agent or https.Agent instance
   */
  agent?: http.Agent | https.Agent;

  /**
   * Custom Agent class to instantiate
   */
  agentClass?: typeof http.Agent;

  /**
   * Options passed to Agent constructor
   */
  agentOptions?: {
    /**
     * Maximum number of sockets per host
     */
    maxSockets?: number;

    /**
     * Maximum number of free sockets per host
     */
    maxFreeSockets?: number;

    /**
     * Keep alive timeout in milliseconds
     */
    timeout?: number;

    /**
     * Send keep-alive packets
     */
    keepAlive?: boolean;

    /**
     * Initial delay for keep-alive packets
     */
    keepAliveMsecs?: number;
  };

  /**
   * Use forever-agent for persistent connections
   */
  forever?: boolean;

  /**
   * Agent pool configuration
   */
  pool?: {
    maxSockets?: number;
  } | false;

  /**
   * Custom http/https modules
   */
  httpModules?: {
    http?: typeof http;
    https?: typeof https;
  };
}
```

**Usage Examples:**

```javascript
const http = require('http');
const https = require('https');

// Custom agent with specific settings
const customAgent = new http.Agent({
  keepAlive: true,
  keepAliveMsecs: 30000,
  maxSockets: 50,
  maxFreeSockets: 10
});

request({
  url: 'http://api.example.com',
  agent: customAgent
});

// Configure agent per protocol
const httpAgent = new http.Agent({ maxSockets: 100 });
const httpsAgent = new https.Agent({
  maxSockets: 100,
  keepAlive: true
});

request({
  url: 'http://example.com',
  agent: httpAgent
});

request({
  url: 'https://secure.example.com',
  agent: httpsAgent
});

// Agent options without custom agent instance
request({
  url: 'http://api.example.com',
  agentOptions: {
    maxSockets: 20,
    keepAlive: true,
    keepAliveMsecs: 10000
  }
});

// Disable agent pooling
request({
  url: 'http://example.com',
  pool: false
});

// Custom agent class
const CustomAgent = require('./my-custom-agent');

request({
  url: 'http://example.com',
  agentClass: CustomAgent,
  agentOptions: { custom: 'option' }
});

// Custom HTTP modules
const customHttp = require('custom-http-module');

request({
  url: 'http://example.com',
  httpModules: {
    http: customHttp
  }
});
```

### Connection Pooling

Configure connection pooling for efficient resource usage.

```javascript { .api }
/**
 * Pool configuration
 */
interface PoolOptions {
  /**
   * Connection pool configuration
   */
  pool?: {
    /**
     * Maximum sockets per host in pool
     */
    maxSockets?: number;
  } | false;  // false disables pooling

  /**
   * Local address to bind for network connections
   */
  localAddress?: string;
}
```

**Usage Examples:**

```javascript
// Custom pool size
request({
  url: 'http://api.example.com',
  pool: { maxSockets: 100 }
});

// Share pool across requests
const myPool = { maxSockets: 50 };

request({ url: 'http://api.example.com', pool: myPool });
request({ url: 'http://api.example.com', pool: myPool });

// Disable pooling for isolated requests
request({
  url: 'http://example.com',
  pool: false
});

// Configure pool in defaults
const api = request.defaults({
  pool: { maxSockets: 100 }
});

// Bind to specific local address
request({
  url: 'http://example.com',
  localAddress: '192.168.1.100'
});
```

### Debug Mode

Control debug logging for request operations.

```javascript { .api }
/**
 * Enable or disable debug logging
 * Property with getter/setter that controls debug output
 * @type boolean
 */
request.debug: boolean;
```

**Usage Examples:**

```javascript
// Enable debug mode
request.debug = true;

// Make requests with debug output
request('http://example.com', function(err, res, body) {
  // Debug information logged to console
});

// Check debug status
if (request.debug) {
  console.log('Debug mode is enabled');
}

// Disable debug mode
request.debug = false;
```

### Request Initialization

Internal parameter initialization for custom wrappers.

```javascript { .api }
/**
 * Initialize request parameters
 * Normalizes various calling patterns (uri, options, callback)
 * @param uri - Request URI
 * @param options - Request options
 * @param callback - Callback function
 * @returns Normalized parameters object
 */
function request.initParams(
  uri: string | object,
  options?: object | function,
  callback?: function
): object;
```

**Usage Examples:**

```javascript
// Used internally but exposed for custom wrappers
const params = request.initParams(
  'http://example.com',
  { method: 'POST' },
  function(err, res, body) {}
);

// Custom wrapper using initParams
function customRequest(uri, options, callback) {
  const params = request.initParams(uri, options, callback);

  // Add custom logic
  params.headers = params.headers || {};
  params.headers['X-Custom'] = 'value';

  return request(params);
}

customRequest('http://example.com', function(err, res, body) {
  console.log('Custom request complete');
});
```

## Usage Patterns

### API Client Library

Build a complete API client with defaults:

```javascript
const request = require('request');

class APIClient {
  constructor(config) {
    this.config = config;
    this.request = request.defaults({
      baseUrl: config.baseUrl,
      headers: {
        'User-Agent': `${config.appName}/${config.version}`,
        'Accept': 'application/json'
      },
      json: true,
      timeout: config.timeout || 10000
    });

    if (config.auth) {
      this.request = this.request.defaults({
        auth: config.auth
      });
    }
  }

  get(path, options, callback) {
    return this.request.get(path, options, callback);
  }

  post(path, data, options, callback) {
    return this.request.post(path, { ...options, body: data }, callback);
  }

  put(path, data, options, callback) {
    return this.request.put(path, { ...options, body: data }, callback);
  }

  delete(path, options, callback) {
    return this.request.delete(path, options, callback);
  }
}

// Usage
const api = new APIClient({
  baseUrl: 'http://api.example.com',
  appName: 'MyApp',
  version: '1.0',
  auth: { user: 'app', pass: 'secret' }
});

api.get('/users', function(err, res, body) {
  console.log('Users:', body);
});
```

### Multi-Environment Configuration

Configure different environments:

```javascript
const environments = {
  development: {
    baseUrl: 'http://localhost:3000',
    strictSSL: false,
    timeout: 30000
  },
  staging: {
    baseUrl: 'https://staging.example.com',
    strictSSL: true,
    timeout: 10000
  },
  production: {
    baseUrl: 'https://api.example.com',
    strictSSL: true,
    timeout: 5000,
    forever: true,
    pool: { maxSockets: 100 }
  }
};

const env = process.env.NODE_ENV || 'development';
const config = environments[env];

const api = config.forever
  ? request.forever({ maxSockets: config.pool.maxSockets }, config)
  : request.defaults(config);

module.exports = api;
```

### High-Performance Configuration

Optimize for high throughput:

```javascript
const highPerformanceRequest = request.forever({
  maxSockets: Infinity,
  maxFreeSockets: 256,
  keepAliveTimeout: 90000
}, {
  gzip: true,
  timeout: 10000,
  pool: { maxSockets: Infinity },
  agentOptions: {
    keepAlive: true,
    keepAliveMsecs: 30000
  }
});

// Make thousands of requests efficiently
for (let i = 0; i < 10000; i++) {
  highPerformanceRequest(`http://api.example.com/item/${i}`, function(err, res, body) {
    if (err) {
      console.error('Request failed:', err.message);
    }
  });
}
```

### Replaying HAR Files

Replay captured network traffic from HAR files:

```javascript
const fs = require('fs');

function replayHAR(harFile, callback) {
  const har = JSON.parse(fs.readFileSync(harFile, 'utf8'));
  const requests = har.log.entries.map(entry => entry.request);

  let completed = 0;
  requests.forEach((harRequest, index) => {
    request({ har: harRequest }, function(err, res, body) {
      completed++;
      console.log(`Request ${index + 1}/${requests.length} complete`);

      if (completed === requests.length) {
        callback();
      }
    });
  });
}

replayHAR('./network-capture.har', function() {
  console.log('All HAR requests replayed');
});
```

### Custom Agent with Monitoring

Agent with request monitoring:

```javascript
const http = require('http');
const EventEmitter = require('events');

class MonitoredAgent extends http.Agent {
  constructor(options) {
    super(options);
    this.events = new EventEmitter();
    this.stats = {
      created: 0,
      closed: 0,
      active: 0
    };
  }

  createConnection(options, callback) {
    this.stats.created++;
    this.stats.active++;
    this.events.emit('socket:created', this.stats);

    const socket = super.createConnection(options, callback);

    socket.on('close', () => {
      this.stats.closed++;
      this.stats.active--;
      this.events.emit('socket:closed', this.stats);
    });

    return socket;
  }
}

const monitoredAgent = new MonitoredAgent({ keepAlive: true });

monitoredAgent.events.on('socket:created', (stats) => {
  console.log('Socket created. Active:', stats.active);
});

const api = request.defaults({ agent: monitoredAgent });
```

## Types

### RequestAPI Type

Complete type definition for configured request instances:

```javascript { .api }
interface RequestAPI {
  // Main function
  (uri: string | URL, options?: object, callback?: function): Request;
  (uri: object, callback?: function): Request;
  (options: object, callback?: function): Request;

  // HTTP methods
  get(uri: string | object, options?: object, callback?: function): Request;
  head(uri: string | object, options?: object, callback?: function): Request;
  post(uri: string | object, options?: object, callback?: function): Request;
  put(uri: string | object, options?: object, callback?: function): Request;
  patch(uri: string | object, options?: object, callback?: function): Request;
  del(uri: string | object, options?: object, callback?: function): Request;
  delete(uri: string | object, options?: object, callback?: function): Request;

  // Cookie management
  jar(store?: CookieStore): RequestJar;
  cookie(str: string): Cookie;

  // Configuration
  defaults(options: object, requester?: function): RequestAPI;
}
```

## Notes

### Defaults Behavior

- Options are merged with each request, not replaced
- Per-request options override default options
- Headers are merged (per-request headers add to or override defaults)
- Nested defaults create new instances without modifying parent
- The `defaults()` method returns a new function, not modifying the original

### Forever Agent

- Keeps HTTP connections alive for reuse across requests
- Significantly improves performance for multiple requests to same host
- Manages connection pool automatically
- Configurable socket limits and keep-alive timeouts
- Best for high-throughput applications making many requests

### HAR Support

- Supports HAR 1.2 specification
- Automatically converts HAR format to request options
- Handles various content types (JSON, form data, multipart)
- Useful for replaying captured network traffic
- Can import from browser dev tools or proxy tools like Charles, Fiddler

### Agent Configuration

- Agents control connection pooling and reuse
- Default agent pools 5 sockets per host
- Custom agents enable fine-grained control
- Different agents can be used for HTTP vs HTTPS
- Agents are protocol-specific (HTTP or HTTPS)

### Performance Considerations

- Use `forever()` for multiple requests to improve performance
- Configure `maxSockets` based on server capacity and rate limits
- Keep-alive connections reduce latency for subsequent requests
- Connection pooling reduces overhead of connection establishment
- Monitor memory usage with large connection pools

### Best Practices

- Create defaults instances at module level for reuse
- Use environment-specific configurations
- Configure reasonable timeouts for all requests
- Enable gzip compression for bandwidth efficiency
- Use forever-agent for high-throughput applications
- Monitor agent statistics in production
- Clean up custom agents properly when done

# Web Session Manager

Build a session manager that maintains authentication state across multiple HTTP requests to different endpoints.

## Requirements

Your task is to implement a session manager that:

1. **Maintains Session State**: Automatically persist and reuse authentication cookies across multiple HTTP requests
2. **Handles Multiple Sessions**: Support multiple independent session contexts that don't interfere with each other
3. **Session Inspection**: Provide a way to retrieve all active cookies for a given session
4. **Session Cleanup**: Clear all cookies for a specific session when requested

## Implementation Details

Create a file `session-manager.js` that exports a `SessionManager` class with the following interface:

### Constructor
```javascript
new SessionManager()
```
Creates a new session manager instance.

### Methods

#### `createSession(sessionId)`
Creates a new session context identified by `sessionId`. Returns the session context object that can be used to make requests.

#### `makeRequest(sessionId, url, callback)`
Makes an HTTP GET request to the specified URL using the session identified by `sessionId`. The callback should follow the standard Node.js callback pattern: `callback(error, response, body)`.

#### `getSessionCookies(sessionId, url)`
Returns an array of cookie objects currently stored for the given session and URL. Each cookie object should include at least the `key` and `value` properties.

#### `clearSession(sessionId)`
Removes all cookies associated with the given session.

## Test Server

For testing purposes, a simple test server is provided that sets and expects cookies:

```javascript
// test-server.js
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/login') {
    res.writeHead(200, {
      'Set-Cookie': 'sessionToken=abc123; Path=/',
      'Content-Type': 'text/plain'
    });
    res.end('Logged in');
  } else if (req.url === '/protected') {
    const cookies = req.headers.cookie || '';
    if (cookies.includes('sessionToken=abc123')) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('Access granted');
    } else {
      res.writeHead(401, {'Content-Type': 'text/plain'});
      res.end('Unauthorized');
    }
  } else {
    res.writeHead(404, {'Content-Type': 'text/plain'});
    res.end('Not found');
  }
});

server.listen(3000);
```

## Test Cases { .test-cases }

### Test 1: Session Persistence { .test-case @test }

**Test File**: `session-manager.test.js`

```javascript
const SessionManager = require('./session-manager');
const manager = new SessionManager();

// Start test server on port 3000
// ... server code ...

// Create a session
manager.createSession('user1');

// Login to get cookie
manager.makeRequest('user1', 'http://localhost:3000/login', (err, res, body) => {
  console.log('Login response:', body); // Should print: "Logged in"

  // Access protected resource using the same session
  manager.makeRequest('user1', 'http://localhost:3000/protected', (err, res, body) => {
    console.log('Protected response:', body); // Should print: "Access granted"
    console.log('Status code:', res.statusCode); // Should print: 200
  });
});
```

**Expected Output**:
- First request receives "Logged in" response
- Second request receives "Access granted" with status code 200
- Cookie from first request is automatically sent in second request

### Test 2: Session Isolation { .test-case @test }

**Test File**: `session-manager.test.js`

```javascript
const SessionManager = require('./session-manager');
const manager = new SessionManager();

manager.createSession('user1');
manager.createSession('user2');

// user1 logs in
manager.makeRequest('user1', 'http://localhost:3000/login', (err, res, body) => {
  // user2 tries to access protected resource without logging in
  manager.makeRequest('user2', 'http://localhost:3000/protected', (err, res, body) => {
    console.log('Status code:', res.statusCode); // Should print: 401
  });
});
```

**Expected Output**:
- user1's cookies should not be accessible to user2
- user2's request returns status code 401 (Unauthorized)

### Test 3: Cookie Inspection { .test-case @test }

**Test File**: `session-manager.test.js`

```javascript
const SessionManager = require('./session-manager');
const manager = new SessionManager();

manager.createSession('user1');

manager.makeRequest('user1', 'http://localhost:3000/login', (err, res, body) => {
  const cookies = manager.getSessionCookies('user1', 'http://localhost:3000');
  console.log('Cookie count:', cookies.length); // Should print: 1
  console.log('Cookie key:', cookies[0].key); // Should print: "sessionToken"
  console.log('Cookie value:', cookies[0].value); // Should print: "abc123"
});
```

**Expected Output**:
- Returns array with 1 cookie object
- Cookie has key "sessionToken" and value "abc123"

### Test 4: Session Cleanup { .test-case @test }

**Test File**: `session-manager.test.js`

```javascript
const SessionManager = require('./session-manager');
const manager = new SessionManager();

manager.createSession('user1');

manager.makeRequest('user1', 'http://localhost:3000/login', (err, res, body) => {
  // Clear the session
  manager.clearSession('user1');

  // Try to access protected resource after clearing
  manager.makeRequest('user1', 'http://localhost:3000/protected', (err, res, body) => {
    console.log('Status code:', res.statusCode); // Should print: 401
  });
});
```

**Expected Output**:
- After clearing session, cookies are no longer sent
- Request returns status code 401 (Unauthorized)

## Dependencies { .dependencies }

### request { .dependency }

HTTP client library for making requests with cookie support.

## Constraints

- Use Node.js standard callback pattern for asynchronous operations
- Each session must maintain its own independent cookie storage
- Cookies should automatically be included in subsequent requests to the same domain
- Session IDs can be any string value

## Notes

- You may assume all requests are to HTTP (not HTTPS) endpoints
- You don't need to handle cookie expiration or complex cookie attributes
- Focus on correct session isolation and cookie persistence

# HTTP Methods

The request package provides convenience functions for all standard HTTP methods. Each function is a wrapper around the main `request()` function with the HTTP method pre-configured.

## Capabilities

### Main Request Function

The primary function for making HTTP requests. Accepts a URI string or options object.

```javascript { .api }
/**
 * Make an HTTP request
 * @param uri - Target URI string or options object containing uri property
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance (readable and writable stream)
 */
function request(
  uri: string | object,
  options?: object,
  callback?: function
): Request;
```

**Usage Examples:**

```javascript
// Simple string URI
request('http://example.com', function(error, response, body) {
  console.log(body);
});

// With options object
request({
  uri: 'http://example.com',
  method: 'GET',
  headers: { 'User-Agent': 'my-app' }
}, callback);

// URI + options
request('http://example.com', {
  headers: { 'User-Agent': 'my-app' }
}, callback);

// Without callback (streaming)
request('http://example.com').pipe(process.stdout);
```

### GET Method

Perform an HTTP GET request.

```javascript { .api }
/**
 * Perform HTTP GET request
 * @param uri - Target URI string or options object
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance
 */
function request.get(
  uri: string | object,
  options?: object,
  callback?: function
): Request;
```

**Usage Examples:**

```javascript
request.get('http://api.example.com/users', function(error, response, body) {
  console.log('Users:', body);
});

// With query parameters
request.get({
  uri: 'http://api.example.com/users',
  qs: { page: 1, limit: 10 }
}, callback);
```

### POST Method

Perform an HTTP POST request. Commonly used for creating resources or submitting data.

```javascript { .api }
/**
 * Perform HTTP POST request
 * @param uri - Target URI string or options object
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance
 */
function request.post(
  uri: string | object,
  options?: object,
  callback?: function
): Request;
```

**Usage Examples:**

```javascript
// Post JSON data
request.post({
  uri: 'http://api.example.com/users',
  json: true,
  body: { name: 'Alice', email: 'alice@example.com' }
}, function(error, response, body) {
  console.log('Created user:', body);
});

// Post form data
request.post({
  uri: 'http://service.com/upload',
  form: { key: 'value', name: 'John' }
}, callback);

// Post multipart form data
request.post({
  uri: 'http://service.com/upload',
  formData: {
    field: 'value',
    file: fs.createReadStream('file.txt')
  }
}, callback);
```

### PUT Method

Perform an HTTP PUT request. Commonly used for updating resources.

```javascript { .api }
/**
 * Perform HTTP PUT request
 * @param uri - Target URI string or options object
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance
 */
function request.put(
  uri: string | object,
  options?: object,
  callback?: function
): Request;
```

**Usage Examples:**

```javascript
// Update resource with JSON
request.put({
  uri: 'http://api.example.com/users/123',
  json: true,
  body: { name: 'Bob', email: 'bob@example.com' }
}, function(error, response, body) {
  console.log('Updated user:', body);
});

// Stream file to PUT endpoint
fs.createReadStream('file.json').pipe(
  request.put('http://api.example.com/upload')
);
```

### PATCH Method

Perform an HTTP PATCH request. Commonly used for partial resource updates.

```javascript { .api }
/**
 * Perform HTTP PATCH request
 * @param uri - Target URI string or options object
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance
 */
function request.patch(
  uri: string | object,
  options?: object,
  callback?: function
): Request;
```

**Usage Examples:**

```javascript
// Partial update with JSON
request.patch({
  uri: 'http://api.example.com/users/123',
  json: true,
  body: { email: 'newemail@example.com' }
}, function(error, response, body) {
  console.log('Patched user:', body);
});
```

### DELETE Method

Perform an HTTP DELETE request. Available as both `del` and `delete` (since `delete` is a reserved keyword in JavaScript).

```javascript { .api }
/**
 * Perform HTTP DELETE request
 * @param uri - Target URI string or options object
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance
 */
function request.del(
  uri: string | object,
  options?: object,
  callback?: function
): Request;

/**
 * Perform HTTP DELETE request (alias for del)
 * @param uri - Target URI string or options object
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance
 */
function request.delete(
  uri: string | object,
  options?: object,
  callback?: function
): Request;
```

**Usage Examples:**

```javascript
// Delete resource
request.del('http://api.example.com/users/123', function(error, response, body) {
  if (!error && response.statusCode === 204) {
    console.log('User deleted successfully');
  }
});

// Using 'delete' alias
request.delete({
  uri: 'http://api.example.com/posts/456',
  headers: { 'Authorization': 'Bearer token123' }
}, callback);
```

### HEAD Method

Perform an HTTP HEAD request. Retrieves headers without the response body.

```javascript { .api }
/**
 * Perform HTTP HEAD request
 * @param uri - Target URI string or options object
 * @param options - Optional configuration object
 * @param callback - Optional callback function(error, response, body)
 * @returns Request instance
 * @note HEAD requests must not include a request body
 */
function request.head(
  uri: string | object,
  options?: object,
  callback?: function
): Request;
```

**Usage Examples:**

```javascript
// Check if resource exists
request.head('http://example.com/file.pdf', function(error, response, body) {
  if (!error && response.statusCode === 200) {
    console.log('File exists');
    console.log('Content-Length:', response.headers['content-length']);
    console.log('Content-Type:', response.headers['content-type']);
  }
});

// Get metadata without downloading content
request.head({
  uri: 'http://example.com/large-file.zip'
}, function(error, response) {
  console.log('File size:', response.headers['content-length']);
  console.log('Last modified:', response.headers['last-modified']);
});
```

## Parameter Flexibility

All HTTP method functions support three parameter patterns:

**Pattern 1: URI only with callback**
```javascript
request.get('http://example.com', callback);
```

**Pattern 2: URI + options + callback**
```javascript
request.post('http://example.com', { json: true, body: data }, callback);
```

**Pattern 3: Options object only (with uri property) + callback**
```javascript
request.put({ uri: 'http://example.com', body: data }, callback);
```

**Pattern 4: Without callback (for streaming)**
```javascript
request.get('http://example.com').pipe(outputStream);
```

## Return Value

All HTTP method functions return a Request instance, which is a Node.js Stream that is both readable and writable. This allows for:

- **Event handling**: Listen to 'response', 'data', 'error', 'complete', etc.
- **Streaming**: Pipe to/from the request
- **Method chaining**: Call additional methods on the returned instance

```javascript
const req = request.get('http://example.com')
  .on('response', function(response) {
    console.log('Status:', response.statusCode);
  })
  .on('error', function(err) {
    console.error('Error:', err);
  })
  .pipe(fs.createWriteStream('output.html'));
```

## Error Handling

All methods handle errors in the same way:

**Via callback (first parameter):**
```javascript
request.get('http://example.com', function(error, response, body) {
  if (error) {
    console.error('Request failed:', error.message);
    return;
  }
  // Process response
});
```

**Via event listener:**
```javascript
request.get('http://example.com')
  .on('error', function(error) {
    console.error('Request failed:', error);
  })
  .pipe(outputStream);
```

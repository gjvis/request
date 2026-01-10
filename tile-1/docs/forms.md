# Forms

Request provides built-in support for URL-encoded forms (application/x-www-form-urlencoded) and multipart form data (multipart/form-data) for file uploads.

## Capabilities

### URL-Encoded Forms

Set the request body to URL-encoded form data.

```javascript { .api }
/**
 * Sets URL-encoded form data (application/x-www-form-urlencoded)
 * @param {object|FormData} form - Form data object or FormData instance
 * @returns {FormData} FormData instance when called without arguments
 */
Request.prototype.form(form);
```

**Usage with Options:**

```javascript
const request = require('request');

// POST with form option
request.post('http://service.com/upload', {
  form: {
    key: 'value',
    name: 'John',
    email: 'john@example.com'
  }
}, function(err, response, body) {
  console.log('Response:', body);
});

// Equivalent with json option in URL
request.post({
  url: 'http://service.com/upload',
  form: { key: 'value' }
}, function(err, response, body) {
  console.log(body);
});
```

**Usage with Method Chaining:**

```javascript
// Using .form() method
request.post('http://service.com/upload')
  .form({ key: 'value', name: 'data' })
  .on('response', function(response) {
    console.log('Status:', response.statusCode);
  });
```

**Advanced Form Usage:**

```javascript
// Access FormData instance for advanced usage
const req = request.post('http://service.com/upload');
const form = req.form();

// Manually add form fields
form.append('field1', 'value1');
form.append('field2', 'value2');
form.append('nested[key]', 'nested value');

req.on('response', function(response) {
  console.log('Uploaded');
});
```

### Multipart Form Data

Set the request body to multipart/form-data format, typically used for file uploads.

```javascript { .api }
/**
 * Sets multipart/form-data request body
 * @param {Array} multipart - Array of multipart sections
 */
Request.prototype.multipart(multipart);
```

**Usage with formData Option:**

The `formData` option is the recommended way to handle multipart uploads:

```javascript
const fs = require('fs');
const request = require('request');

// Upload file with multipart
const formData = {
  // Simple key-value pairs
  name: 'My File',
  description: 'A test file',

  // File from path
  file: fs.createReadStream('document.pdf'),

  // Multiple files
  attachments: [
    fs.createReadStream('file1.jpg'),
    fs.createReadStream('file2.jpg')
  ],

  // Buffer data
  buffer: Buffer.from([1, 2, 3]),

  // File with metadata
  custom_file: {
    value: fs.createReadStream('data.json'),
    options: {
      filename: 'data.json',
      contentType: 'application/json'
    }
  }
};

request.post({
  url: 'http://service.com/upload',
  formData: formData
}, function(err, response, body) {
  if (err) {
    console.error('Upload failed:', err);
    return;
  }
  console.log('Upload successful:', body);
});
```

**Usage with multipart Method:**

```javascript
// Using .multipart() method with array format
request.post('http://service.com/upload')
  .multipart([
    {
      'Content-Disposition': 'form-data; name="field1"',
      body: 'value1'
    },
    {
      'Content-Disposition': 'form-data; name="file"; filename="test.txt"',
      'Content-Type': 'text/plain',
      body: fs.createReadStream('test.txt')
    }
  ])
  .on('response', function(response) {
    console.log('Upload complete');
  });
```

**Usage with multipart Option:**

```javascript
// Using multipart option in request config
request({
  method: 'POST',
  uri: 'http://service.com/upload',
  multipart: [
    {
      'Content-Disposition': 'form-data; name="data"',
      'Content-Type': 'application/json',
      body: JSON.stringify({ key: 'value' })
    },
    {
      'Content-Disposition': 'form-data; name="file"; filename="doc.pdf"',
      'Content-Type': 'application/pdf',
      body: fs.createReadStream('document.pdf')
    }
  ]
}, function(err, response, body) {
  console.log('Uploaded');
});
```

### Multipart Related

For multipart/related (e.g., for APIs that need multiple related parts), use the multipart API with specific content types:

```javascript
const request = require('request');

request({
  method: 'POST',
  preambleCRLF: true,
  postambleCRLF: true,
  uri: 'http://service.com/upload',
  multipart: [
    {
      'Content-Type': 'application/json',
      body: JSON.stringify({ metadata: 'value' })
    },
    {
      'Content-Type': 'text/plain',
      body: 'Plain text content'
    }
  ]
}, function(err, response, body) {
  console.log('Multipart related uploaded');
});
```

### JSON Body

Set JSON body and automatically set Content-Type header.

```javascript { .api }
/**
 * Sets JSON request body and Content-Type header
 * @param {any} val - Value to JSON stringify and send
 */
Request.prototype.json(val);
```

**Usage with json Option:**

```javascript
// POST JSON data
request.post({
  uri: 'http://api.example.com/users',
  json: {
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
  }
}, function(err, response, body) {
  // body is automatically parsed as JSON
  console.log('Created user:', body);
});

// json: true automatically parses JSON responses
request.get({
  uri: 'http://api.example.com/users/123',
  json: true
}, function(err, response, body) {
  // body is parsed JSON object
  console.log('User:', body.name);
});
```

**Usage with json Method:**

```javascript
// Using .json() method
request.post('http://api.example.com/data')
  .json({ key: 'value', data: [1, 2, 3] })
  .on('response', function(response) {
    console.log('Status:', response.statusCode);
  });
```

### Query String Parameters

Set query string parameters.

```javascript { .api }
/**
 * Sets query string parameters
 * @param {object} q - Query string object
 * @param {boolean} [clobber] - Whether to replace existing query string
 */
Request.prototype.qs(q, clobber);
```

**Usage with qs Option:**

```javascript
// Add query parameters
request.get({
  uri: 'http://api.example.com/search',
  qs: {
    q: 'nodejs',
    page: 1,
    limit: 10,
    sort: 'date'
  }
}, function(err, response, body) {
  console.log(body);
});
// Requests: http://api.example.com/search?q=nodejs&page=1&limit=10&sort=date

// Array values
request.get({
  uri: 'http://api.example.com/items',
  qs: {
    id: [1, 2, 3],
    tags: ['nodejs', 'javascript']
  }
}, function(err, response, body) {
  console.log(body);
});
// Requests: http://api.example.com/items?id[0]=1&id[1]=2&id[2]=3&tags[0]=nodejs&tags[1]=javascript
```

**Usage with qs Method:**

```javascript
// Using .qs() method
request.get('http://api.example.com/search')
  .qs({ q: 'test', page: 2 })
  .on('response', function(response) {
    console.log('Search complete');
  });

// Clobber existing query string
request.get('http://api.example.com/search?existing=param')
  .qs({ new: 'param' }, true) // true = replace existing
  .on('response', function(response) {
    console.log('Queried');
  });
```

**Custom Query String Options:**

```javascript
// Use querystring module instead of qs
request.get({
  uri: 'http://api.example.com/data',
  qs: { key: 'value' },
  useQuerystring: true
}, function(err, response, body) {
  console.log(body);
});

// Custom stringify options
request.get({
  uri: 'http://api.example.com/data',
  qs: { array: [1, 2, 3] },
  qsStringifyOptions: {
    arrayFormat: 'brackets' // array[]=1&array[]=2&array[]=3
  }
}, function(err, response, body) {
  console.log(body);
});
```

## Form Examples

### Complete Form Upload Example

```javascript
const request = require('request');
const fs = require('fs');

// Complete file upload with progress
const formData = {
  name: 'Document Upload',
  category: 'reports',
  file: {
    value: fs.createReadStream('report.pdf'),
    options: {
      filename: 'monthly-report.pdf',
      contentType: 'application/pdf',
      knownLength: fs.statSync('report.pdf').size
    }
  }
};

const req = request.post({
  url: 'http://api.example.com/documents',
  formData: formData
}, function(err, response, body) {
  if (err) {
    console.error('Upload failed:', err);
    return;
  }
  console.log('Status:', response.statusCode);
  console.log('Response:', body);
});

// Track upload progress (approximate)
let uploaded = 0;
req.on('data', function(chunk) {
  uploaded += chunk.length;
  console.log('Uploaded:', uploaded, 'bytes');
});
```

### Form with Authentication

```javascript
// Form upload with Bearer token
request.post({
  url: 'http://api.example.com/upload',
  formData: {
    file: fs.createReadStream('data.csv'),
    metadata: JSON.stringify({ type: 'csv', version: 1 })
  },
  headers: {
    'Authorization': 'Bearer your-token-here'
  }
}, function(err, response, body) {
  console.log('Upload complete');
});
```

### Multiple File Upload

```javascript
const fs = require('fs');
const request = require('request');

// Upload multiple files at once
const formData = {
  title: 'My Album',
  photos: [
    fs.createReadStream('photo1.jpg'),
    fs.createReadStream('photo2.jpg'),
    fs.createReadStream('photo3.jpg')
  ]
};

request.post({
  url: 'http://api.example.com/albums',
  formData: formData
}, function(err, response, body) {
  console.log('Album uploaded:', body);
});
```

### Dynamic Form Data

```javascript
// Build form data dynamically
const formData = {
  timestamp: Date.now(),
  user: 'john_doe'
};

// Add files conditionally
const files = ['file1.txt', 'file2.txt', 'file3.txt'];
files.forEach(function(filename, index) {
  if (fs.existsSync(filename)) {
    formData['file' + index] = fs.createReadStream(filename);
  }
});

request.post({
  url: 'http://api.example.com/batch-upload',
  formData: formData
}, function(err, response, body) {
  console.log('Batch upload complete');
});
```

## Request Body Options

Summary of body-related options:

```javascript { .api }
interface BodyOptions {
  /** Raw request body (string, Buffer, or Stream) */
  body?: string | Buffer | Stream;

  /** URL-encoded form data (application/x-www-form-urlencoded) */
  form?: object;

  /** Multipart form data (multipart/form-data) */
  formData?: object;

  /** Multipart sections array */
  multipart?: Array<object>;

  /** JSON body (automatically stringified, sets Content-Type) */
  json?: any | boolean;

  /** Query string parameters */
  qs?: object;

  /** Use querystring module instead of qs */
  useQuerystring?: boolean;

  /** Options for qs.stringify */
  qsStringifyOptions?: object;

  /** Options for qs.parse */
  qsParseOptions?: object;

  /** Add CRLF before multipart boundary */
  preambleCRLF?: boolean;

  /** Add CRLF after multipart boundary */
  postambleCRLF?: boolean;
}
```

## Important Notes

1. **Mutually Exclusive**: Only use one of `body`, `form`, `formData`, `multipart`, or `json` per request

2. **Content-Type**:
   - `form` sets Content-Type to `application/x-www-form-urlencoded`
   - `formData`/`multipart` sets Content-Type to `multipart/form-data`
   - `json` sets Content-Type to `application/json`

3. **File Streams**: When piping file streams, they are consumed. Create new streams if you need to retry uploads

4. **Memory**: `formData` and `multipart` stream files efficiently without loading them entirely into memory

5. **Form Method**: Calling `.form()` without arguments returns the FormData instance for manual manipulation

# Body Handling

The request package supports multiple formats for sending request bodies including URL-encoded forms, multipart form data, JSON, and raw data streams.

## Capabilities

### URL-Encoded Forms

Send application/x-www-form-urlencoded form data.

```javascript { .api }
/**
 * Set URL-encoded form body or get FormData object
 * @param data - Form data object (if provided, sets form body)
 * @returns FormData object (if no data provided) or Request instance (if data provided)
 */
function Request.prototype.form(data?: object): FormData | Request;
```

**Usage Examples:**

```javascript
// Using form option
request.post({
  uri: 'http://service.com/login',
  form: {
    username: 'user',
    password: 'pass'
  }
}, callback);

// Using form() method
request
  .post('http://service.com/login')
  .form({ username: 'user', password: 'pass' });

// Complex form data
request.post({
  uri: 'http://service.com/submit',
  form: {
    name: 'John Doe',
    email: 'john@example.com',
    age: 30,
    subscribed: true,
    tags: ['nodejs', 'javascript']
  }
}, callback);

// Get form object for manual manipulation
const req = request.post('http://service.com/upload');
const form = req.form();
form.append('field1', 'value1');
form.append('field2', 'value2');
```

### Multipart Form Data

Send multipart/form-data, commonly used for file uploads.

```javascript { .api }
/**
 * Set multipart/form-data body
 * @param data - Multipart data array or options object
 * @returns Request instance for chaining
 */
function Request.prototype.multipart(data: array | object): Request;
```

**FormData Option (Recommended for Files):**

```javascript { .api }
/**
 * Multipart form data using formData option
 */
formData: object
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Using formData option (recommended)
request.post({
  uri: 'http://service.com/upload',
  formData: {
    // Simple field
    name: 'file-name',
    // File from stream
    file: fs.createReadStream('document.pdf'),
    // Multiple files
    attachments: [
      fs.createReadStream('file1.jpg'),
      fs.createReadStream('file2.jpg')
    ],
    // Buffer
    custom: Buffer.from('custom data'),
    // File with custom options
    avatar: {
      value: fs.createReadStream('avatar.png'),
      options: {
        filename: 'avatar.png',
        contentType: 'image/png'
      }
    }
  }
}, function(error, response, body) {
  if (error) {
    console.error('Upload failed:', error);
  } else {
    console.log('Upload successful');
  }
});

// Using multipart option (lower-level)
request.post({
  uri: 'http://service.com/upload',
  multipart: [
    {
      'Content-Disposition': 'form-data; name="field"',
      'body': 'value'
    },
    {
      'Content-Disposition': 'form-data; name="file"; filename="file.txt"',
      'Content-Type': 'text/plain',
      'body': fs.createReadStream('file.txt')
    }
  ]
}, callback);

// Chunked multipart
request.post({
  uri: 'http://service.com/upload',
  multipart: {
    chunked: true,
    data: [
      {
        'Content-Type': 'application/json',
        'body': JSON.stringify({ key: 'value' })
      },
      {
        'Content-Type': 'text/plain',
        'body': 'plain text data'
      }
    ]
  }
}, callback);
```

**Multipart Options:**

```javascript { .api }
interface MultipartOptions {
  preambleCRLF?: boolean;
  postambleCRLF?: boolean;
}
```

- **preambleCRLF** (boolean): Add CRLF before boundary
- **postambleCRLF** (boolean): Add CRLF after boundary

```javascript
request.post({
  uri: 'http://service.com/upload',
  formData: { file: fs.createReadStream('file.txt') },
  preambleCRLF: true,
  postambleCRLF: true
}, callback);
```

### JSON Bodies

Send and receive JSON data with automatic serialization and parsing.

```javascript { .api }
/**
 * Set JSON body and mark response for JSON parsing
 * @param val - Value to JSON-serialize as body
 * @returns Request instance for chaining
 */
function Request.prototype.json(val: any): Request;

/**
 * JSON option: boolean or value to serialize
 */
json: boolean | any;

/**
 * Custom JSON replacer function
 */
jsonReplacer: function;

/**
 * Custom JSON reviver function
 */
jsonReviver: function;
```

**Usage Examples:**

```javascript
// Send and receive JSON
request.post({
  uri: 'http://api.example.com/users',
  json: true,
  body: {
    name: 'Alice',
    email: 'alice@example.com',
    age: 25
  }
}, function(error, response, body) {
  // body is automatically parsed as JSON
  console.log('Created user ID:', body.id);
});

// Shorthand syntax
request.post({
  uri: 'http://api.example.com/users',
  json: {
    name: 'Bob',
    email: 'bob@example.com'
  }
}, callback);

// Using method syntax
request
  .post('http://api.example.com/users')
  .json({ name: 'Charlie', email: 'charlie@example.com' });

// With custom replacer
request.post({
  uri: 'http://api.example.com/data',
  json: true,
  body: {
    publicData: 'visible',
    privateData: 'hidden',
    timestamp: new Date()
  },
  jsonReplacer: function(key, value) {
    // Filter out private fields
    if (key === 'privateData') return undefined;
    // Convert dates to ISO strings
    if (value instanceof Date) return value.toISOString();
    return value;
  }
}, callback);

// With custom reviver
request.get({
  uri: 'http://api.example.com/data',
  json: true,
  jsonReviver: function(key, value) {
    // Parse ISO date strings back to Date objects
    if (typeof value === 'string' && /^\d{4}-\d{2}-\d{2}T/.test(value)) {
      return new Date(value);
    }
    return value;
  }
}, function(error, response, body) {
  console.log(body.createdAt instanceof Date); // true
});
```

### Raw Body

Send raw request body as string, Buffer, or stream.

```javascript { .api }
/**
 * Raw body option
 */
body: string | Buffer | Stream;
```

**Usage Examples:**

```javascript
// String body
request.post({
  uri: 'http://api.example.com/data',
  body: 'raw string data',
  headers: {
    'Content-Type': 'text/plain'
  }
}, callback);

// Buffer body
const buffer = Buffer.from('binary data', 'utf8');
request.post({
  uri: 'http://api.example.com/upload',
  body: buffer,
  headers: {
    'Content-Type': 'application/octet-stream'
  }
}, callback);

// Stream body
const fs = require('fs');
request.post({
  uri: 'http://api.example.com/upload',
  body: fs.createReadStream('largefile.dat'),
  headers: {
    'Content-Type': 'application/octet-stream',
    'Content-Length': fs.statSync('largefile.dat').size
  }
}, callback);

// XML body
const xml = '<?xml version="1.0"?><root><item>value</item></root>';
request.post({
  uri: 'http://api.example.com/xml-endpoint',
  body: xml,
  headers: {
    'Content-Type': 'application/xml'
  }
}, callback);
```

### Query String in Request Body

Use query string parameters in the request body.

```javascript { .api }
/**
 * Query string object for body
 */
qs: object;
```

**Usage Examples:**

```javascript
// Add query string to URL
request.get({
  uri: 'http://api.example.com/search',
  qs: {
    q: 'nodejs',
    page: 1,
    limit: 10
  }
}, callback);
// Results in: http://api.example.com/search?q=nodejs&page=1&limit=10

// Query string method
request
  .get('http://api.example.com/search')
  .qs({ q: 'nodejs', page: 1 });
```

### Streaming Request Bodies

Stream data to the request body for efficient handling of large files.

```javascript
const fs = require('fs');

// Pipe file to PUT request
fs.createReadStream('largefile.dat')
  .pipe(request.put('http://api.example.com/upload'));

// Pipe with error handling
fs.createReadStream('file.txt')
  .on('error', function(err) {
    console.error('Read error:', err);
  })
  .pipe(request.put('http://api.example.com/upload'))
  .on('error', function(err) {
    console.error('Upload error:', err);
  })
  .on('complete', function(response) {
    console.log('Upload complete:', response.statusCode);
  });

// Pipe request to request
request.get('http://source.com/file.pdf')
  .pipe(request.put('http://dest.com/file.pdf'));

// Write chunks manually
const req = request.post('http://api.example.com/stream');
req.write('chunk1\n');
req.write('chunk2\n');
req.write('chunk3\n');
req.end();
```

## Body Handling with Response

### Response Body Encoding

```javascript
// Get response as string
request({
  uri: 'http://example.com/text',
  encoding: 'utf8'
}, function(error, response, body) {
  console.log(typeof body); // 'string'
});

// Get response as Buffer
request({
  uri: 'http://example.com/binary',
  encoding: null
}, function(error, response, body) {
  console.log(Buffer.isBuffer(body)); // true
});
```

### Streaming Response Bodies

```javascript
const fs = require('fs');

// Stream response to file
request('http://example.com/file.pdf')
  .pipe(fs.createWriteStream('output.pdf'));

// Transform stream
const zlib = require('zlib');
request('http://example.com/data.json')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('data.json'));

// Process chunks
request('http://example.com/stream')
  .on('data', function(chunk) {
    console.log('Received', chunk.length, 'bytes');
  })
  .on('end', function() {
    console.log('Stream complete');
  });
```

## Complete Usage Example

```javascript
const request = require('request');
const fs = require('fs');

// Upload file with form fields
request.post({
  uri: 'http://api.example.com/uploads',
  formData: {
    // Text fields
    title: 'My Document',
    description: 'Important file',
    category: 'documents',
    // File upload
    file: {
      value: fs.createReadStream('document.pdf'),
      options: {
        filename: 'document.pdf',
        contentType: 'application/pdf'
      }
    },
    // JSON metadata
    metadata: {
      value: JSON.stringify({
        author: 'John Doe',
        createdAt: new Date().toISOString()
      }),
      options: {
        contentType: 'application/json'
      }
    }
  },
  headers: {
    'Authorization': 'Bearer token123'
  },
  timeout: 30000
}, function(error, response, body) {
  if (error) {
    console.error('Upload failed:', error);
  } else if (response.statusCode === 201) {
    console.log('Upload successful:', body);
  } else {
    console.error('Upload failed with status:', response.statusCode);
  }
});
```

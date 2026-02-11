# Forms and File Uploads

Support for URL-encoded forms, multipart form data, and file uploads.

## Capabilities

### URL-Encoded Forms

Submit form data with `application/x-www-form-urlencoded` encoding.

```javascript { .api }
/**
 * URL-encoded form options
 */
interface FormOption {
  /** Form data as object or string */
  form?: object | string;
}
```

**Usage Examples:**

```javascript
// Form as object
request.post('http://service.com/upload', {
  form: {
    key: 'value',
    name: 'John Doe',
    email: 'john@example.com'
  }
}, callback);

// Form as string
request.post({
  url: 'http://service.com/upload',
  form: 'key=value&name=John%20Doe'
}, callback);

// Using .form() method
request.post('http://service.com/upload')
  .form({key: 'value'})
  .on('response', function(response) {
    console.log('Uploaded!');
  });
```

### Multipart Form Data

Submit multipart form data with file uploads using `multipart/form-data` encoding.

```javascript { .api }
/**
 * Multipart form data options
 */
interface FormDataOption {
  /** Multipart form data with file support */
  formData?: object;
}

/**
 * FormData field value can be:
 * - Simple value (string, number, Buffer)
 * - Stream (for files)
 * - Array of values
 * - Object with value and options
 */
type FormDataValue =
  | string
  | number
  | Buffer
  | Stream
  | Array<FormDataValue>
  | FormDataValueWithOptions;

interface FormDataValueWithOptions {
  /** Field value */
  value: string | Buffer | Stream;
  /** Field options */
  options: {
    /** Filename for the field */
    filename?: string;
    /** Content type */
    contentType?: string;
    /** Known file size */
    knownLength?: number;
  };
}
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Simple multipart form
request.post({
  url: 'http://service.com/upload',
  formData: {
    my_field: 'my_value',
    my_number: 123
  }
}, callback);

// File upload from filesystem
request.post({
  url: 'http://service.com/upload',
  formData: {
    file: fs.createReadStream('photo.jpg'),
    description: 'My photo'
  }
}, callback);

// Buffer upload
const buffer = Buffer.from([1, 2, 3]);
request.post({
  url: 'http://service.com/upload',
  formData: {
    my_buffer: buffer
  }
}, callback);

// Multiple files
request.post({
  url: 'http://service.com/upload',
  formData: {
    attachments: [
      fs.createReadStream('file1.jpg'),
      fs.createReadStream('file2.jpg')
    ]
  }
}, callback);

// File with custom metadata
request.post({
  url: 'http://service.com/upload',
  formData: {
    custom_file: {
      value: fs.createReadStream('/dev/urandom'),
      options: {
        filename: 'topsecret.jpg',
        contentType: 'image/jpeg'
      }
    }
  }
}, callback);

// Mixed form data
request.post({
  url: 'http://service.com/upload',
  formData: {
    name: 'John Doe',
    email: 'john@example.com',
    avatar: fs.createReadStream('avatar.png'),
    attachments: [
      fs.createReadStream('doc1.pdf'),
      fs.createReadStream('doc2.pdf')
    ]
  }
}, function(err, response, body) {
  if (err) {
    return console.error('upload failed:', err);
  }
  console.log('Upload successful!', body);
});
```

### Advanced Multipart with form-data API

Access the underlying form-data object for advanced use cases.

```javascript { .api }
/**
 * Get FormData instance for manual manipulation
 * @returns FormData instance from form-data module
 */
interface Request {
  form(): FormData;
}

interface FormData {
  append(key: string, value: any, options?: object): void;
  getHeaders(): object;
  getLength(callback: (err: Error | null, length: number) => void): void;
}
```

**Usage Example:**

```javascript
const fs = require('fs');

// Advanced form manipulation
const r = request.post('http://service.com/upload', function(err, response, body) {
  console.log('Server response:', body);
});

const form = r.form();
form.append('my_field', 'my_value');
form.append('my_buffer', Buffer.from([1, 2, 3]));
form.append('custom_file', fs.createReadStream('unicycle.jpg'), {
  filename: 'unicycle.jpg',
  contentType: 'image/jpeg'
});
```

### Multipart Related Requests

Create `multipart/related` requests with custom parts and boundaries.

```javascript { .api }
/**
 * Multipart/related configuration
 */
interface MultipartOption {
  /** Array of multipart parts */
  multipart?: Array<MultipartPart> | MultipartConfig;
  /** Add CRLF before boundary */
  preambleCRLF?: boolean;
  /** Add CRLF after boundary */
  postambleCRLF?: boolean;
}

interface MultipartPart {
  /** Content-Type for this part */
  'content-type'?: string;
  /** Part body (string, Buffer, or Stream) */
  body: string | Buffer | Stream;
}

interface MultipartConfig {
  /** Use chunked transfer encoding */
  chunked?: boolean;
  /** Array of multipart parts */
  data: Array<MultipartPart>;
}
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Multipart/related with array
request({
  method: 'PUT',
  uri: 'http://service.com/upload',
  multipart: [
    {
      'content-type': 'application/json',
      body: JSON.stringify({
        foo: 'bar',
        _attachments: {
          'message.txt': {
            follows: true,
            length: 18,
            'content_type': 'text/plain'
          }
        }
      })
    },
    {body: 'I am an attachment'},
    {body: fs.createReadStream('image.png')}
  ]
}, callback);

// Multipart/related with config object
request({
  method: 'PUT',
  uri: 'http://service.com/upload',
  multipart: {
    chunked: false,
    data: [
      {
        'content-type': 'application/json',
        body: JSON.stringify({foo: 'bar'})
      },
      {body: 'I am an attachment'}
    ]
  }
}, callback);

// With boundary CRLF options (for .NET WebAPI compatibility)
request({
  method: 'PUT',
  preambleCRLF: true,
  postambleCRLF: true,
  uri: 'http://service.com/upload',
  multipart: [
    {
      'content-type': 'application/json',
      body: JSON.stringify({foo: 'bar'})
    },
    {body: 'I am an attachment'}
  ]
}, callback);
```

### Form Methods on Request Instance

Chainable methods for configuring forms on a Request instance.

```javascript { .api }
/**
 * Set URL-encoded form data
 * If form parameter is provided, sets the form data
 * If no parameter, returns FormData instance for manual manipulation
 * @param form - Form data as object or string
 * @returns this (if form provided) or FormData instance
 */
interface Request {
  form(form?: object | string): Request | FormData;
}

/**
 * Set multipart request data
 * @param multipart - Multipart configuration
 * @returns this
 */
interface Request {
  multipart(multipart: Array<MultipartPart> | MultipartConfig): Request;
}
```

**Usage Examples:**

```javascript
// Chainable form method
request.post('http://service.com/upload')
  .form({key: 'value'})
  .on('response', function(response) {
    console.log('Response received');
  });

// Get FormData instance
const req = request.post('http://service.com/upload', callback);
const formData = req.form();
formData.append('field1', 'value1');
formData.append('field2', 'value2');

// Chainable multipart method
request.put('http://service.com/upload')
  .multipart([
    {'content-type': 'application/json', body: '{"foo":"bar"}'},
    {body: 'attachment data'}
  ])
  .on('response', function(response) {
    console.log('Uploaded');
  });
```

## Types

### Form Data Types

```javascript { .api }
/**
 * Simple form data as key-value pairs
 */
type SimpleFormData = {
  [key: string]: string | number | boolean;
};

/**
 * Multipart form data with file support
 */
type MultipartFormData = {
  [key: string]: FormDataValue;
};

type FormDataValue =
  | string
  | number
  | Buffer
  | Stream
  | Array<string | number | Buffer | Stream | FormDataValueWithOptions>
  | FormDataValueWithOptions;

interface FormDataValueWithOptions {
  value: string | Buffer | Stream;
  options: {
    filename?: string;
    contentType?: string;
    knownLength?: number;
  };
}

/**
 * Multipart/related part
 */
interface MultipartPart {
  'content-type'?: string;
  body: string | Buffer | Stream;
}

/**
 * Multipart/related configuration
 */
interface MultipartConfig {
  chunked?: boolean;
  data: Array<MultipartPart>;
}
```

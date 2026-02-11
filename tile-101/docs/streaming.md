# Streaming and Events

Request instances are Node.js streams supporting piping for both request and response bodies, with comprehensive event handling.

## Capabilities

### Stream Interface

Request instances extend Node.js Stream and can be used as both readable and writable streams.

```javascript { .api }
/**
 * Request instance extends Stream
 */
interface Request extends Stream {
  /** Pipe response to destination stream */
  pipe(dest: Stream, opts?: object): Stream;

  /** Write data to request body */
  write(chunk: string | Buffer, encoding?: string, callback?: function): boolean;

  /** End the request */
  end(chunk?: string | Buffer): void;

  /** Pause response stream */
  pause(): void;

  /** Resume response stream */
  resume(): void;

  /** Abort the request */
  abort(): void;

  /** Destroy request/response */
  destroy(): void;
}
```

### Piping Response to File

Stream response data directly to a file or other writable stream.

**Usage Examples:**

```javascript
const fs = require('fs');

// Pipe response to file
request('http://google.com/doodle.png')
  .pipe(fs.createWriteStream('doodle.png'));

// With error handling
request('http://mysite.com/doodle.png')
  .on('error', function(err) {
    console.error(err);
  })
  .pipe(fs.createWriteStream('doodle.png'));

// Pipe with response event
request('http://google.com/img.png')
  .on('response', function(response) {
    console.log(response.statusCode);  // 200
    console.log(response.headers['content-type']);  // 'image/png'
  })
  .pipe(fs.createWriteStream('img.png'));
```

### Piping Request Body from File

Stream file data as request body for uploads.

**Usage Examples:**

```javascript
const fs = require('fs');

// Pipe file to PUT request
fs.createReadStream('file.json')
  .pipe(request.put('http://mysite.com/obj.json'));

// Request automatically sets Content-Type based on file extension
fs.createReadStream('data.json')
  .pipe(request.put('http://mysite.com/upload'));
  // Content-Type: application/json

// Pipe file to POST
fs.createReadStream('image.png')
  .pipe(request.post('http://mysite.com/upload'));
```

### Piping Request to Request

Pipe response from one request directly to another request.

**Usage Examples:**

```javascript
// GET and PUT through pipe
request.get('http://google.com/img.png')
  .pipe(request.put('http://mysite.com/img.png'));

// Content-Type and Content-Length preserved
request.get('http://source.com/data.json')
  .pipe(request.put('http://dest.com/data.json'));

// One-line proxying
http.createServer(function(req, resp) {
  req.pipe(request('http://mysite.com/doodle.png')).pipe(resp);
});
```

### Stream Proxying

Use request as a proxy by piping through HTTP server.

**Usage Examples:**

```javascript
const http = require('http');

// Simple proxy server
http.createServer(function(req, resp) {
  if (req.url === '/doodle.png') {
    if (req.method === 'PUT') {
      req.pipe(request.put('http://mysite.com/doodle.png'));
    } else if (req.method === 'GET' || req.method === 'HEAD') {
      request.get('http://mysite.com/doodle.png').pipe(resp);
    }
  }
});

// Bi-directional proxy
http.createServer(function(req, resp) {
  if (req.url === '/doodle.png') {
    const x = request('http://mysite.com/doodle.png');
    req.pipe(x);
    x.pipe(resp);
  }
});

// With default options
const r = request.defaults({proxy: 'http://localproxy.com'});
http.createServer(function(req, resp) {
  if (req.url === '/doodle.png') {
    r.get('http://google.com/doodle.png').pipe(resp);
  }
});
```

### Request Events

Request instances emit various events during the request lifecycle.

```javascript { .api }
/**
 * Events emitted by Request instances
 */

/** Emitted on request or response error */
on(event: 'error', listener: (error: Error) => void): this;

/** Emitted when response is received */
on(event: 'response', listener: (response: Response) => void): this;

/** Emitted when request/response cycle completes */
on(event: 'complete', listener: (response: Response, body?: string | Buffer | object) => void): this;

/** Emitted for each response data chunk */
on(event: 'data', listener: (chunk: Buffer | string) => void): this;

/** Emitted when response ends */
on(event: 'end', listener: (chunk?: Buffer | string) => void): this;

/** Emitted when response closes */
on(event: 'close', listener: () => void): this;

/** Emitted when internal request object is created */
on(event: 'request', listener: (req: http.ClientRequest) => void): this;

/** Emitted when socket is assigned */
on(event: 'socket', listener: (socket: net.Socket) => void): this;

/** Emitted when write buffer drains */
on(event: 'drain', listener: () => void): this;

/** Emitted when source is piped to request */
on(event: 'pipe', listener: (src: Stream) => void): this;

/** Emitted when request is aborted */
on(event: 'abort', listener: () => void): this;
```

**Usage Examples:**

```javascript
// Error event
request('http://example.com')
  .on('error', function(err) {
    console.error('Request failed:', err);
  });

// Response event
request('http://example.com')
  .on('response', function(response) {
    console.log('Status:', response.statusCode);
    console.log('Headers:', response.headers);
  });

// Data event (streaming without callback)
request('http://example.com')
  .on('data', function(chunk) {
    console.log('Received chunk:', chunk.length, 'bytes');
  })
  .on('end', function() {
    console.log('Response complete');
  });

// Complete event (no callback provided)
request('http://example.com')
  .on('complete', function(response, body) {
    console.log('Status:', response.statusCode);
    console.log('Body:', body);
  });

// Socket event
request('http://example.com')
  .on('socket', function(socket) {
    console.log('Socket assigned:', socket.remoteAddress);
  });

// Request event
request('http://example.com')
  .on('request', function(req) {
    console.log('HTTP request created');
  });

// Abort event
const req = request('http://example.com')
  .on('abort', function() {
    console.log('Request aborted');
  });

setTimeout(function() {
  req.abort();
}, 1000);
```

### Response Stream Events

When using gzip decompression, the response stream and decompressed content stream are separate.

**Usage Example:**

```javascript
// Gzip with separate streams
request({
  method: 'GET',
  uri: 'http://www.google.com',
  gzip: true
})
.on('data', function(data) {
  // Decompressed data chunks
  console.log('Decoded chunk:', data);
})
.on('response', function(response) {
  // Unmodified response stream (compressed)
  response.on('data', function(data) {
    // Compressed data chunks
    console.log('Received', data.length, 'bytes of compressed data');
  });
});
```

### Stream Encoding

Set encoding for response stream.

```javascript { .api }
/**
 * Encoding options
 */
interface EncodingOption {
  /** Response encoding (default: undefined/utf8, null for Buffer) */
  encoding?: string | null;
}
```

**Usage Examples:**

```javascript
// Default encoding (utf8)
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
  encoding: 'utf16le'
}, function(err, response, body) {
  console.log('UTF-16 string:', body);
});
```

### Pausing and Resuming

Control response stream flow.

```javascript { .api }
/**
 * Pause response stream
 */
interface Request {
  pause(): void;
}

/**
 * Resume response stream
 */
interface Request {
  resume(): void;
}
```

**Usage Example:**

```javascript
const req = request('http://example.com')
  .on('data', function(chunk) {
    console.log('Chunk received');
    req.pause();

    // Process chunk
    setTimeout(function() {
      req.resume();
    }, 1000);
  });
```

### Aborting Requests

Abort an in-progress request.

```javascript { .api }
/**
 * Abort the request
 */
interface Request {
  abort(): void;
}
```

**Usage Examples:**

```javascript
// Abort with timeout
const req = request('http://example.com');

setTimeout(function() {
  req.abort();
}, 5000);

// Abort on condition
const req = request('http://example.com')
  .on('data', function(chunk) {
    if (someCondition) {
      req.abort();
    }
  });

// Abort event
request('http://example.com')
  .on('abort', function() {
    console.log('Request was aborted');
  })
  .abort();
```

### Writing Request Body

Write data to request body manually.

```javascript { .api }
/**
 * Write chunk to request
 * @param chunk - Data to write
 * @param encoding - Encoding (default: 'utf8')
 * @param callback - Callback when write completes
 * @returns true if buffer is empty, false if should wait for drain
 */
interface Request {
  write(chunk: string | Buffer, encoding?: string, callback?: function): boolean;
}

/**
 * End request
 * @param chunk - Final chunk to write
 */
interface Request {
  end(chunk?: string | Buffer): void;
}
```

**Usage Example:**

```javascript
// Manual body writing
const req = request.post('http://example.com/upload');

req.write('chunk1');
req.write('chunk2');
req.write('chunk3');
req.end();

// With encoding
const req = request.post('http://example.com/upload');
req.write('data', 'utf8');
req.end();

// End with final chunk
const req = request.post('http://example.com/upload');
req.write('chunk1');
req.end('final chunk');
```

## Types

### Stream Types

```javascript { .api }
/**
 * Request stream interface
 */
interface Request extends Stream {
  readable: boolean;
  writable: boolean;

  pipe(dest: Stream, opts?: {end?: boolean}): Stream;
  write(chunk: string | Buffer, encoding?: string, callback?: function): boolean;
  end(chunk?: string | Buffer): void;
  pause(): void;
  resume(): void;
  abort(): void;
  destroy(): void;

  on(event: string, listener: function): this;
  once(event: string, listener: function): this;
  removeListener(event: string, listener: function): this;
  emit(event: string, ...args: any[]): boolean;
}

/**
 * Response stream (http.IncomingMessage)
 */
interface Response extends Stream {
  statusCode: number;
  headers: object;

  on(event: 'data', listener: (chunk: Buffer) => void): this;
  on(event: 'end', listener: () => void): this;
  on(event: 'close', listener: () => void): this;
  on(event: 'error', listener: (err: Error) => void): this;

  pipe(dest: Stream, opts?: object): Stream;
  pause(): void;
  resume(): void;
}
```

## Notes

- Request is both readable (response) and writable (request body) stream
- Piping preserves Content-Type and Content-Length headers
- Error handling is critical when piping to prevent crashes
- Response stream with `gzip: true` is decompressed automatically
- `encoding: null` returns response body as Buffer instead of string
- Stream events fire regardless of callback presence
- Aborting a request emits 'abort' event
- When piping, the destination stream is returned (chainable)

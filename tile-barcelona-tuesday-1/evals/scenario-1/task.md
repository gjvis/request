# HTTP Client Utility

Build a command-line utility that downloads a file from a URL while handling cookies, proxies, and streaming efficiently.

## Requirements

### Core Functionality

The utility should download a file from a provided URL and save it to a local path. It must handle the following scenarios:

1. **Cookie Management**: Maintain cookies across redirects. If a cookie file path is provided, persist cookies to disk so they can be reused in subsequent downloads.
2. **Proxy Support**: Automatically detect and use proxy server from HTTP_PROXY environment variable if set.
3. **Streaming**: Stream the response directly to disk without loading the entire file into memory.
4. **Redirect Tracking**: Track and count the number of HTTP redirects followed.

### Input Parameters

Your utility should accept the following parameters:

- `url` (string) - The URL to download from
- `outputPath` (string) - Local file path where content should be saved
- `cookieFile` (string, optional) - Path to persist cookies between runs
- `followRedirects` (boolean, optional) - Whether to follow HTTP redirects (default: true)

### Output

The utility should:
- Save the downloaded content to the specified output path
- Return a summary object containing:
  - `finalUrl` (string) - Final URL after following any redirects
  - `statusCode` (number) - HTTP status code of the final response
  - `bytesDownloaded` (number) - Total bytes written to disk
  - `redirectCount` (number) - Number of redirects followed

### Error Handling

- Handle network errors gracefully and provide meaningful error messages
- Handle HTTP error status codes (4xx, 5xx)
- Validate that required parameters are provided

## Implementation

[@generates](./src/downloader.js)

## API

```javascript { #api }
/**
 * Downloads a file from a URL with cookie persistence and proxy support
 *
 * @param {Object} options - Download options
 * @param {string} options.url - URL to download from
 * @param {string} options.outputPath - Local file path to save content
 * @param {string} [options.cookieFile] - Optional path to persist cookies
 * @param {boolean} [options.followRedirects=true] - Whether to follow redirects
 * @returns {Promise<Object>} Download summary with finalUrl, statusCode, bytesDownloaded, redirectCount
 */
async function download(options);

module.exports = { download };
```

## Test Cases

- Downloads a simple file successfully [@test](../test/downloader.test.js)
- Persists cookies to a file and reuses them on subsequent requests [@test](../test/downloader.test.js)
- Streams large files without excessive memory usage [@test](../test/downloader.test.js)
- Follows redirects and tracks redirect count [@test](../test/downloader.test.js)

## Dependencies { .dependencies }

### request { .dependency }

Provides HTTP client functionality for making requests with cookie persistence, proxy support, and streaming.

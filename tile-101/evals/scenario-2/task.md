# Build a File Upload Utility

Create a Node.js utility that uploads files to a server using multipart form data encoding.

## Requirements

Your utility should:

1. Upload one or more files to a specified endpoint
2. Include additional form fields along with the file(s)
3. Support uploading files from the filesystem
4. Handle multiple file attachments in a single request
5. Set custom filenames for uploaded files
6. Return the server response after upload

## Upload Specification

- Endpoint: `https://httpbin.org/post`
- Method: POST
- Encoding: multipart/form-data
- Should include both file data and text fields

## Example Usage

```javascript
const uploader = require('./file-uploader');

uploader.uploadFiles({
  endpoint: 'https://httpbin.org/post',
  files: {
    avatar: '/path/to/profile.jpg',
    document: '/path/to/resume.pdf'
  },
  fields: {
    username: 'johndoe',
    description: 'Profile update'
  }
}, (err, response) => {
  console.log('Upload complete:', response);
});
```

## Expected Behavior

- Files should be sent as streams
- Form fields should be included in the same multipart request
- Custom filenames should be preserved
- The function should handle multiple files simultaneously

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with built-in support for multipart form data and file uploads.

## Test Cases

### Test 1: Single File Upload @test

Input: One file path with form fields

Expected behavior: Should upload the file as multipart/form-data along with text fields

### Test 2: Multiple Files @test

Input: Multiple file paths in the files object

Expected behavior: Should upload all files in a single multipart request

### Test 3: Custom Filename @test

Input: File with custom filename specification

Expected behavior: Should preserve or set custom filename in the multipart data

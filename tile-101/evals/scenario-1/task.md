# Build a REST API Client for User Management

Create a Node.js module that interacts with a JSON REST API for managing user data.

## Requirements

Your module should provide functions to:

1. Create a new user (POST request with JSON payload)
2. Get user details (GET request, parse JSON response)
3. Update user information (PUT request with JSON payload)
4. Handle JSON parsing automatically for both requests and responses
5. Return parsed JavaScript objects to the caller

## API Specification

The mock API endpoint is: `https://jsonplaceholder.typicode.com`

- Create user: POST to `/users` with JSON body
- Get user: GET from `/users/{id}` returns JSON
- Update user: PUT to `/users/{id}` with JSON body

## Example Usage

```javascript
const userApi = require('./user-api');

// Create user
userApi.createUser({
  name: 'John Doe',
  email: 'john@example.com'
}, (err, user) => {
  console.log('Created:', user);
});

// Get user
userApi.getUser(1, (err, user) => {
  console.log('User:', user.name);
});

// Update user
userApi.updateUser(1, {
  name: 'Jane Doe',
  email: 'jane@example.com'
}, (err, user) => {
  console.log('Updated:', user);
});
```

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client for Node.js with built-in JSON handling capabilities.

## Test Cases

### Test 1: Create User @test

Input: `{ name: 'Test User', email: 'test@example.com' }`

Expected behavior: Should POST JSON data and return parsed response object with user data

### Test 2: Get User @test

Input: `userId = 1`

Expected behavior: Should GET user data and return parsed JavaScript object (not string)

### Test 3: Update User @test

Input: `userId = 1, { name: 'Updated Name' }`

Expected behavior: Should PUT JSON data and return parsed response object with updated data

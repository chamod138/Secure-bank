# Lambda Test Scripts

This document contains the complete test scripts for the Lambda function, separated into AWS Lambda Console events and API Gateway / Postman testing.

## 1. AWS Lambda Console Test Events (Flat JSON)

Use these event payloads in the AWS Lambda console under the **Test** tab. Create a new test event and paste each JSON payload in turn.

### A. Test CORS Preflight (OPTIONS)
```json
{
  "httpMethod": "OPTIONS",
  "headers": {
    "Origin": "https://example.com",
    "Access-Control-Request-Method": "POST",
    "Access-Control-Request-Headers": "Content-Type"
  }
}
```
Expected: `200 OK` with CORS headers.

### B. Test Successful Registration
```json
{
  "httpMethod": "POST",
  "path": "/register",
  "body": {
    "username": "johndoe",
    "password": "StrongPass123!",
    "adminKey": "your-admin-key"
  }
}
```
Expected: `201 Created` with success message.

### C. Test Failed Registration (Wrong Admin Key)
```json
{
  "httpMethod": "POST",
  "path": "/register",
  "body": {
    "username": "janedoe",
    "password": "ValidPass123!",
    "adminKey": "wrong-key"
  }
}
```
Expected: `403 Forbidden` with invalid admin key error.

### D. Test Failed Registration (Weak Password)
```json
{
  "httpMethod": "POST",
  "path": "/register",
  "body": {
    "username": "janedoe",
    "password": "short",
    "adminKey": "your-admin-key"
  }
}
```
Expected: `400 Bad Request` with password strength validation error.

### E. Test Successful Login
Make sure you registered `johndoe` first using Test B.
```json
{
  "httpMethod": "POST",
  "path": "/login",
  "body": {
    "username": "johndoe",
    "password": "StrongPass123!"
  }
}
```
Expected: `200 OK` with `success: true`.

### F. Test Failed Login (Wrong Password)
```json
{
  "httpMethod": "POST",
  "path": "/login",
  "body": {
    "username": "johndoe",
    "password": "WrongPass123!"
  }
}
```
Expected: `200 OK` with `success: false` and `remainingAttempts: 4`.

### G. Test API Gateway Wrapped Event (Real-world Format)
This simulates the event structure API Gateway sends to Lambda.
```json
{
  "resource": "/login",
  "path": "/login",
  "httpMethod": "POST",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": "{\"username\":\"johndoe\",\"password\":\"StrongPass123!\"}",
  "isBase64Encoded": false
}
```

## 2. cURL Scripts (For API Gateway Testing)

If your Lambda function is deployed behind API Gateway, use these commands in your terminal. Replace `<YOUR_API_URL>` with your actual invoke URL.

### Register a new user via API
```bash
curl -X POST "<YOUR_API_URL>/register" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "johndoe",
    "password": "StrongPass123!",
    "adminKey": "your-admin-key"
  }'
```

### Login via API
```bash
curl -X POST "<YOUR_API_URL>/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "johndoe",
    "password": "StrongPass123!"
  }'
```

## 3. Automated Lockout Test Script (Bash)

The account locks after 5 failed login attempts. Use this script to automate the lockout test with 6 attempts.

```bash
#!/bin/bash
URL="<YOUR_API_URL>/login"
for i in {1..6}; do
  echo "Attempt $i"
  curl -X POST "$URL" \
    -H "Content-Type: application/json" \
    -d '{"username":"johndoe","password":"WrongPass123!"}'
  echo -e "\n"
done
```

## 4. Postman Collection Import

Save the following JSON as `auth-tests.json` and import it into Postman.

```json
{
  "info": {
    "name": "Secure Bank Auth Tests",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Register User",
      "request": {
        "method": "POST",
        "header": [
          { "key": "Content-Type", "value": "application/json" }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\"username\":\"johndoe\",\"password\":\"StrongPass123!\",\"adminKey\":\"your-admin-key\"}"
        },
        "url": { "raw": "<YOUR_API_URL>/register" }
      }
    },
    {
      "name": "Login User",
      "request": {
        "method": "POST",
        "header": [
          { "key": "Content-Type", "value": "application/json" }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\"username\":\"johndoe\",\"password\":\"StrongPass123!\"}"
        },
        "url": { "raw": "<YOUR_API_URL>/login" }
      }
    }
  ]
}
```

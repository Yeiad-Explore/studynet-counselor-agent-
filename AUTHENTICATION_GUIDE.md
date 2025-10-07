# JWT Authentication Guide

## Overview

The StudyNet Counselor RAG API now uses **JWT (JSON Web Token)** authentication for all endpoints except public ones (health check, login, register).

## Authentication Endpoints

### Base URL
```
http://localhost:8000/api/auth/
```

## Endpoints

### 1. Register New User

**Endpoint:** `POST /api/auth/register/`

**Public:** Yes (No authentication required)

**Request Body:**
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "password_confirm": "SecurePass123!",
  "first_name": "John",
  "last_name": "Doe"
}
```

**Success Response (201):**
```json
{
  "message": "User registered successfully",
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "john@example.com",
    "first_name": "John",
    "last_name": "Doe"
  },
  "tokens": {
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }
}
```

**Error Responses:**
- `400`: Validation errors (passwords don't match, username exists, etc.)
- `500`: Server error

---

### 2. Login

**Endpoint:** `POST /api/auth/login/`

**Public:** Yes (No authentication required)

**Request Body:**
```json
{
  "username": "john_doe",
  "password": "SecurePass123!"
}
```

**Success Response (200):**
```json
{
  "message": "Login successful",
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "john@example.com",
    "first_name": "John",
    "last_name": "Doe"
  },
  "tokens": {
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
    "access": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }
}
```

**Error Responses:**
- `400`: Missing fields
- `401`: Invalid credentials or account disabled
- `500`: Server error

---

### 3. Refresh Access Token

**Endpoint:** `POST /api/auth/token/refresh/`

**Public:** Yes (No authentication required)

**Request Body:**
```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

**Success Response (200):**
```json
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

**Error Responses:**
- `400`: Missing refresh token
- `401`: Invalid or expired refresh token
- `500`: Server error

---

### 4. Logout

**Endpoint:** `POST /api/auth/logout/`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

**Success Response (200):**
```json
{
  "message": "Logout successful"
}
```

**Error Responses:**
- `400`: Missing or invalid refresh token
- `401`: Unauthorized
- `500`: Server error

---

### 5. Get User Profile

**Endpoint:** `GET /api/auth/profile/`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer <access_token>
```

**Success Response (200):**
```json
{
  "id": 1,
  "username": "john_doe",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "is_active": true,
  "is_staff": false,
  "date_joined": "2025-01-01T00:00:00Z",
  "last_login": "2025-01-02T00:00:00Z"
}
```

**Error Responses:**
- `401`: Unauthorized
- `500`: Server error

---

### 6. Update User Profile

**Endpoint:** `PUT /api/auth/profile/update/` or `PATCH /api/auth/profile/update/`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer <access_token>
```

**Request Body (all fields optional):**
```json
{
  "email": "newemail@example.com",
  "first_name": "Johnny",
  "last_name": "Doe"
}
```

**Success Response (200):**
```json
{
  "message": "Profile updated successfully",
  "user": {
    "id": 1,
    "username": "john_doe",
    "email": "newemail@example.com",
    "first_name": "Johnny",
    "last_name": "Doe"
  }
}
```

**Error Responses:**
- `400`: Email already in use
- `401`: Unauthorized
- `500`: Server error

---

### 7. Change Password

**Endpoint:** `POST /api/auth/password/change/`

**Authentication:** Required

**Headers:**
```
Authorization: Bearer <access_token>
```

**Request Body:**
```json
{
  "old_password": "OldPass123!",
  "new_password": "NewPass456!",
  "new_password_confirm": "NewPass456!"
}
```

**Success Response (200):**
```json
{
  "message": "Password changed successfully"
}
```

**Error Responses:**
- `400`: Validation errors (old password incorrect, passwords don't match, weak password)
- `401`: Unauthorized
- `500`: Server error

---

## JWT Token Configuration

### Token Lifetimes
- **Access Token:** 1 hour
- **Refresh Token:** 7 days

### Token Rotation
- Refresh tokens are rotated on each refresh
- Old refresh tokens are blacklisted after rotation

---

## Using Authentication in Requests

### Example: cURL

#### 1. Register/Login to get tokens
```bash
curl -X POST http://localhost:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "password": "SecurePass123!"
  }'
```

#### 2. Use access token for authenticated requests
```bash
curl -X POST http://localhost:8000/api/query/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..." \
  -d '{
    "query": "What universities are in Sydney?",
    "session_id": "test_session"
  }'
```

#### 3. Refresh access token when expired
```bash
curl -X POST http://localhost:8000/api/auth/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }'
```

---

### Example: JavaScript/Fetch

```javascript
// 1. Login
const loginResponse = await fetch('http://localhost:8000/api/auth/login/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    username: 'john_doe',
    password: 'SecurePass123!'
  })
});

const { tokens } = await loginResponse.json();
localStorage.setItem('access_token', tokens.access);
localStorage.setItem('refresh_token', tokens.refresh);

// 2. Make authenticated request
const queryResponse = await fetch('http://localhost:8000/api/query/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${localStorage.getItem('access_token')}`
  },
  body: JSON.stringify({
    query: 'What universities are in Sydney?',
    session_id: 'test_session'
  })
});

// 3. Handle 401 (token expired) - refresh token
if (queryResponse.status === 401) {
  const refreshResponse = await fetch('http://localhost:8000/api/auth/token/refresh/', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      refresh: localStorage.getItem('refresh_token')
    })
  });

  const { access } = await refreshResponse.json();
  localStorage.setItem('access_token', access);

  // Retry original request
  // ...
}
```

---

### Example: Python Requests

```python
import requests

# 1. Login
login_response = requests.post(
    'http://localhost:8000/api/auth/login/',
    json={
        'username': 'john_doe',
        'password': 'SecurePass123!'
    }
)

tokens = login_response.json()['tokens']
access_token = tokens['access']
refresh_token = tokens['refresh']

# 2. Make authenticated request
query_response = requests.post(
    'http://localhost:8000/api/query/',
    headers={
        'Authorization': f'Bearer {access_token}'
    },
    json={
        'query': 'What universities are in Sydney?',
        'session_id': 'test_session'
    }
)

# 3. Refresh token if needed
if query_response.status_code == 401:
    refresh_response = requests.post(
        'http://localhost:8000/api/auth/token/refresh/',
        json={'refresh': refresh_token}
    )

    access_token = refresh_response.json()['access']
    # Retry original request
```

---

## Protected Endpoints

All endpoints except the following require authentication:

### Public Endpoints (No Authentication Required)
- `GET /api/` - Frontend
- `GET /api/developer/` - Developer dashboard page
- `GET /api/health/` - Health check
- `POST /api/auth/register/` - User registration
- `POST /api/auth/login/` - User login
- `POST /api/auth/token/refresh/` - Refresh access token

### Protected Endpoints (Authentication Required)
All other endpoints require a valid JWT access token in the `Authorization` header:

- `POST /api/query/` - Process queries
- `POST /api/upload/document/` - Upload documents
- `POST /api/upload/csv/` - Upload CSV files
- `GET /api/memory/{session_id}/` - Get session memory
- `GET /api/sessions/` - List sessions
- `GET /api/metrics/` - System metrics
- `GET /api/analytics/queries/` - Query analytics
- `GET /api/dashboard/` - Developer dashboard API
- And all other endpoints...

---

## Testing Authentication

### Using Postman

1. **Create Registration Request:**
   - Method: `POST`
   - URL: `http://localhost:8000/api/auth/register/`
   - Body (JSON):
     ```json
     {
       "username": "testuser",
       "email": "test@example.com",
       "password": "TestPass123!",
       "password_confirm": "TestPass123!"
     }
     ```

2. **Copy Access Token from Response**

3. **Test Protected Endpoint:**
   - Method: `POST`
   - URL: `http://localhost:8000/api/query/`
   - Headers:
     - `Authorization`: `Bearer <paste_access_token_here>`
   - Body (JSON):
     ```json
     {
       "query": "What is the cost of studying in Sydney?",
       "session_id": "test_123"
     }
     ```

---

## Security Considerations

### Token Storage
- **Frontend (Browser):** Store tokens in `localStorage` or `sessionStorage` (not cookies for JWT)
- **Mobile:** Use secure storage (Keychain on iOS, Keystore on Android)
- **Never** expose tokens in URLs or logs

### Token Refresh Strategy
- Implement automatic token refresh before expiration
- Store refresh token securely
- Handle token refresh failures (redirect to login)

### HTTPS
- **Always use HTTPS in production**
- JWT tokens are sensitive and should never be transmitted over HTTP

### Password Requirements
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- Django's built-in password validation is enforced

---

## Troubleshooting

### "Authentication credentials were not provided"
- Ensure you're including the `Authorization` header
- Format: `Authorization: Bearer <access_token>`

### "Token is invalid or expired"
- Access token has expired (1 hour lifetime)
- Use refresh token to get a new access token

### "Invalid refresh token"
- Refresh token has expired (7 days) or been blacklisted
- User needs to login again

### "User account is disabled"
- Contact administrator to activate account

---

## Migration from AllowAny

If you have existing frontend code using the API without authentication:

1. **Update all requests to include Authorization header**
2. **Implement login/register flow**
3. **Handle token expiration and refresh**
4. **Store tokens securely**

Example migration:

**Before:**
```javascript
fetch('http://localhost:8000/api/query/', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ query: 'test' })
});
```

**After:**
```javascript
const accessToken = localStorage.getItem('access_token');

fetch('http://localhost:8000/api/query/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${accessToken}`
  },
  body: JSON.stringify({ query: 'test' })
});
```

---

## Next Steps

1. **Update `.env` with your Azure OpenAI credentials**
2. **Run migrations:** `python manage.py migrate`
3. **Create a superuser:** `python manage.py createsuperuser`
4. **Start the server:** `python manage.py runserver`
5. **Test authentication endpoints using the examples above**
6. **Update your frontend to use JWT authentication**

---

**Documentation Date:** 2025-10-07
**API Version:** 1.0.0
**Authentication Method:** JWT (djangorestframework-simplejwt 5.5.1)

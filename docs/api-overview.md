# NovaMart API Overview

## Overview

NovaMart exposes a comprehensive REST API built with Spring Boot that provides full access to all e-commerce functionality. The API follows RESTful principles and is documented with SpringDoc OpenAPI (Swagger UI). This document provides a detailed overview of all API endpoints, authentication mechanisms, request/response formats, and usage guidelines.

## Base URL

- **Production**: [Configured via environment variable]
- **Swagger UI**: /swagger-ui/index.html (relative path)
- **OpenAPI Spec**: /v3/api-docs (relative path)
- **Security Note**: Production URLs should be configured via environment variables and not exposed in documentation

## API Architecture

### RESTful Design Principles

- **Resource-based URLs**: Clear, hierarchical resource paths
- **HTTP Methods**: Proper use of GET, POST, PUT, DELETE
- **Status Codes**: Appropriate HTTP status codes for responses
- **Stateless**: JWT-based authentication without server-side sessions
- **JSON Format**: Request/response bodies in JSON format

### API Versioning

Current version: v1 (implicit in URL structure)

Future versions will use URL path versioning (e.g., /api/v2/...)

## Authentication

### JWT Authentication

NovaMart uses JWT (JSON Web Tokens) for authentication with a dual-token system:

#### Access Token
- **Expiry**: Short-lived (configured via JWT_EXPIRATION environment variable)
- **Algorithm**: HMAC256
- **Usage**: Used for authenticated API requests
- **Header**: `Authorization: Bearer <access_token>`

#### Refresh Token
- **Expiry**: Extended (configured via REFRESH_TOKEN_EXPIRATION environment variable)
- **Storage**: Database
- **Usage**: Obtain new access tokens
- **Rotation**: Old token invalidated on refresh
- **Security Note**: Exact expiry times should be configured in environment variables

### Token Type Claims

Each JWT includes a `type` claim to prevent token misuse:
- `access`: Used for API requests
- `refresh`: Used only for token refresh

### Authentication Flow

1. User registers or logs in
2. Server returns access token + refresh token
3. Client stores refresh token securely
4. Client uses access token in Authorization header
5. When access token expires, use refresh token to get new pair
6. Old refresh token is invalidated on rotation

### Google OAuth2

Alternative authentication method:
- Redirect to Google OAuth2 endpoint
- User authenticates with Google
- Google redirects back with authorization code
- Server exchanges code for tokens
- Server creates user (if new) and cart
- Server issues JWT tokens
- Redirect to frontend with tokens

## API Endpoints

### Authentication Endpoints

#### POST /api/register
Register a new user account.

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123",
  "firstName": "John",
  "lastName": "Doe",
  "phoneNumber": "1234567890"
}
```

**Response**: 201 Created
```json
{
  "message": "Registration successful. Please verify your email."
}
```

**Features**:
- Sends OTP via email (stored in Redis with TTL)
- Sends clickable verification link
- Auto-creates cart
- Auto-assigns USER role

#### GET /api/verify-email
Verify email via clickable link token.

**Query Parameters**:
- `token`: Verification token

**Response**: 200 OK
```json
{
  "message": "Email verified successfully"
}
```

#### POST /api/verify-otp
Verify email via OTP.

**Request Body**:
```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

**Response**: 200 OK
```json
{
  "message": "Email verified successfully"
}
```

**Features**:
- Max attempts configured via environment
- Locks after max attempts
- OTP expiry configured via environment (stored in Redis with TTL)
- **Security Note**: Exact limits and timings should be configured in environment variables

#### POST /api/login
Authenticate user and receive JWT tokens.

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "SecurePass123"
}
```

**Response**: 200 OK
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

**Features**:
- Returns access + refresh tokens
- Access token expiry configured via environment
- Refresh token expiry configured via environment

#### POST /api/refresh
Refresh access token using refresh token.

**Request Body**:
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response**: 200 OK
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

**Features**:
- Token rotation
- Old refresh token invalidated
- New pair issued

#### POST /api/logout
Invalidate refresh token.

**Request Body**:
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response**: 200 OK
```json
{
  "message": "Logged out successfully"
}
```

#### POST /api/resend-otp
Resend verification OTP.

**Request Body**:
```json
{
  "email": "user@example.com"
}
```

**Response**: 200 OK
```json
{
  "message": "OTP sent successfully"
}
```

**Features**:
- Cooldown period configured via environment (enforced via Redis)
- Rate limited (limits configured via environment)
- **Security Note**: Exact timing and limits should be configured in environment variables

#### POST /api/forgot-password
Initiate password reset.

**Request Body**:
```json
{
  "email": "user@example.com"
}
```

**Response**: 200 OK
```json
{
  "message": "Password reset OTP sent"
}
```

#### POST /api/reset-password
Reset password with OTP.

**Request Body**:
```json
{
  "email": "user@example.com",
  "otp": "123456",
  "newPassword": "NewSecurePass123"
}
```

**Response**: 200 OK
```json
{
  "message": "Password reset successfully"
}
```

**Features**:
- Max attempts configured via environment
- Invalidates all refresh tokens
- OTP expiry configured via environment (stored in Redis with TTL)
- **Security Note**: Exact limits and timings should be configured in environment variables

#### GET /api/verification-status
Check email verification status.

**Query Parameters**:
- `email`: User email

**Response**: 200 OK
```json
{
  "verified": true
}
```

### User Endpoints

#### GET /api/admin/users
Get paginated list of all users (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK
```json
{
  "content": [
    {
      "userId": 1,
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": {
        "roleId": 102,
        "roleName": "USER"
      }
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "totalElements": 100,
    "totalPages": 10
  }
}
```

#### GET /api/public/users/me
Get current user profile.

**Headers**:
- `Authorization: Bearer <access_token>`

**Response**: 200 OK
```json
{
  "userId": 1,
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "phoneNumber": "1234567890",
  "role": {
    "roleId": 102,
    "roleName": "USER"
  }
}
```

#### PUT /api/public/users/me
Update current user profile.

**Headers**:
- `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "firstName": "Jane",
  "lastName": "Smith",
  "phoneNumber": "9876543210"
}
```

**Response**: 200 OK
```json
{
  "userId": 1,
  "email": "user@example.com",
  "firstName": "Jane",
  "lastName": "Smith",
  "phoneNumber": "9876543210"
}
```

#### DELETE /api/admin/users/{userId}
Delete user (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `userId`: User ID to delete

**Response**: 200 OK
```json
{
  "message": "User deleted successfully"
}
```

**Features**:
- Cleans up cart items
- Cascades to related entities

### Product Endpoints

#### POST /api/admin/categories/{categoryId}/product
Add product to category (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `categoryId`: Category ID

**Request Body**:
```json
{
  "productName": "Laptop",
  "description": "High-performance laptop",
  "price": 999.99,
  "discount": 10,
  "quantity": 50,
  "image": "laptop.jpg"
}
```

**Response**: 201 Created
```json
{
  "productId": 1,
  "productName": "Laptop",
  "specialPrice": 899.99,
  "quantity": 50
}
```

**Features**:
- Auto-calculates special price
- Validates category exists

#### GET /api/public/products
Get paginated product list (cached).

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK
```json
{
  "content": [
    {
      "productId": 1,
      "productName": "Laptop",
      "specialPrice": 899.99,
      "quantity": 50,
      "category": {
        "categoryId": 1,
        "categoryName": "Electronics"
      }
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "totalElements": 100,
    "totalPages": 10
  }
}
```

#### GET /api/admin/products
Get paginated product list (Admin only, cached).

**Headers**:
- `Authorization: Bearer <access_token>`

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK (same as public endpoint)

#### GET /api/public/categories/{categoryId}/products
Get products by category (cached).

**Path Parameters**:
- `categoryId`: Category ID

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK (same format as product list)

#### GET /api/public/products/keyword/{keyword}
Search products by keyword (cached).

**Path Parameters**:
- `keyword`: Search keyword (case-insensitive)

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK (same format as product list)

#### PUT /api/admin/products/{productId}
Update product (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `productId`: Product ID

**Request Body**:
```json
{
  "productName": "Updated Laptop",
  "price": 1099.99,
  "discount": 15,
  "quantity": 45
}
```

**Response**: 200 OK
```json
{
  "productId": 1,
  "productName": "Updated Laptop",
  "specialPrice": 934.99,
  "quantity": 45
}
```

**Features**:
- Recalculates special price
- Syncs all carts
- Evicts cache

#### PUT /api/admin/products/{productId}/image
Upload product image (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`
- `Content-Type: multipart/form-data`

**Path Parameters**:
- `productId`: Product ID

**Request Body**:
- `file`: Image file (multipart)

**Response**: 200 OK
```json
{
  "message": "Image uploaded successfully",
  "imageUrl": "https://res.cloudinary.com/your-cloud-name/image/upload/v123/product123.jpg",
  "publicId": "product123"
}
```

#### DELETE /api/admin/products/{productId}
Delete product (Admin only, soft delete).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `productId`: Product ID

**Response**: 200 OK
```json
{
  "message": "Product deleted successfully"
}
```

**Features**:
- Soft delete
- Nullifies order item references
- Clears from all carts
- Evicts cache

### Category Endpoints

#### POST /api/admin/category
Create category (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "categoryName": "Electronics"
}
```

**Response**: 201 Created
```json
{
  "categoryId": 1,
  "categoryName": "Electronics"
}
```

#### GET /api/public/categories
Get paginated category list.

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK
```json
{
  "content": [
    {
      "categoryId": 1,
      "categoryName": "Electronics"
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "totalElements": 20,
    "totalPages": 2
  }
}
```

#### PUT /api/admin/categories/{categoryId}
Update category (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `categoryId`: Category ID

**Request Body**:
```json
{
  "categoryName": "Updated Electronics"
}
```

**Response**: 200 OK
```json
{
  "categoryId": 1,
  "categoryName": "Updated Electronics"
}
```

#### DELETE /api/admin/categories/{categoryId}
Delete category (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `categoryId`: Category ID

**Response**: 200 OK
```json
{
  "message": "Category deleted successfully"
}
```

### Cart Endpoints

#### POST /api/public/carts/{cartId}/products/{productId}/quantity/{quantity}
Add product to cart.

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `cartId`: Cart ID
- `productId`: Product ID
- `quantity`: Quantity to add

**Response**: 201 Created
```json
{
  "cartItemId": 1,
  "quantity": 2,
  "price": 899.99
}
```

**Features**:
- Checks available stock
- Increments reservedQuantity
- Prevents duplicates
- Recalculates cart total

#### GET /api/admin/carts
Get all carts (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Response**: 200 OK
```json
{
  "content": [
    {
      "cartId": 1,
      "totalPrice": 1799.98,
      "cartItems": [...]
    }
  ]
}
```

#### GET /api/public/users/{emailId}/carts/{cartId}
Get user's cart.

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `emailId`: User email
- `cartId`: Cart ID

**Response**: 200 OK
```json
{
  "cartId": 1,
  "totalPrice": 1799.98,
  "cartItems": [
    {
      "cartItemId": 1,
      "quantity": 2,
      "price": 899.99,
      "product": {
        "productId": 1,
        "productName": "Laptop"
      }
    }
  ]
}
```

#### PUT /api/public/carts/{cartId}/products/{productId}/quantity/{quantity}
Update cart item quantity.

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `cartId`: Cart ID
- `productId`: Product ID
- `quantity`: New quantity

**Response**: 200 OK
```json
{
  "cartItemId": 1,
  "quantity": 3,
  "price": 899.99
}
```

**Features**:
- Adjusts reservedQuantity diff
- Recalculates cart total

#### DELETE /api/public/carts/{cartId}/product/{productId}
Remove item from cart.

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `cartId`: Cart ID
- `productId`: Product ID

**Response**: 200 OK
```json
{
  "message": "Item removed from cart"
}
```

**Features**:
- Releases reservedQuantity
- Recalculates cart total

### Order Endpoints

#### POST /api/public/users/{emailId}/carts/{cartId}/payments/{paymentMethod}/order
Place order with payment processing.

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `emailId`: User email
- `cartId`: Cart ID
- `paymentMethod`: Payment method (e.g., "CASHFREE")

**Response**: 201 Created
```json
{
  "orderId": 1,
  "orderStatus": "PENDING_PAYMENT",
  "orderDate": "2024-01-15T10:30:00",
  "totalAmount": 1799.98,
  "paymentId": 1
}
```

**Features**:
- Pre-checks stock
- Creates payment entity
- Saves shipping address snapshot
- Initiates payment process based on payment method
- Sends confirmation email (async) after successful payment

### Payment Endpoints

#### POST /api/payments/cashfree/create
Initiate Cashfree payment for an order.

**Headers**:
- `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "orderId": 1,
  "amount": 1799.98,
  "customerEmail": "user@example.com",
  "customerPhone": "1234567890"
}
```

**Response**: 200 OK
```json
{
  "paymentSessionId": "session_abc123",
  "paymentLink": "https://payments.cashfree.com/order/session_abc123",
  "expiryTime": "2024-01-15T11:00:00"
}
```

**Features**:
- Creates payment order with Cashfree API
- Generates payment link for user checkout
- Sets 30-minute expiry for payment completion
- Stores payment session for callback verification

#### POST /api/payments/cashfree/callback
Cashfree payment callback webhook.

**Headers**:
- `X-Cashfree-Signature`: HMAC signature for verification

**Request Body**:
```json
{
  "orderId": "order_abc123",
  "orderAmount": 1799.98,
  "referenceId": "1",
  "txStatus": "SUCCESS",
  "paymentMode": "UPI",
  "txTime": "2024-01-15T10:35:00"
}
```

**Response**: 200 OK
```json
{
  "message": "Payment processed successfully"
}
```

**Features**:
- Verifies HMAC signature for security
- Updates payment status based on callback
- Creates order on successful payment
- Decrements product quantity on success
- Clears cart on successful payment
- Sends order confirmation email
- Handles payment failures gracefully

#### GET /api/payments/{orderId}/status
Check payment status for an order.

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `orderId`: Order ID

**Response**: 200 OK
```json
{
  "orderId": 1,
  "paymentStatus": "SUCCESS",
  "paymentMethod": "CASHFREE",
  "amount": 1799.98,
  "paymentDate": "2024-01-15T10:35:00",
  "retryExpiry": "2024-01-15T11:00:00"
}
```

**Features**:
- Returns current payment status
- Shows retry window expiry if payment failed
- Allows user to check payment status before retry

#### GET /api/admin/orders
Get all orders (Admin only, cached).

**Headers**:
- `Authorization: Bearer <access_token>`

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK
```json
{
  "content": [
    {
      "orderId": 1,
      "orderStatus": "Order Accepted !",
      "orderDate": "2024-01-15T10:30:00",
      "totalAmount": 1799.98,
      "user": {
        "userId": 1,
        "email": "user@example.com"
      },
      "payment": {
        "paymentStatus": "SUCCESS",
        "paymentMethod": "CASHFREE"
      }
    }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "totalElements": 50,
    "totalPages": 5
  }
}
```

#### GET /api/public/users/{emailId}/orders
Get user's orders (cached).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `emailId`: User email

**Query Parameters**:
- `pageNumber`: Page number (default: 0)
- `pageSize`: Page size (default: 10)
- `sortBy`: Sort field
- `sortOrder`: Sort direction (ASC/DESC)

**Response**: 200 OK (same format as admin orders)

#### GET /api/public/users/{emailId}/orders/{orderId}
Get single order (cached).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `emailId`: User email
- `orderId`: Order ID

**Response**: 200 OK
```json
{
  "orderId": 1,
  "orderStatus": "Order Accepted !",
  "orderDate": "2024-01-15T10:30:00",
  "totalAmount": 1799.98,
  "shippingAddress": "123 Main St, City, State 12345",
  "orderItems": [
    {
      "orderItemId": 1,
      "quantity": 2,
      "price": 899.99,
      "productName": "Laptop"
    }
  ],
  "payment": {
    "paymentId": 1,
    "paymentMethod": "CREDIT_CARD",
    "paymentStatus": "COMPLETED",
    "amount": 1799.98
  }
}
```

#### PUT /api/admin/users/{emailId}/orders/{orderId}/orderStatus/{orderStatus}
Update order status (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `emailId`: User email
- `orderId`: Order ID
- `orderStatus`: New status ("Order Accepted !", SHIPPED, DELIVERED)

**Response**: 200 OK
```json
{
  "orderId": 1,
  "orderStatus": "SHIPPED"
}
```

**Features**:
- Evicts cache
- Updates order status

### Address Endpoints

#### POST /api/admin/address
Create address (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Request Body**:
```json
{
  "streetAddress": "123 Main St",
  "city": "Springfield",
  "state": "IL",
  "postalCode": "62701",
  "country": "USA"
}
```

**Response**: 201 Created
```json
{
  "addressId": 1,
  "streetAddress": "123 Main St",
  "city": "Springfield",
  "state": "IL",
  "postalCode": "62701",
  "country": "USA"
}
```

#### GET /api/admin/addresses
Get all addresses (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Response**: 200 OK
```json
{
  "content": [
    {
      "addressId": 1,
      "streetAddress": "123 Main St",
      "city": "Springfield",
      "state": "IL",
      "postalCode": "62701",
      "country": "USA"
    }
  ]
}
```

#### GET /api/admin/addresses/{addressId}
Get single address (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `addressId`: Address ID

**Response**: 200 OK (same format as address list item)

#### PUT /api/admin/addresses/{addressId}
Update address (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `addressId`: Address ID

**Request Body**:
```json
{
  "streetAddress": "456 Oak Ave",
  "city": "Shelbyville",
  "state": "IL",
  "postalCode": "62702",
  "country": "USA"
}
```

**Response**: 200 OK
```json
{
  "addressId": 1,
  "streetAddress": "456 Oak Ave",
  "city": "Shelbyville",
  "state": "IL",
  "postalCode": "62702",
  "country": "USA"
}
```

**Features**:
- Deduplication: reassigns users if same address exists

#### DELETE /api/admin/addresses/{addressId}
Delete address (Admin only).

**Headers**:
- `Authorization: Bearer <access_token>`

**Path Parameters**:
- `addressId`: Address ID

**Response**: 200 OK
```json
{
  "message": "Address deleted successfully"
}
```

**Features**:
- Removes from all associated users

### File Service Endpoints

#### GET /api/images/{fileName}
Serve product image.

**Path Parameters**:
- `fileName`: Image filename

**Response**: 200 OK (image file)

**Content-Type**: image/jpeg, image/png, etc.

## HTTP Status Codes

### Success Codes
- **200 OK**: Request successful
- **201 Created**: Resource created successfully
- **204 No Content**: Request successful, no content returned

### Client Error Codes
- **400 Bad Request**: Invalid request data
- **401 Unauthorized**: Authentication required or invalid
- **403 Forbidden**: Insufficient permissions
- **404 Not Found**: Resource not found
- **409 Conflict**: Resource conflict (e.g., duplicate email)
- **429 Too Many Requests**: Rate limit exceeded

### Server Error Codes
- **500 Internal Server Error**: Server error
- **503 Service Unavailable**: Service temporarily unavailable

## Error Response Format

All error responses follow this format:

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation failed for field 'email'",
  "path": "/api/register"
}
```

## Rate Limiting

API endpoints are rate limited to prevent abuse:

- **Login**: Configured attempts per time window per IP
- **Register**: Configured attempts per time window per IP
- **Forgot Password**: Configured attempts per time window per IP
- **Security Note**: Exact rate limits should be configured in environment variables and not exposed in documentation

Rate limit headers are included in responses:
- `X-RateLimit-Limit`: Maximum requests
- `X-RateLimit-Remaining`: Remaining requests
- `X-RateLimit-Reset`: Reset time (Unix timestamp)

## Pagination

List endpoints support pagination with these parameters:

- `pageNumber`: Page number (0-indexed, default: 0)
- `pageSize`: Items per page (default: 10, max: 100)
- `sortBy`: Field to sort by
- `sortOrder`: Sort direction (ASC or DESC, default: ASC)

Pagination response includes:
```json
{
  "content": [...],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10,
    "totalElements": 100,
    "totalPages": 10,
    "first": true,
    "last": false
  }
}
```

## Caching

Read operations are cached for performance:

- Product listings
- Category-based product queries
- Keyword searches
- Order data
- User lists

Cache is automatically evicted on write operations.

## CORS Configuration

CORS is configured to allow requests from:
- Frontend URL: Configured via environment variable
- Allowed methods: GET, POST, PUT, DELETE, OPTIONS
- Allowed headers: Authorization, Content-Type
- **Security Note**: Frontend URL should be configured via environment variables

## Security Best Practices

### For API Consumers

1. **Never expose refresh tokens** in client-side code
2. **Use HTTPS** for all API calls
3. **Validate SSL certificates**
4. **Implement proper error handling**
5. **Respect rate limits**
6. **Store tokens securely** (httpOnly cookies for web, secure storage for mobile)
7. **Implement token refresh logic** before expiry
8. **Sanitize user input** before sending to API

### Token Management

1. **Access tokens**: Short-lived (configured via environment), store in memory
2. **Refresh tokens**: Long-lived (configured via environment), store securely
3. **Rotate refresh tokens** on every use
4. **Clear tokens** on logout
5. **Handle token expiry** gracefully
- **Security Note**: Exact expiry times should be configured in environment variables

## API Testing

### Swagger UI

Interactive API testing available at:
[Production URL]/swagger-ui/index.html (configured via environment)
- **Security Note**: Production URL should be configured via environment variable

Features:
- Try out endpoints directly
- View request/response schemas
- Authentication support
- Example requests

### cURL Examples

**Register User**:
```bash
curl -X POST [PRODUCTION_URL]/api/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"SecurePass123","firstName":"John","lastName":"Doe"}'
```

**Login**:
```bash
curl -X POST [PRODUCTION_URL]/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"SecurePass123"}'
```

**Get Products**:
```bash
curl -X GET [PRODUCTION_URL]/api/public/products \
  -H "Authorization: Bearer <access_token>"
```
- **Security Note**: Replace [PRODUCTION_URL] with your configured backend URL

## Payment Integration

### Cashfree Payment Gateway

NovaMart integrates with Cashfree payment gateway for secure payment processing.

#### Payment Flow

1. **Initiation**: User selects Cashfree as payment method at checkout
2. **Order Creation**: System creates payment order via Cashfree API
3. **Redirection**: User redirected to Cashfree checkout page
4. **Payment**: User completes payment on Cashfree platform
5. **Callback**: Cashfree sends webhook callback with payment status
6. **Processing**: System verifies callback and updates order status
7. **Completion**: Order confirmed on successful payment, cart cleared
8. **Retry**: If payment fails, user has 30-minute window to retry

#### Payment Statuses

- `PENDING`: Payment initiated, awaiting user action
- `SUCCESS`: Payment completed successfully
- `FAILED`: Payment failed, retry available within 30 minutes
- `EXPIRED`: Payment window expired (30 minutes)

#### Security Features

- HMAC signature verification for all webhooks
- Secure API key management via environment variables
- PCI DSS compliant payment processing
- Encrypted data transmission
- Token-based payment sessions

#### Environment Configuration

```bash
CASHFREE_APP_ID=your_app_id
CASHFREE_SECRET_KEY=your_secret_key
CASHFREE_API_URL=https://sandbox.cashfree.com/api/v2  # or production URL
```

#### Webhook Configuration

Configure webhook URL in Cashfree dashboard:
- **Sandbox**: `https://your-backend.com/api/payments/cashfree/callback`
- **Production**: `https://your-backend.com/api/payments/cashfree/callback`

#### Error Handling

- Payment failures are logged with detailed error messages
- Users can retry failed payments within 30-minute window
- Automatic cleanup of expired payment sessions
- Alert notifications for payment anomalies

### Current Version
- Version: v1
- Status: Stable
- Deprecation: Not planned

### Versioning Strategy
Future versions will use URL path versioning:
- `/api/v1/` - Current version
- `/api/v2/` - Future version

### Deprecation Policy
- Minimum 6 months notice before deprecation
- Clear migration guide provided
- Sunset date announced in advance

## Support and Documentation

### Additional Resources
- Swagger UI: [PRODUCTION_URL]/swagger-ui/index.html (configured via environment)
- OpenAPI Spec: [PRODUCTION_URL]/v3/api-docs (configured via environment)
- Project README: [README.md](../README.md)
- Tech Stack: [tech-stack.md](tech-stack.md)
- **Security Note**: Production URL should be configured via environment variable

### Contact
For API support and questions, refer to the project repository.

## Conclusion

NovaMart's API provides a comprehensive, secure, and well-documented interface for all e-commerce operations. Following RESTful principles and implementing robust authentication, the API serves as the backbone for both the React frontend and potential third-party integrations. The use of JWT tokens, caching, rate limiting, and proper error handling ensures a reliable and performant API experience.

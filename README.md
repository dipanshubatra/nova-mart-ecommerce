<p align="center">
  <img src="assets/banner.png" alt="NovaMart Banner" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Java-17-orange?style=flat-square&logo=java" alt="Java 17">
  <img src="https://img.shields.io/badge/Spring%20Boot-3.0.1-green?style=flat-square&logo=spring-boot" alt="Spring Boot 3.0.1">
  <img src="https://img.shields.io/badge/PostgreSQL-Database-blue?style=flat-square&logo=postgresql" alt="PostgreSQL">
  <img src="https://img.shields.io/badge/JWT-Authentication-red?style=flat-square&logo=jwt" alt="JWT">
  <img src="https://img.shields.io/badge/Docker-Container-blue?style=flat-square&logo=docker" alt="Docker">
  <img src="https://img.shields.io/badge/Deployed-Render-success?style=flat-square" alt="Deployed on Render">
  <img src="https://img.shields.io/badge/License-MIT-yellow?style=flat-square" alt="MIT License">
</p>

<h1 align="center">NovaMart</h1>

<p align="center">A full-stack e-commerce platform with secure authentication, real-time inventory management, and automated order processing.</p>

## Live Demo

🌐 **Live Application**  
https://martnova.vercel.app

## Overview

NovaMart is a production-ready e-commerce platform built with Spring Boot 3 and React.

It features comprehensive user authentication with JWT and Google OAuth2, real-time cart management with stock reservation, automated email notifications via Brevo API, and intelligent caching for optimal performance.

The system is fully containerized with Docker and deployed on Render with PostgreSQL database.

## Screenshots

| Home Page | Product Page | Cart Page |
|-----------|--------------|-----------|
| ![Home Page](screenshots/home-page.png) | ![Product Page](screenshots/product-page.png) | ![Cart Page](screenshots/cart-page.png) |

| Checkout Page | Admin Dashboard | Swagger API |
|---------------|-----------------|-------------|
| ![Checkout Page](screenshots/checkout-page.png) | ![Admin Dashboard](screenshots/admin-dashboard.png) | ![Swagger API](screenshots/swagger-api.png) |

## Tech Stack

| Category | Technology | Purpose |
|----------|------------|---------|
| Backend Language | Java 17 | Core application logic |
| Backend Framework | Spring Boot 3.0.1 | Application framework |
| Security | Spring Security + JWT (auth0 java-jwt 4.2.1) | Authentication & authorization |
| OAuth2 | Spring OAuth2 Client | Google OAuth2 login |
| Database | Spring Data JPA + PostgreSQL | Data persistence |
| Caching | Spring Boot Cache (@Cacheable, @CacheEvict) | Performance optimization |
| Email | Spring Boot Mail + Brevo SMTP API | Transactional emails via REST |
| Validation | Spring Boot Validation | Input validation |
| API Documentation | SpringDoc OpenAPI / Swagger UI 2.0.2 | API documentation |
| Rate Limiting | Bucket4j 8.1.1 | IP-based rate limiting |
| Object Mapping | ModelMapper 3.1.1 | DTO mapping |
| Code Generation | Lombok 1.18.30 | Reduce boilerplate |
| Scheduling | Spring Boot Scheduling (@Scheduled) | Scheduled tasks |
| Async Processing | Spring Boot Async (@Async) | Asynchronous operations |
| Containerization | Docker | Container deployment |
| Frontend | React | User interface |
| Frontend Deployment | Vercel | Frontend hosting |
| Backend Deployment | Render | Backend hosting |
| Database Hosting | PostgreSQL on Render | Managed database |

## Architecture

![System Architecture](diagrams/system-architecture.png)

The system follows a microservices-inspired architecture with clear separation of concerns.

The Spring Boot backend handles all business logic, authentication, and data persistence, while the React frontend provides a responsive user interface.

PostgreSQL serves as the primary database, with Redis caching for performance optimization.

The system is deployed using Docker containers on Render for the backend and Vercel for the frontend.

## Features

### 🔐 Authentication Module

- User registration with email verification (OTP + verification link)

- Email verification via clickable link token

- Email verification via 6-digit OTP (max 5 attempts, then locked)

- JWT-based login with access and refresh tokens

- Refresh token rotation (old token invalidated, new pair issued)

- Secure logout with refresh token invalidation

- OTP resend with 3-minute cooldown enforcement

- Forgot password with OTP reset

- Password reset with max 5 attempts (invalidates all refresh tokens)

- Email verification status check

- Google OAuth2 login with auto user/cart creation and JWT token issuance

### 🛡️ JWT Security

- Access Token: 15 minutes expiry (HMAC256)

- Refresh Token: 7 days expiry, stored in database

- Token rotation on every refresh

- JWT Filter validates token type (access vs refresh)

- Token type claim ("access"/"refresh") prevents token misuse

### 🚦 Rate Limiting

- Login: 5 attempts per minute per IP

- Register: 5 attempts per 10 minutes per IP

- Forgot Password: 3 attempts per 10 minutes per IP

- In-memory ConcurrentHashMap per IP using Bucket4j

### 👤 User Module

- Paginated user list (Admin only)

- Get own profile from JWT

- Update own profile

- Delete user with cart cleanup (Admin only)

- Auto-creates cart on registration

- Role-based access: ADMIN (roleId=101), USER (roleId=102)

- Roles seeded via CommandLineRunner on startup

### 📦 Product Module

- Add product to category (Admin)

- Paginated product list (cached)

- Admin product list

- Products by category (cached)

- Case-insensitive keyword search (cached)

- Update product with special price recalculation and cart synchronization

- Upload product image (multipart)

- Soft-safe delete with order item nullification and cart clearing

- Special price auto-calculation: price - (discount% * price), rounded to 2 decimal places

- Caching: @Cacheable on getAllProducts, searchByCategory, searchByKeyword; @CacheEvict on write operations

### 📂 Category Module

- Create category (Admin)

- Paginated category list (public)

- Update category (Admin)

- Delete category (Admin)

### 🛒 Cart Module

- Add product to cart with stock reservation check

- Stock availability check: quantity - reservedQuantity

- Reserved quantity increment on product

- Duplicate product prevention in cart

- All carts list (Admin)

- Get user's cart

- Update quantity with reserved quantity adjustment

- Remove from cart with reserved quantity release

- Auto-recalculate cart total on every operation

- Abandoned cart cleanup scheduler

### 📋 Order Module

- Place order with stock pre-check

- Payment entity creation

- Shipping address snapshot on order

- Product quantity decrement

- Cart clearing after order

- Order confirmation email (async via Brevo)

- Paginated all orders (Admin, cached)

- User's orders (cached by email)

- Single order details (cached)

- Update order status (Admin, cache evicted)

- Order statuses: "Order Accepted !" → SHIPPED → DELIVERED

### 📍 Address Module

- Create address

- All addresses list

- Single address details

- Update address with deduplication (reassigns users if same address exists)

- Delete address (removes from all associated users)

### 📧 Email System (Brevo REST API)

All emails sent asynchronously via @Async:

- Verification email: 6-digit OTP (10min expiry) + clickable verification link, HTML styled

- Password reset OTP email: 6-digit OTP, 10min expiry

- Order confirmation email: Full order summary table with items, quantities, prices, total, animated HTML

- Daily report email: Revenue, total orders, new users, cancelled orders for previous day

### ⏰ Scheduler

- Daily report at 9:00 AM (cron: "0 0 9 * * *")

- Fetches yesterday's revenue, total orders, new user count, cancelled orders

- Sends daily report email to admin

### 📁 File Service

- Image upload for products (multipart, stored in images/ folder)

- Serve product images via GET /api/images/{fileName}

### 💾 Caching Strategy

- Spring Cache (in-memory / simple cache)

- Cached endpoints: products, productsByCategory, productsByKeyword, orders, allOrders, users

- @CacheEvict on all mutating operations

- Cache keys use pageNumber, pageSize, sortBy, sortOrder combinations

### 🔒 Security Configuration

- Public routes: /api/public/**, /api/verify-email, /api/register, /api/login, /api/refresh, /api/resend-otp, /api/verify-otp, /api/forgot-password, /api/reset-password, /swagger-ui/**, /v3/api-docs/**

- Admin routes: /api/admin/**

- CORS configured

- BCrypt password encoding

- OAuth2 login configured

### 🗄️ Database

- PostgreSQL (Render hosted)

- Hibernate DDL auto=update

- HikariCP pool: max 3 connections, min idle 1 (Render free tier optimized)

- Entities: User, Role, Cart, CartItem, Product, Category, Order, OrderItem, Payment, Address, RefreshToken, EmailVerificationToken, PasswordResetToken

### 🚀 Deployment

- Backend: Render (Docker container)

- Frontend: Vercel (React)

- Database: PostgreSQL on Render

- Environment variables: SPRING_DATASOURCE_JDBC_URL, SPRING_DATASOURCE_USERNAME, SPRING_DATASOURCE_PASSWORD, GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, PORT

- Dockerfile present

## API Endpoints

### Authentication
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | /api/register | Public | Register user, sends OTP + verification link email |
| GET | /api/verify-email | Public | Verify via link token |
| POST | /api/verify-otp | Public | Verify via OTP (max 5 attempts) |
| POST | /api/login | Public | JWT login (returns accessToken + refreshToken) |
| POST | /api/refresh | Public | Refresh token rotation |
| POST | /api/logout | Public | Invalidate refresh token |
| POST | /api/resend-otp | Public | Resend OTP (3-minute cooldown) |
| POST | /api/forgot-password | Public | Send password reset OTP |
| POST | /api/reset-password | Public | Reset password (max 5 attempts) |
| GET | /api/verification-status | Public | Check if email is verified |

### User
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | /api/admin/users | Admin | Paginated user list |
| GET | /api/public/users/me | User | Get own profile from JWT |
| PUT | /api/public/users/me | User | Update own profile |
| DELETE | /api/admin/users/{userId} | Admin | Delete user with cart cleanup |

### Product
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | /api/admin/categories/{categoryId}/product | Admin | Add product |
| GET | /api/public/products | Public | Paginated product list (cached) |
| GET | /api/admin/products | Admin | Admin product list |
| GET | /api/public/categories/{categoryId}/products | Public | Products by category (cached) |
| GET | /api/public/products/keyword/{keyword} | Public | Case-insensitive search (cached) |
| PUT | /api/admin/products/{productId} | Admin | Update product |
| PUT | /api/admin/products/{productId}/image | Admin | Upload product image |
| DELETE | /api/admin/products/{productId} | Admin | Soft-safe delete |

### Category
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | /api/admin/category | Admin | Create category |
| GET | /api/public/categories | Public | Paginated list |
| PUT | /api/admin/categories/{categoryId} | Admin | Update category |
| DELETE | /api/admin/categories/{categoryId} | Admin | Delete category |

### Cart
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | /api/public/carts/{cartId}/products/{productId}/quantity/{quantity} | User | Add to cart |
| GET | /api/admin/carts | Admin | All carts |
| GET | /api/public/users/{emailId}/carts/{cartId} | User | Get user's cart |
| PUT | /api/public/carts/{cartId}/products/{productId}/quantity/{quantity} | User | Update quantity |
| DELETE | /api/public/carts/{cartId}/product/{productId} | User | Remove from cart |

### Order
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | /api/public/users/{emailId}/carts/{cartId}/payments/{paymentMethod}/order | User | Place order |
| GET | /api/admin/orders | Admin | Paginated all orders (cached) |
| GET | /api/public/users/{emailId}/orders | User | User's orders (cached) |
| GET | /api/public/users/{emailId}/orders/{orderId} | User | Single order (cached) |
| PUT | /api/admin/users/{emailId}/orders/{orderId}/orderStatus/{orderStatus} | Admin | Update status |

### Address
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | /api/admin/address | Admin | Create address |
| GET | /api/admin/addresses | Admin | All addresses |
| GET | /api/admin/addresses/{addressId} | Admin | Single address |
| PUT | /api/admin/addresses/{addressId} | Admin | Update address |
| DELETE | /api/admin/addresses/{addressId} | Admin | Delete address |

### File Service
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | /api/images/{fileName} | Public | Serve product images |

## Authentication Flow

![Authentication Flow](diagrams/authentication-flow.png)

NovaMart uses a dual authentication system: JWT-based authentication for traditional login and Google OAuth2 for social login.

Users register with email and receive both a 6-digit OTP and a clickable verification link.

Upon verification, users can login to receive an access token (15 min expiry) and refresh token (7 days expiry).

Refresh tokens are rotated on every use to enhance security.

Google OAuth2 users are auto-created with a cart on first login and receive JWT tokens immediately after OAuth2 success.

## Caching Strategy

The system implements Spring Cache with in-memory storage for optimal performance.

**Cached Operations:**

- `getAllProducts` - Cache key: combination of pageNumber, pageSize, sortBy, sortOrder

- `searchByCategory` - Cache key: categoryId + pageNumber, pageSize, sortBy, sortOrder

- `searchByKeyword` - Cache key: keyword + pageNumber, pageSize, sortBy, sortOrder

- `getAllOrders` - Cache key: pageNumber, pageSize, sortBy, sortOrder

- `getUserOrders` - Cache key: email + pageNumber, pageSize, sortBy, sortOrder

- `getOrderById` - Cache key: orderId

- `getAllUsers` - Cache key: pageNumber, pageSize, sortBy, sortOrder

**Cache Eviction:**

All mutating operations (create, update, delete) trigger @CacheEvict to ensure cache consistency.

Cache keys are designed to be unique based on query parameters to prevent cache pollution.

## Rate Limiting

Rate limiting is implemented using Bucket4j with in-memory ConcurrentHashMap storage per IP address.

**Limits:**

- Login endpoint: 5 attempts per minute per IP

- Register endpoint: 5 attempts per 10 minutes per IP

- Forgot Password endpoint: 3 attempts per 10 minutes per IP

The token bucket algorithm ensures fair usage and prevents brute force attacks on sensitive endpoints.

## Email System

NovaMart integrates with Brevo SMTP API for all email communications.

All emails are sent asynchronously using @Async to prevent blocking the main thread.

**Email Types:**

1. **Verification Email** - Contains 6-digit OTP (10 min expiry) and clickable verification link with HTML styling

2. **Password Reset OTP Email** - Contains 6-digit OTP (10 min expiry) for password reset

3. **Order Confirmation Email** - Full order summary table with items, quantities, prices, and total with animated HTML

4. **Daily Report Email** - Sent to admin at 9:00 AM daily with revenue, total orders, new users, and cancelled orders for the previous day

## Database Schema

See [docs/database-design.md](docs/database-design.md) for detailed database schema documentation.

**Entities (13 total):**

- User

- Role

- Cart

- CartItem

- Product

- Category

- Order

- OrderItem

- Payment

- Address

- RefreshToken

- EmailVerificationToken

- PasswordResetToken

## Performance Testing

Performance testing results and documentation are available in the [performance-testing/](performance-testing/) folder.

- [jmeter-test-plan.md](performance-testing/jmeter-test-plan.md) - JMeter test plan configuration

- [auth-api-results.png](performance-testing/auth-api-results.png) - Authentication API performance results

- [product-api-results.png](performance-testing/product-api-results.png) - Product API performance results

- [cart-api-results.png](performance-testing/cart-api-results.png) - Cart API performance results

- [order-api-results.png](performance-testing/order-api-results.png) - Order API performance results

- [performance-summary.md](performance-testing/performance-summary.md) - Overall performance summary

## Project Structure

```
novamart-showcase/
├── README.md
├── diagrams/
│   ├── system-architecture.png
│   ├── authentication-flow.png
│   ├── order-workflow.png
│   ├── redis-cache-flow.png
│   └── deployment-architecture.png
├── screenshots/
│   ├── home-page.png
│   ├── product-page.png
│   ├── cart-page.png
│   ├── checkout-page.png
│   ├── admin-dashboard.png
│   └── swagger-api.png
├── performance-testing/
│   ├── jmeter-test-plan.md
│   ├── auth-api-results.png
│   ├── product-api-results.png
│   ├── cart-api-results.png
│   ├── order-api-results.png
│   └── performance-summary.md
├── docs/
│   ├── project-overview.md
│   ├── tech-stack.md
│   ├── database-design.md
│   ├── api-overview.md
│   └── deployment-guide.md
├── assets/
│   ├── banner.png
│   ├── logo.png
│   └── demo.gif
└── LICENSE
```

## Local Setup / Getting Started

### Prerequisites

- Java 17 or higher

- Maven 3.6+

- PostgreSQL 14+

- Docker (optional, for containerized deployment)

### Environment Variables

Create a `.env` file or set the following environment variables:

```bash
SPRING_DATASOURCE_JDBC_URL=jdbc:postgresql://localhost:5432/novamart
SPRING_DATASOURCE_USERNAME=your_db_username
SPRING_DATASOURCE_PASSWORD=your_db_password
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
PORT=8080
```

### Running with Docker

```bash
# Build the Docker image
docker build -t novamart-backend .

# Run the container
docker run -p 8080:8080 --env-file .env novamart-backend
```

### Running without Docker

```bash
# Clone the repository
git clone <repository-url>
cd novamart-backend

# Build the project
mvn clean install

# Run the application
mvn spring-boot:run
```

The application will start on `http://localhost:8080`

## Environment Variables

| Variable | Description |
|----------|-------------|
| SPRING_DATASOURCE_JDBC_URL | PostgreSQL database JDBC URL |
| SPRING_DATASOURCE_USERNAME | Database username |
| SPRING_DATASOURCE_PASSWORD | Database password |
| GOOGLE_CLIENT_ID | Google OAuth2 client ID |
| GOOGLE_CLIENT_SECRET | Google OAuth2 client secret |
| PORT | Application port (default: 8080) |

## Deployment

NovaMart is deployed using a modern cloud-native architecture.

**Backend:**

- Hosted on Render as a Docker container

- Auto-scaling based on traffic

- HTTPS enabled by default

- Live URL: https://ecommercebackend-2be7.onrender.com

**Frontend:**

- Hosted on Vercel for optimal React performance

- Global CDN distribution

- Automatic deployments from Git

- Live URL: https://martnova.vercel.app

**Database:**

- PostgreSQL managed database on Render

- Automated backups

- High availability configuration

- Connection pooling optimized for Render free tier (max 3 connections, min idle 1)

**Deployment Process:**

1. Code pushed to Git repository

2. Render automatically builds Docker image

3. Container deployed to Render infrastructure

4. Frontend deployed to Vercel via Git integration

5. Environment variables configured in Render dashboard

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

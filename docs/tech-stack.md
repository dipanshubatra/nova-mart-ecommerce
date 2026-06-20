# NovaMart Technology Stack

## Overview

NovaMart is built on a modern, enterprise-grade technology stack designed for scalability, security, and performance. This document provides a comprehensive overview of all technologies used in the project, their purposes, and how they integrate to create a cohesive e-commerce platform.

## Backend Technologies

### Java 17

**Purpose**: Core application programming language

**Details**:
- Latest LTS (Long Term Support) version of Java
- Provides enhanced performance and security features
- Supports modern language features like records, pattern matching, and sealed classes
- Compatible with Spring Boot 3.x framework requirements

**Why Java 17**:
- Long-term support until 2029
- Improved garbage collection and memory management
- Better performance for enterprise applications
- Strong typing and compile-time error detection

### Spring Boot 3.0.1

**Purpose**: Application framework and dependency management

**Details**:
- Latest major version of Spring Boot with Jakarta EE 10+ support
- Auto-configuration reduces boilerplate code
- Embedded Tomcat server for easy deployment
- Comprehensive ecosystem of Spring projects

**Key Features Used**:
- Spring Boot Starter dependencies
- Actuator for application monitoring
- Configuration properties management
- Profile-based configuration (dev, prod)

### Spring Security

**Purpose**: Authentication and authorization framework

**Details**:
- Declarative security with annotations
- Integration with JWT for stateless authentication
- OAuth2 client for Google login integration
- BCrypt password encoding for secure password storage

**Security Features**:
- Role-based access control (RBAC)
- Method-level security with @PreAuthorize
- CSRF protection
- CORS configuration

### JWT (auth0 java-jwt 4.2.1)

**Purpose**: Token-based authentication

**Details**:
- HMAC256 algorithm for token signing
- Access tokens with short expiry (configured in environment)
- Refresh tokens with extended expiry (configured in environment)
- Token type claims to prevent token misuse
- **Security Note**: Exact token expiry times are configured via environment variables and should not be exposed in documentation

**Implementation**:
- JWTFilter for token validation
- Refresh token rotation strategy
- Token storage in database for revocation capability

### Spring OAuth2 Client

**Purpose**: Social login integration

**Details**:
- Google OAuth2 provider configuration
- OAuth2SuccessHandler for custom post-login logic
- Automatic user and cart creation on first login
- JWT token issuance after OAuth2 success

### Spring Data JPA

**Purpose**: Database abstraction layer

**Details**:
- Hibernate as JPA implementation
- Repository pattern for data access
- Automatic CRUD operations
- Query method generation

**Features Used**:
- Entity relationships (@OneToMany, @ManyToOne, etc.)
- Pagination and sorting support
- Custom query methods with @Query
- Transaction management with @Transactional

### PostgreSQL

**Purpose**: Relational database management system

**Details**:
- Render-hosted managed database
- ACID compliance for data integrity
- Advanced features like JSON support, full-text search
- Connection pooling with HikariCP

**Configuration**:
- HikariCP pool: max 3 connections, min idle 1
- Optimized for Render free tier
- DDL auto=update for schema management

### Spring Boot Cache + Redis (Upstash)

**Purpose**: Performance optimization through distributed caching and OTP storage

**Details**:
- Redis distributed cache implementation via Upstash (256MB)
- Spring Cache abstraction with Redis backend
- @Cacheable for method result caching
- @CacheEvict for cache invalidation
- Cache key generation based on method parameters
- Automatic TTL management and expiration
- Connection pooling for optimal performance
- **OTP Storage**: Email verification and password reset OTPs stored in Redis with TTL

**Redis Configuration**:
- Upstash Redis 256MB allocation
- Distributed caching for horizontal scalability
- Persistent cache storage across deployments
- High-performance key-value operations
- Automatic failover and redundancy

**Cached Operations**:
- Product listings and searches
- Order data by user
- User lists for admin
- Category-based product queries
- Authentication tokens and sessions
- **OTP tokens** for email verification and password reset

### Cloudinary

**Purpose**: Cloud image storage and CDN delivery

**Details**:
- Cloud-based image storage and management
- Automatic image optimization and transformation
- Global CDN for fast image delivery
- Secure image upload with API authentication
- On-the-fly image resizing and format conversion
- Responsive image generation

**Features Used**:
- Product image upload and storage
- Automatic image optimization
- CDN delivery for performance
- Secure URL generation
- Image transformation capabilities

**Configuration**:
- Cloud name for account identification
- API key and secret for authentication
- Upload presets for image processing
- Folder organization for product images

### Spring Boot Mail + Brevo SMTP API

**Purpose**: Transactional email delivery

**Details**:
- Brevo (formerly Sendinblue) REST API integration
- Asynchronous email sending with @Async
- HTML-styled email templates
- OTP generation and validation

**Email Types**:
- Verification emails with OTP and links
- Password reset notifications
- Order confirmation with summaries
- Daily business reports

### Spring Boot Validation

**Purpose**: Input validation

**Details**:
- JSR-380 validation annotations
- Custom validation constraints
- Automatic validation on @Valid annotated parameters
- Integration with Spring MVC

**Common Validations**:
- @NotNull, @NotBlank, @Size
- @Email, @Pattern
- Custom validators for business rules

### SpringDoc OpenAPI / Swagger UI 2.0.2

**Purpose**: API documentation

**Details**:
- Automatic API documentation generation
- Interactive Swagger UI for testing
- OpenAPI 3.0 specification
- Integration with Spring Boot controllers

**Features**:
- Request/response schema documentation
- Authentication configuration
- Grouped endpoints by controller
- Example request/response values

### Bucket4j 8.1.1

**Purpose**: Rate limiting implementation

**Details**:
- Token bucket algorithm implementation
- In-memory ConcurrentHashMap storage
- IP-based rate limiting
- Configurable limits per endpoint

**Rate Limits**:
- Login: 5 attempts per minute per IP
- Register: 5 attempts per 10 minutes per IP
- Forgot Password: 3 attempts per 10 minutes per IP

### ModelMapper 3.1.1

**Purpose**: Object mapping between DTOs and entities

**Details**:
- Automatic property mapping
- Custom mapping configuration
- Type conversion support
- Reduces manual mapping code

**Usage**:
- Entity to DTO conversion
- DTO to Entity conversion
- Nested object mapping
- Custom property name mapping

### Lombok 1.18.30

**Purpose**: Code generation to reduce boilerplate

**Details**:
- Compile-time annotation processing
- Generates getters, setters, constructors
- Builder pattern support
- Logging annotations

**Common Annotations**:
- @Data, @Getter, @Setter
- @AllArgsConstructor, @NoArgsConstructor
- @Builder
- @Slf4j for logging

### Spring Boot Scheduling (@Scheduled)

**Purpose**: Scheduled task execution

**Details**:
- Cron expression support
- Fixed rate and fixed delay options
- Time zone configuration
- Integration with Spring context

**Scheduled Tasks**:
- Daily report generation at 9:00 AM
- Abandoned cart cleanup
- Data synchronization tasks

### Spring Boot Async (@Async)

**Purpose**: Asynchronous method execution

**Details**:
- Non-blocking operation execution
- Thread pool configuration
- Future return type support
- Exception handling

**Async Operations**:
- Email sending via Brevo API
- Order confirmation emails
- Daily report generation

## Frontend Technologies

### React

**Purpose**: User interface framework

**Details**:
- Component-based architecture
- Virtual DOM for performance
- State management with hooks
- Modern JavaScript (ES6+) features

**Deployment**:
- Hosted on Vercel
- Global CDN distribution
- Automatic deployments from Git
- Optimized build process

## DevOps & Deployment

### Docker

**Purpose**: Containerization

**Details**:
- Dockerfile for application packaging
- Consistent deployment across environments
- Layer-based image optimization
- Multi-stage build support

**Benefits**:
- Environment parity
- Easy scaling
- Dependency isolation
- Reproducible builds

### Render

**Purpose**: Backend hosting platform

**Details**:
- Docker container deployment
- Auto-scaling based on traffic
- HTTPS enabled by default
- Environment variable management

**Features**:
- Automatic SSL certificates
- Log aggregation
- Health checks
- Rollback capabilities

### Vercel

**Purpose**: Frontend hosting platform

**Details**:
- React application deployment
- Global edge network
- Automatic optimizations
- Preview deployments

**Benefits**:
- Fast content delivery
- Automatic HTTPS
- Git integration
- Zero configuration needed

## Development Tools

### Maven

**Purpose**: Build and dependency management

**Details**:
- Project object model (pom.xml)
- Dependency resolution
- Build lifecycle management
- Plugin integration

**Key Plugins**:
- Spring Boot Maven Plugin
- Maven Compiler Plugin
- Maven Surefire Plugin for testing

### Git

**Purpose**: Version control

**Details**:
- Distributed version control
- Branch management
- Collaboration features
- Integration with CI/CD

## Security Libraries

### BCrypt

**Purpose**: Password hashing

**Details**:
- One-way password hashing
- Salt generation
- Adaptive hashing cost
- Built into Spring Security

**Security Benefits**:
- Protection against rainbow table attacks
- Slow hashing to prevent brute force
- Automatic salt management

## Monitoring & Observability

### Spring Boot Actuator

**Purpose**: Application monitoring

**Details**:
- Health check endpoints
- Metrics collection
- Environment information
- Thread dump capabilities

**Endpoints**:
- /actuator/health
- /actuator/metrics
- /actuator/info

## Technology Integration Diagram

The technologies in NovaMart work together as follows:

1. **Frontend (React)** communicates with **Backend (Spring Boot)** via REST API
2. **Spring Boot** uses **Spring Security** for authentication with **JWT** tokens
3. **Spring Data JPA** abstracts database operations with **PostgreSQL**
4. **Spring Boot Cache** optimizes performance by caching frequently accessed data
5. **Brevo API** handles email communications asynchronously
6. **Bucket4j** implements rate limiting for security
7. **Docker** containers the application for deployment on **Render**
8. **Vercel** hosts the React frontend with global CDN

## Version Management

All dependencies are managed through Maven's pom.xml file with specific version constraints to ensure compatibility and security:

- Regular security updates
- Dependency vulnerability scanning
- Semantic versioning for libraries
- Compatibility testing before upgrades

## Technology Selection Rationale

The technology stack was selected based on:

1. **Industry Adoption**: Widely used technologies with strong community support
2. **Performance**: Optimized for high-throughput e-commerce operations
3. **Security**: Enterprise-grade security features built-in
4. **Scalability**: Designed to handle growth in users and transactions
5. **Developer Experience**: Modern tools that improve productivity
6. **Cost Efficiency**: Open-source technologies with free tier hosting options

## Future Technology Considerations

Potential technology enhancements for future versions:

- Redis for distributed caching
- Elasticsearch for advanced search capabilities
- RabbitMQ or Kafka for message queuing
- Kubernetes for container orchestration
- GraphQL for flexible API queries
- Microservices architecture for better scalability

## Conclusion

NovaMart's technology stack represents a balanced combination of proven enterprise technologies and modern development practices. Each technology serves a specific purpose in creating a secure, performant, and maintainable e-commerce platform. The stack is designed to be both developer-friendly and production-ready, with clear upgrade paths and strong community support.

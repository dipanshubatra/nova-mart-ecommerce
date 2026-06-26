# NovaMart Project Overview

## Introduction

NovaMart is a production-ready, full-stack e-commerce platform designed to provide a complete online shopping experience. Built with modern technologies and best practices, it offers secure authentication, real-time inventory management, automated order processing, and comprehensive admin capabilities.

## Project Vision

The vision behind NovaMart is to create a scalable, secure, and user-friendly e-commerce solution that can serve as a foundation for building online stores of any size. The platform emphasizes security, performance, and ease of use for both customers and administrators.

## Core Objectives

- **Security First**: Implement robust authentication and authorization mechanisms with JWT and OAuth2
- **Performance Optimization**: Utilize caching strategies and efficient database queries
- **Real-time Inventory**: Manage stock levels with reservation system to prevent overselling
- **Automated Operations**: Handle email notifications, order processing, and reporting automatically
- **Scalability**: Design architecture that can grow with business needs
- **Developer Experience**: Provide clear API documentation and easy deployment options

## Target Users

### Customers
- Browse and search products by category or keyword
- Add items to cart with real-time stock availability
- Complete checkout with multiple payment methods
- Track order status and history
- Manage profile and addresses

### Administrators
- Manage product catalog and categories
- Handle user accounts and roles
- Process and update orders
- View daily business reports
- Monitor system performance

## Key Features Overview

### Authentication & Security
- Dual authentication system: JWT-based and Google OAuth2
- Email verification with OTP (stored in Redis) and clickable links
- Refresh token rotation for enhanced security
- Rate limiting to prevent abuse
- Role-based access control (RBAC)

### Product Management
- Category-based product organization
- Special price calculation with discounts
- Image upload for products via Cloudinary (cloud storage with CDN)
- Soft delete with data integrity
- Cached product listings for performance

### Shopping Cart
- Real-time stock reservation system
- Automatic total calculation
- Duplicate product prevention
- Abandoned cart cleanup

### Order Processing
- Stock validation before order placement
- Cashfree payment gateway integration
- 30-minute retry window for failed payments
- Payment status tracking via webhooks
- Shipping address snapshots
- Automated confirmation emails
- Order status workflow management

### Email Communications
- Verification emails with OTP
- Password reset notifications
- Order confirmation with detailed summaries
- Daily business reports for administrators

### Performance & Caching
- Spring Cache implementation with Redis backend
- Cached product listings and searches
- Cached order data
- OTP storage in Redis with TTL for email verification and password reset
- Cache eviction on data modifications

### Monitoring & Observability
- Grafana dashboards for real-time metrics visualization
- Loki log aggregation for centralized logging
- Application performance monitoring (CPU, memory, response times)
- Business metrics tracking (orders, payments, revenue)
- Configurable alerts for system health and KPIs
- Log correlation across services for debugging

### Database Design
- PostgreSQL for reliable data storage
- 13 entities covering all business aspects
- Optimized connection pooling
- Automatic schema updates

## Technology Philosophy

NovaMart leverages modern, industry-standard technologies to ensure reliability, maintainability, and performance:

- **Spring Boot 3**: Latest version of the popular Java framework for rapid development
- **React**: Modern JavaScript library for responsive user interfaces
- **PostgreSQL**: Robust relational database with advanced features
- **Docker**: Containerization for consistent deployment
- **Brevo API**: Reliable email service for transactional communications
- **Cashfree**: Secure payment gateway with retry mechanism
- **Grafana + Loki**: Comprehensive monitoring and log aggregation
- **Redis**: Distributed caching for performance optimization

## Architecture Principles

The system follows several key architectural principles:

1. **Separation of Concerns**: Clear boundaries between modules and responsibilities
2. **RESTful API Design**: Standard HTTP methods and status codes
3. **Stateless Authentication**: JWT tokens for scalable session management
4. **Asynchronous Processing**: Non-blocking email sending and scheduled tasks
5. **Caching Strategy**: Intelligent caching to reduce database load
6. **Rate Limiting**: Protect against abuse and ensure fair usage
7. **Monitoring**: Real-time observability with Grafana and Loki
8. **Payment Security**: Secure payment processing with Cashfree

## Deployment Strategy

NovaMart is designed for cloud-native deployment:

- **Backend**: Render (Docker container) with auto-scaling
- **Frontend**: Vercel with global CDN distribution
- **Database**: Managed PostgreSQL on Render with automated backups
- **Cache**: Redis on Upstash (256MB) for distributed caching
- **Monitoring**: Grafana + Loki for metrics and log aggregation
- **Payment**: Cashfree payment gateway integration
- **Environment Variables**: Secure configuration management

## Future Considerations

While the current implementation provides a solid foundation, potential enhancements could include:

- Advanced search with filters and sorting
- Product reviews and ratings
- Wishlist functionality
- Multiple payment gateway integrations
- Analytics and reporting dashboards with Grafana
- Mobile application development
- Multi-language support
- Advanced inventory management features
- Distributed tracing with Jaeger or Zipkin
- Message queuing with RabbitMQ or Kafka
- Elasticsearch for advanced search capabilities

## Conclusion

NovaMart represents a comprehensive e-commerce solution that balances functionality, security, and performance. Its modular design and modern technology stack make it suitable for both learning purposes and production use. The platform demonstrates best practices in Spring Boot development, REST API design, and cloud deployment strategies.

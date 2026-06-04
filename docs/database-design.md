# NovaMart Database Design

## Overview

NovaMart uses PostgreSQL as its primary database management system. The database schema consists of 13 entities that collectively support all e-commerce operations including user management, product catalog, shopping cart, order processing, and authentication. This document provides a comprehensive overview of the database design, entity relationships, and key design decisions.

## Database Configuration

### PostgreSQL Settings

- **Host**: Render-managed PostgreSQL
- **Connection Pool**: HikariCP with max 3 connections, min idle 1
- **DDL Strategy**: auto=update (automatic schema updates)
- **Transaction Management**: Spring @Transactional annotations
- **ORM**: Hibernate (JPA implementation)

### Connection Pooling

The HikariCP connection pool is optimized for Render's free tier:

- **Maximum Pool Size**: 3 connections
- **Minimum Idle Connections**: 1
- **Connection Timeout**: 30 seconds
- **Idle Timeout**: 10 minutes
- **Max Lifetime**: 30 minutes

## Entity Overview

NovaMart's database consists of 13 entities organized into the following functional groups:

1. **Authentication & Security**: User, Role, RefreshToken, EmailVerificationToken, PasswordResetToken
2. **Product Catalog**: Product, Category
3. **Shopping Cart**: Cart, CartItem
4. **Order Management**: Order, OrderItem, Payment
5. **User Data**: Address

## Entity Details

### 1. User

**Purpose**: Stores user account information and authentication credentials

**Key Fields**:
- `userId` (Primary Key) - Unique identifier
- `email` - User email address (unique)
- `password` - BCrypt hashed password
- `firstName` - User's first name
- `lastName` - User's last name
- `phoneNumber` - Contact number
- `role` - Reference to Role entity
- `cart` - One-to-one relationship with Cart
- `addresses` - One-to-many relationship with Address

**Relationships**:
- Many-to-one with Role
- One-to-one with Cart
- One-to-many with Address
- One-to-many with Order

**Indexes**:
- Unique index on email
- Index on role_id

**Design Notes**:
- Password stored as BCrypt hash for security
- Email serves as unique identifier
- Auto-creates cart on registration

### 2. Role

**Purpose**: Defines user roles for access control

**Key Fields**:
- `roleId` (Primary Key) - Unique identifier
- `roleName` - Role name (ADMIN, USER)

**Relationships**:
- One-to-many with User

**Predefined Roles**:
- ADMIN (roleId=101) - Full system access
- USER (roleId=102) - Standard customer access

**Design Notes**:
- Roles seeded via CommandLineRunner on startup
- Used for role-based access control (RBAC)

### 3. Cart

**Purpose**: Represents user's shopping cart

**Key Fields**:
- `cartId` (Primary Key) - Unique identifier
- `user` - Reference to User entity
- `cartItems` - One-to-many relationship with CartItem
- `totalPrice` - Calculated total of all items

**Relationships**:
- Many-to-one with User
- One-to-many with CartItem

**Design Notes**:
- Auto-created on user registration
- Total price recalculated on every operation
- Cleared after order placement

### 4. CartItem

**Purpose**: Individual items in a shopping cart

**Key Fields**:
- `cartItemId` (Primary Key) - Unique identifier
- `cart` - Reference to Cart entity
- `product` - Reference to Product entity
- `quantity` - Quantity of product in cart
- `price` - Price at time of adding to cart

**Relationships**:
- Many-to-one with Cart
- Many-to-one with Product

**Design Notes**:
- Prevents duplicate products in same cart
- Quantity updates adjust product reservedQuantity
- Price snapshot at add time

### 5. Product

**Purpose**: Product catalog information

**Key Fields**:
- `productId` (Primary Key) - Unique identifier
- `productName` - Product name
- `description` - Product description
- `price` - Base price
- `discount` - Discount percentage
- `specialPrice` - Calculated price (price - discount%)
- `quantity` - Available stock quantity
- `reservedQuantity` - Quantity reserved in carts
- `category` - Reference to Category entity
- `image` - Product image filename
- `cartItems` - One-to-many relationship with CartItem
- `orderItems` - One-to-many relationship with OrderItem

**Relationships**:
- Many-to-one with Category
- One-to-many with CartItem
- One-to-many with OrderItem

**Indexes**:
- Index on category_id
- Index on product_name (for search)

**Design Notes**:
- Available stock = quantity - reservedQuantity
- Special price auto-calculated: price - (discount% * price)
- Soft delete nullifies order item references
- Image stored in images/ folder

### 6. Category

**Purpose**: Product categorization

**Key Fields**:
- `categoryId` (Primary Key) - Unique identifier
- `categoryName` - Category name
- `products` - One-to-many relationship with Product

**Relationships**:
- One-to-many with Product

**Design Notes**:
- Hierarchical structure possible (not implemented in current version)
- Used for product organization and filtering

### 7. Order

**Purpose**: Customer order information

**Key Fields**:
- `orderId` (Primary Key) - Unique identifier
- `user` - Reference to User entity
- `orderItems` - One-to-many relationship with OrderItem
- `payment` - Reference to Payment entity
- `orderStatus` - Current status ("Order Accepted !", SHIPPED, DELIVERED)
- `orderDate` - Order creation timestamp
- `shippingAddress` - Snapshot of shipping address
- `totalAmount` - Order total

**Relationships**:
- Many-to-one with User
- One-to-many with OrderItem
- One-to-one with Payment

**Indexes**:
- Index on user_id
- Index on order_date
- Index on order_status

**Design Notes**:
- Shipping address snapshot preserves address at order time
- Order statuses are custom strings
- Cached by user email for performance

### 8. OrderItem

**Purpose**: Individual items in an order

**Key Fields**:
- `orderItemId` (Primary Key) - Unique identifier
- `order` - Reference to Order entity
- `product` - Reference to Product entity (nullable for soft delete)
- `quantity` - Quantity ordered
- `price` - Price at time of order
- `productName` - Product name snapshot
- `productImage` - Product image snapshot

**Relationships**:
- Many-to-one with Order
- Many-to-one with Product (nullable)

**Design Notes**:
- Product reference nullable for soft delete support
- Snapshots preserve product data at order time
- Allows order history even if product deleted

### 9. Payment

**Purpose**: Payment transaction information

**Key Fields**:
- `paymentId` (Primary Key) - Unique identifier
- `order` - Reference to Order entity
- `paymentMethod` - Payment method used
- `paymentDate` - Payment timestamp
- `paymentStatus` - Payment status
- `amount` - Payment amount

**Relationships**:
- One-to-one with Order

**Design Notes**:
- One-to-one relationship with Order
- Tracks payment method and status
- Amount matches order total

### 10. Address

**Purpose**: User shipping/billing addresses

**Key Fields**:
- `addressId` (Primary Key) - Unique identifier
- `streetAddress` - Street address
- `city` - City name
- `state` - State/province
- `postalCode` - Postal/ZIP code
- `country` - Country name
- `users` - Many-to-many relationship with User

**Relationships**:
- Many-to-many with User

**Design Notes**:
- Deduplication: if same address exists, reassigns users
- Delete removes from all associated users
- Used for shipping and billing

### 11. RefreshToken

**Purpose**: JWT refresh token storage for token rotation

**Key Fields**:
- `tokenId` (Primary Key) - Unique identifier
- `token` - Refresh token string
- `user` - Reference to User entity
- `expiryDate` - Token expiration timestamp

**Relationships**:
- Many-to-one with User

**Design Notes**:
- Expiry configured via environment variable
- Rotated on every refresh
- All tokens invalidated on password reset
- Stored in database for revocation capability
- **Security Note**: Exact expiry time should be configured in environment variables

### 12. EmailVerificationToken

**Purpose**: Email verification token storage

**Key Fields**:
- `tokenId` (Primary Key) - Unique identifier
- `token` - Verification token string
- `user` - Reference to User entity
- `expiryDate` - Token expiration timestamp

**Relationships**:
- Many-to-one with User

**Design Notes**:
- 10-minute expiry
- Used for email verification via link
- Single-use token

### 13. PasswordResetToken

**Purpose**: Password reset token storage

**Key Fields**:
- `tokenId` (Primary Key) - Unique identifier
- `token` - Reset token string
- `user` - Reference to User entity
- `expiryDate` - Token expiration timestamp

**Relationships**:
- Many-to-one with User

**Design Notes**:
- Expiry configured via environment variable
- OTP length configured via environment
- Max attempts configured via environment
- Invalidates all refresh tokens on use
- **Security Note**: Exact expiry, OTP length, and attempt limits should be configured in environment variables

## Entity Relationship Diagram

```
User (1) ----< (1) Cart
  |
  +----< (1..*) Address
  |
  +----< (1..*) Order
  |
  +----< (1..*) RefreshToken
  |
  +----< (1) EmailVerificationToken
  |
  +----< (1) PasswordResetToken
  |
  +----> (1) Role

Cart (1) ----< (1..*) CartItem
CartItem (1) ----> (1) Product

Order (1) ----< (1..*) OrderItem
Order (1) ----> (1) Payment
OrderItem (1) ----> (0..1) Product

Category (1) ----< (1..*) Product
```

## Database Constraints

### Primary Keys
- All entities have auto-generated primary keys
- Uses `@GeneratedValue(strategy = GenerationType.IDENTITY)`

### Foreign Keys
- All relationships use foreign key constraints
- Cascading rules:
  - User deletion cascades to cart, tokens
  - Cart deletion cascades to cart items
  - Order deletion cascades to order items
  - Category deletion restricts if products exist

### Unique Constraints
- User.email - unique
- Role.roleName - unique
- Category.categoryName - unique (optional)

### Not Null Constraints
- All required fields marked with `@NotNull`
- Database-level validation

## Indexing Strategy

### Performance Indexes
- User.email (unique)
- Product.category_id
- Product.product_name (for search)
- Order.user_id
- Order.order_date
- Order.order_status

### Index Maintenance
- Automatic index creation by Hibernate
- Indexes updated on data modifications
- Optimized for common query patterns

## Transaction Management

### Transaction Boundaries
- Service layer methods marked with `@Transactional`
- Read-only transactions for queries
- Write transactions for modifications

### Isolation Level
- Default READ_COMMITTED
- Prevents dirty reads
- Allows non-repeatable reads (acceptable for e-commerce)

### Rollback Behavior
- Automatic rollback on RuntimeException
- Manual rollback for business rule violations

## Data Integrity

### Referential Integrity
- Foreign key constraints enforced by database
- Cascade rules maintain consistency
- Soft delete for products to preserve order history

### Business Rule Validation
- Application-level validation before database operations
- Stock availability checks before cart operations
- Order validation before placement

## Caching Strategy

### Cached Entities
- Product listings
- Order data by user
- User lists for admin
- Category-based queries

### Cache Eviction
- Automatic eviction on write operations
- @CacheEvict annotations on mutating methods
- Manual cache clearing when needed

## Database Security

### Access Control
- Database credentials stored in environment variables
- Least privilege principle applied
- Connection from authorized IPs only

### Data Encryption
- Passwords hashed with BCrypt
- JWT tokens signed with HMAC256
- SSL/TLS for database connections

## Backup Strategy

### Render Managed Backups
- Automated daily backups
- Point-in-time recovery
- 7-day retention period

### Backup Verification
- Regular backup integrity checks
- Restore testing procedures
- Backup monitoring alerts

## Performance Optimization

### Query Optimization
- N+1 query prevention with JOIN FETCH
- Pagination for large result sets
- Lazy loading where appropriate

### Connection Pooling
- HikariCP for efficient connection management
- Configured for Render free tier limits
- Connection timeout and reuse

### Caching
- Spring Cache for frequently accessed data
- Cache keys based on query parameters
- Cache eviction on data changes

## Migration Strategy

### Schema Updates
- DDL auto=update for development
- Version-controlled migration scripts for production
- Backward compatibility considerations

### Data Migration
- Export/import utilities
- Data transformation scripts
- Migration testing procedures

## Monitoring

### Performance Monitoring
- Query execution time tracking
- Slow query identification
- Index usage statistics

### Health Monitoring
- Connection pool metrics
- Database size monitoring
- Query throughput tracking

## Future Enhancements

### Potential Database Improvements
- Partitioning for large order tables
- Read replicas for reporting queries
- Full-text search with PostgreSQL FTS
- JSON columns for flexible metadata
- Database sharding for horizontal scaling

### Advanced Features
- Audit logging for compliance
- Data archiving for old orders
- Geographic data distribution
- Multi-tenancy support

## Conclusion

NovaMart's database design provides a solid foundation for e-commerce operations with proper normalization, relationships, and constraints. The schema supports all business requirements while maintaining data integrity and performance. The design follows best practices for Spring Boot JPA applications and is optimized for the PostgreSQL database on Render's infrastructure.

The 13 entities work together to create a comprehensive data model that handles user management, product catalog, shopping cart, order processing, and authentication in a secure and efficient manner.

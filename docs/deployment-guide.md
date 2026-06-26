# NovaMart Deployment Guide

## Overview

This guide provides comprehensive instructions for deploying NovaMart to production environments. NovaMart is designed for cloud-native deployment with separate hosting for backend, frontend, and database components. The current production deployment uses Render for backend hosting, Vercel for frontend hosting, and Render's managed PostgreSQL service for the database.

## Architecture Overview

NovaMart follows a modern microservices-inspired architecture with clear separation of concerns:

- **Backend**: Spring Boot application deployed as Docker container on Render
- **Frontend**: React application deployed on Vercel
- **Database**: PostgreSQL managed database on Render
- **Cache**: Redis distributed cache on Upstash (256MB) for performance optimization
- **Email Service**: Brevo API integration for transactional emails

## Prerequisites

### Required Accounts

1. **Render Account** (https://render.com)
   - Free tier available
   - Required for backend and database hosting

2. **Vercel Account** (https://vercel.com)
   - Free tier available
   - Required for frontend hosting

3. **Upstash Account** (https://upstash.com)
   - Free tier available (256MB Redis)
   - Required for distributed caching

4. **Brevo Account** (https://www.brevo.com)
   - Free tier available
   - Required for email services

5. **Google Cloud Console** (https://console.cloud.google.com)
   - Required for Google OAuth2 setup
   - Free tier available

6. **Cloudinary Account** (https://cloudinary.com)
   - Free tier available
   - Required for cloud image storage and CDN

7. **Cashfree Account** (https://www.cashfree.com)
   - Free sandbox tier available
   - Required for payment processing

8. **Grafana Account** (https://grafana.com)
   - Free tier available
   - Required for metrics visualization

9. **Loki** (can be self-hosted or cloud-hosted)
   - Required for log aggregation

### Required Tools

- Git
- Docker (for local testing)
- Maven 3.6+
- Java 17+
- Node.js 16+ (for frontend)
- PostgreSQL client (optional, for database management)

## Backend Deployment (Render)

### Step 1: Prepare the Backend

#### 1.1 Verify Dockerfile

Ensure your project has a `Dockerfile` in the root directory:

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/novamart-backend.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 1.2 Build the Application

```bash
# Navigate to project directory
cd novamart-backend

# Build with Maven
mvn clean package -DskipTests

# Verify JAR file exists
ls target/*.jar
```

#### 1.3 Test Locally with Docker

```bash
# Build Docker image
docker build -t novamart-backend .

# Run container
docker run -p 8080:8080 \
  -e SPRING_DATASOURCE_JDBC_URL=jdbc:postgresql://localhost:5432/novamart \
  -e SPRING_DATASOURCE_USERNAME=your_username \
  -e SPRING_DATASOURCE_PASSWORD=your_password \
  novamart-backend

# Test the application
curl http://localhost:8080/api/public/products
```

### Step 2: Set Up Render

#### 2.1 Create Render Account

1. Visit https://render.com
2. Sign up using GitHub, GitLab, or email
3. Verify your email address

#### 2.2 Create PostgreSQL Database

1. Log in to Render dashboard
2. Click "New +" → "PostgreSQL"
3. Configure database:
   - **Name**: novamart-db
   - **Database**: novamart
   - **User**: novamart_user
   - **Region**: Choose nearest region
   - **Plan**: Free (recommended for development)
4. Click "Create Database"
5. Wait for database to be provisioned (2-3 minutes)
6. Note the following credentials:
   - Internal Database URL
   - External Database URL
   - Database User
   - Database Password
   - Database Name

#### 2.3 Configure Database Connection Pooling

The application is optimized for Render's free tier with these HikariCP settings:

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 3
      minimum-idle: 1
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

### Step 2.5: Set Up Upstash Redis

#### 2.5.1 Create Upstash Account

1. Visit https://upstash.com
2. Sign up using GitHub, GitLab, or email
3. Verify your email address

#### 2.5.2 Create Redis Database

1. Log in to Upstash dashboard
2. Click "Create Database"
3. Configure database:
   - **Name**: novamart-redis
   - **Region**: Choose nearest region (prefer same as Render backend)
   - **Plan**: Free (256MB - recommended for development)
4. Click "Create"
5. Wait for database to be provisioned (1-2 minutes)
6. Note the following credentials:
   - REST API URL
   - REST API Token
   - Redis URL (for direct connections)

#### 2.5.3 Configure Redis Connection

The application uses Spring Data Redis with Lettuce client for optimal performance:

```yaml
spring:
  redis:
    host: ${SPRING_REDIS_HOST}
    port: ${SPRING_REDIS_PORT}
    password: ${SPRING_REDIS_PASSWORD}
    timeout: 2000ms
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: -1ms
```

**Redis Configuration Benefits**:
- Distributed caching for horizontal scalability
- Persistent cache storage across deployments
- High-performance key-value operations
- Automatic failover and redundancy
- Connection pooling for optimal performance

### Step 3: Deploy Backend to Render

#### 3.1 Connect Git Repository

1. Push your backend code to GitHub/GitLab
2. In Render dashboard, click "New +" → "Web Service"
3. Connect your Git repository
4. Select the branch to deploy (main/master)

#### 3.2 Configure Build Settings

**Build & Deploy** settings:

- **Runtime**: Docker
- **Docker Context**: . (root directory)
- **Dockerfile Path**: Dockerfile

#### 3.3 Configure Environment Variables

Add the following environment variables in Render:

```bash
# Database Configuration
SPRING_DATASOURCE_JDBC_URL=jdbc:postgresql://<internal-db-url>:5432/novamart
SPRING_DATASOURCE_USERNAME=<database-user>
SPRING_DATASOURCE_PASSWORD=<database-password>

# Server Configuration
PORT=8080

# Redis Configuration (Upstash)
SPRING_REDIS_HOST=<upstash-redis-host>
SPRING_REDIS_PORT=<upstash-redis-port>
SPRING_REDIS_PASSWORD=<upstash-redis-password>

# Cloudinary Configuration
CLOUDINARY_CLOUD_NAME=<your-cloudinary-cloud-name>
CLOUDINARY_API_KEY=<your-cloudinary-api-key>
CLOUDINARY_API_SECRET=<your-cloudinary-api-secret>

# Google OAuth2 (if using)
GOOGLE_CLIENT_ID=<your-google-client-id>
GOOGLE_CLIENT_SECRET=<your-google-client-secret>

# Brevo Email Configuration
BREVO_API_KEY=<your-brevo-api-key>
BREVO_SENDER_EMAIL=<your-sender-email>
BREVO_SENDER_NAME=<your-sender-name>

# Cashfree Payment Configuration
CASHFREE_APP_ID=<your-cashfree-app-id>
CASHFREE_SECRET_KEY=<your-cashfree-secret-key>
CASHFREE_API_URL=https://sandbox.cashfree.com/api/v2

# Monitoring Configuration
GRAFANA_URL=<your-grafana-url>
LOKI_URL=<your-loki-url>
PROMETHEUS_URL=<your-prometheus-url>

# JWT Configuration
JWT_SECRET=<your-jwt-secret-key>
JWT_EXPIRATION=<configured-in-milliseconds>
REFRESH_TOKEN_EXPIRATION=<configured-in-milliseconds>
```
**Security Note**: Exact expiration values should be configured based on your security requirements and not exposed in documentation

**Important Notes**:
- Use Render's internal database URL for better performance
- Generate a secure JWT_SECRET (at least 32 characters)
- Keep all secrets secure and never commit to Git

#### 3.4 Deploy the Application

1. Click "Create Web Service"
2. Render will automatically:
   - Build the Docker image
   - Deploy the container
   - Start the application
3. Wait for deployment to complete (3-5 minutes)
4. Monitor deployment logs in Render dashboard

#### 3.5 Verify Deployment

1. Check the service URL (e.g., https://novamart-backend.onrender.com)
2. Test the health endpoint:
```bash
curl https://novamart-backend.onrender.com/actuator/health
```
3. Test API endpoint:
```bash
curl https://novamart-backend.onrender.com/api/public/products
```

### Step 4: Configure Google OAuth2 (Optional)

#### 4.1 Create Google OAuth2 Application

1. Visit Google Cloud Console
2. Create a new project or select existing
3. Navigate to APIs & Services → Credentials
4. Click "Create Credentials" → "OAuth client ID"
5. Configure:
   - Application type: Web application
   - Authorized redirect URIs: https://novamart-backend.onrender.com/oauth2/callback
6. Note Client ID and Client Secret

#### 4.2 Update Environment Variables

Add Google OAuth2 credentials to Render:

```bash
GOOGLE_CLIENT_ID=<your-client-id>
GOOGLE_CLIENT_SECRET=<your-client-secret>
```

### Step 5: Configure Cloudinary Image Service

#### 5.1 Create Cloudinary Account

1. Visit https://cloudinary.com
2. Sign up for free account
3. Verify your email address
4. Navigate to Dashboard → Settings
5. Note the following credentials:
   - Cloud Name
   - API Key
   - API Secret

#### 5.2 Configure Image Upload Settings

Cloudinary supports automatic image optimization and transformation:

- Upload presets for image processing
- Automatic format optimization
- Responsive image generation
- CDN delivery for fast image access

#### 5.3 Update Environment Variables

Add Cloudinary credentials to Render:

```bash
CLOUDINARY_CLOUD_NAME=<your-cloud-name>
CLOUDINARY_API_KEY=<your-api-key>
CLOUDINARY_API_SECRET=<your-api-secret>
```

### Step 6: Configure Brevo Email Service

#### 6.1 Create Brevo Account

1. Visit https://www.brevo.com
2. Sign up for free account
3. Verify your email address
4. Navigate to SMTP & API → API Keys
5. Create new API key
6. Note the API key

#### 6.2 Configure Email Templates

Brevo supports HTML email templates. Configure templates for:
- Email verification (with OTP and link)
- Password reset (with OTP)
- Order confirmation (with order details)
- Daily report (with business metrics)

#### 6.3 Update Environment Variables

Add Brevo credentials to Render:

```bash
BREVO_API_KEY=<your-api-key>
BREVO_SENDER_EMAIL=noreply@yourdomain.com
BREVO_SENDER_NAME=NovaMart
```

### Step 7: Configure Cashfree Payment Gateway

#### 7.1 Create Cashfree Account

1. Visit https://www.cashfree.com
2. Sign up for sandbox account (free)
3. Verify your email address
4. Navigate to Dashboard → Settings → API Keys
5. Note the following credentials:
   - App ID
   - Secret Key

#### 7.2 Configure Payment Settings

Cashfree supports multiple payment methods:
- UPI (Unified Payments Interface)
- Credit/Debit Cards
- Net Banking
- Wallets (Paytm, PhonePe, etc.)
- EMI options

#### 7.3 Update Environment Variables

Add Cashfree credentials to Render:

```bash
CASHFREE_APP_ID=<your-app-id>
CASHFREE_SECRET_KEY=<your-secret-key>
CASHFREE_API_URL=https://sandbox.cashfree.com/api/v2
```

#### 7.4 Configure Webhooks

1. In Cashfree dashboard, navigate to Webhooks
2. Add webhook URL: `https://your-backend.onrender.com/api/payments/cashfree/callback`
3. Select events to monitor:
   - Payment Success
   - Payment Failed
   - Payment Expired
4. Save webhook configuration

### Step 8: Configure Grafana Monitoring

#### 8.1 Set Up Grafana

**Option 1: Grafana Cloud (Recommended)**

1. Visit https://grafana.com/products/cloud/
2. Sign up for free account
3. Create a new stack
4. Note the Grafana URL and API credentials

**Option 2: Self-Hosted Grafana**

```bash
# Using Docker
docker run -d -p 3000:3000 --name=grafana grafana/grafana

# Access at http://localhost:3000
# Default credentials: admin/admin
```

#### 8.2 Configure Data Sources

1. Log in to Grafana
2. Navigate to Configuration → Data Sources
3. Add Prometheus as data source for metrics
4. Add Loki as data source for logs
5. Configure connection URLs and authentication

#### 8.3 Create Dashboards

**Application Metrics Dashboard**:
- CPU usage
- Memory usage
- Response times
- Error rates
- Request throughput

**Business Metrics Dashboard**:
- Order volume
- Payment success rates
- User registrations
- Revenue tracking

#### 8.4 Configure Alerts

1. Navigate to Alerting → New Alert Rule
2. Set up alerts for:
   - High error rates (>5%)
   - Slow response times (>2s)
   - Payment failures (>10%)
   - Database connection issues
3. Configure notification channels (email, Slack, etc.)

### Step 9: Configure Loki Log Aggregation

#### 9.1 Set Up Loki

**Option 1: Grafana Cloud Loki**

Included with Grafana Cloud stack.

**Option 2: Self-Hosted Loki**

```bash
# Using Docker
docker run -d -p 3100:3100 --name=loki grafana/loki
```

#### 9.2 Configure Application Logging

Add Loki configuration to `application.properties`:

```properties
# Logging configuration for Loki
logging.level.root=INFO
logging.level.com.novamart=DEBUG
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
```

#### 9.3 Install Promtail (Log Agent)

```bash
# Using Docker
docker run -d --name=promtail \
  -v /path/to/promtail-config.yml:/etc/promtail/config.yml \
  grafana/promtail
```

Configure `promtail-config.yml`:

```yaml
server:
  http_listen_port: 9080

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: novamart-logs
    static_configs:
      - targets:
          - localhost
        labels:
          app: novamart
          env: production
```

## Frontend Deployment (Vercel)

### Step 1: Prepare the Frontend

#### 1.1 Verify React Application

Ensure your React application is properly configured:

```javascript
// src/config/api.js
export const API_BASE_URL = 'https://novamart-backend.onrender.com';
```

#### 1.2 Build Locally

```bash
# Navigate to frontend directory
cd novamart-frontend

# Install dependencies
npm install

# Build for production
npm run build

# Test production build locally
npm run preview
```

### Step 2: Set Up Vercel

#### 2.1 Create Vercel Account

1. Visit https://vercel.com
2. Sign up using GitHub, GitLab, or email
3. Verify your email address

#### 2.2 Deploy to Vercel

**Option 1: Vercel CLI**

```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Deploy
vercel

# Deploy to production
vercel --prod
```

**Option 2: Vercel Dashboard**

1. Log in to Vercel dashboard
2. Click "Add New..." → "Project"
3. Import your Git repository
4. Configure:
   - **Framework Preset**: React
   - **Build Command**: npm run build
   - **Output Directory**: build
   - **Install Command**: npm install
5. Add environment variables (if needed):
   - `REACT_APP_API_BASE_URL`: https://novamart-backend.onrender.com
6. Click "Deploy"

#### 2.3 Configure Domain

1. In Vercel project settings
2. Navigate to "Domains"
3. Add custom domain (optional)
4. Configure DNS records as instructed by Vercel

### Step 3: Verify Frontend Deployment

1. Visit the deployed URL (e.g., https://martnova.vercel.app)
2. Test user registration
3. Test product browsing
4. Test cart functionality
5. Test checkout process

## Database Management

### Connecting to Database

#### Using pgAdmin

1. Install pgAdmin (PostgreSQL management tool)
2. Create new server connection
3. Use Render's external database URL
4. Connect and manage database

#### Using Command Line

```bash
# Install PostgreSQL client
# On Ubuntu/Debian
sudo apt-get install postgresql-client

# On macOS
brew install postgresql

# Connect to database
psql -h <external-db-host> -U <database-user> -d novamart
```

### Database Backups

Render automatically creates daily backups. To manually backup:

```bash
# Export database
pg_dump -h <external-db-host> -U <database-user> novamart > backup.sql

# Restore database
psql -h <external-db-host> -U <database-user> novamart < backup.sql
```

### Database Migrations

For schema changes, use Flyway or Liquibase:

```bash
# Add Flyway dependency to pom.xml
# Create migration scripts in src/main/resources/db/migration
# Run migrations on startup
```

## Environment Variables Reference

### Backend Environment Variables

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| SPRING_DATASOURCE_JDBC_URL | PostgreSQL JDBC URL | Yes | jdbc:postgresql://dpg-xxx.oregon-postgres.render.com:5432/novamart |
| SPRING_DATASOURCE_USERNAME | Database username | Yes | novamart_user |
| SPRING_DATASOURCE_PASSWORD | Database password | Yes | secure_password |
| PORT | Application port | Yes | 8080 |
| SPRING_REDIS_HOST | Upstash Redis host | Yes | redis-xxx.upstash.io |
| SPRING_REDIS_PORT | Upstash Redis port | Yes | 6379 |
| SPRING_REDIS_PASSWORD | Upstash Redis password | Yes | your-redis-password |
| CLOUDINARY_CLOUD_NAME | Cloudinary cloud name | Yes | your-cloud-name |
| CLOUDINARY_API_KEY | Cloudinary API key | Yes | xxxxxxxxxxxxxx |
| CLOUDINARY_API_SECRET | Cloudinary API secret | Yes | xxxxxxxxxxxxxx |
| GOOGLE_CLIENT_ID | Google OAuth2 Client ID | Optional | xxx.apps.googleusercontent.com |
| GOOGLE_CLIENT_SECRET | Google OAuth2 Client Secret | Optional | GOCSPX-xxx |
| BREVO_API_KEY | Brevo API key | Yes | xkeysib-xxx |
| BREVO_SENDER_EMAIL | Brevo sender email | Yes | noreply@novamart.com |
| BREVO_SENDER_NAME | Brevo sender name | Yes | NovaMart |
| CASHFREE_APP_ID | Cashfree application ID | Yes | your-app-id |
| CASHFREE_SECRET_KEY | Cashfree secret key | Yes | your-secret-key |
| CASHFREE_API_URL | Cashfree API URL | Yes | https://sandbox.cashfree.com/api/v2 |
| GRAFANA_URL | Grafana dashboard URL | Yes | https://your-grafana-instance.com |
| LOKI_URL | Loki log aggregation URL | Yes | http://your-loki-instance:3100 |
| PROMETHEUS_URL | Prometheus metrics URL | Optional | http://your-prometheus:9090 |
| JWT_SECRET | JWT signing secret | Yes | your-secure-secret-key-min-32-chars |
| JWT_EXPIRATION | JWT access token expiry (ms) | No | <configured-in-milliseconds> |
| REFRESH_TOKEN_EXPIRATION | Refresh token expiry (ms) | No | <configured-in-milliseconds> |
- **Security Note**: Exact expiration values should be configured based on security requirements

### Frontend Environment Variables

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| REACT_APP_API_BASE_URL | Backend API URL | Yes | https://novamart-backend.onrender.com |

## Monitoring and Logging

### Render Monitoring

Render provides built-in monitoring:

- **Metrics**: CPU, memory, response time
- **Logs**: Real-time and historical logs
- **Alerts**: Configure alerts for errors and performance

Access monitoring in Render dashboard under your service.

### Vercel Monitoring

Vercel provides:

- **Analytics**: Page views, visitors, performance
- **Logs**: Build and deployment logs
- **Speed Insights**: Core Web Vitals

Access monitoring in Vercel dashboard under your project.

### Application Logging

Configure logging in `application.properties`:

```properties
# Logging configuration
logging.level.root=INFO
logging.level.com.novamart=DEBUG
logging.level.org.springframework.web=INFO
logging.level.org.hibernate=INFO

# Loki integration (if using Promtail)
logging.file.name=/var/log/novamart/application.log
logging.logback.rollingpolicy.max-file-size=10MB
logging.logback.rollingpolicy.max-history=30
```

### Metrics Collection

Spring Boot Actuator provides metrics that can be scraped by Prometheus:

```properties
# Enable actuator endpoints
management.endpoints.web.exposure.include=health,metrics,info,prometheus
management.metrics.export.prometheus.enabled=true
```

Access metrics at `/actuator/prometheus` endpoint.

## Security Best Practices

### Backend Security

1. **Environment Variables**
   - Never commit secrets to Git
   - Use Render's environment variable management
   - Rotate secrets regularly

2. **HTTPS**
   - Render provides automatic SSL
   - Force HTTPS redirects
   - Use secure cookies

3. **Database Security**
   - Use strong passwords
   - Limit database access to Render's internal network
   - Regular security updates

4. **API Security**
   - Implement rate limiting
   - Validate all inputs
   - Use JWT for authentication
   - Implement CORS properly

### Frontend Security

1. **Environment Variables**
   - Never expose backend secrets
   - Only expose public API URLs
   - Use build-time environment variables

2. **HTTPS**
   - Vercel provides automatic SSL
   - Force HTTPS redirects
   - Use secure cookies

3. **Content Security**
   - Implement CSP headers
   - Sanitize user inputs
   - Validate all API responses

## Scaling Considerations

### Backend Scaling

Render supports automatic scaling:

- **Free Tier**: Limited resources, suitable for development
- **Starter ($7/month)**: Better performance, suitable for small production
- **Standard ($25/month)**: More resources, suitable for growing applications
- **Pro ($100+/month)**: High performance, suitable for large applications

To scale:

1. Upgrade service plan in Render dashboard
2. Configure auto-scaling rules
3. Monitor performance metrics
4. Optimize database queries

### Database Scaling

PostgreSQL scaling options:

- **Vertical Scaling**: Upgrade to larger instance
- **Read Replicas**: Add read replicas for reporting
- **Connection Pooling**: Optimize connection pool size
- **Caching**: Implement Redis for caching

### Frontend Scaling

Vercel provides automatic scaling:

- **Edge Network**: Global CDN for fast content delivery
- **Serverless Functions**: Automatic scaling based on traffic
- **Build Optimization**: Optimize bundle size and loading

## Troubleshooting

### Common Backend Issues

#### Application Won't Start

**Symptoms**: Deployment fails, service shows "Crashed"

**Solutions**:
1. Check deployment logs in Render dashboard
2. Verify environment variables are set correctly
3. Ensure database is accessible
4. Check port configuration (PORT=8080)
5. Verify Dockerfile is correct

#### Database Connection Errors

**Symptoms**: Application starts but can't connect to database

**Solutions**:
1. Verify database credentials
2. Check database is running
3. Ensure network allows connection
4. Use internal database URL for better performance
5. Check connection pool settings

#### Email Not Sending

**Symptoms**: Emails not received

**Solutions**:
1. Verify Brevo API key is correct
2. Check sender email is verified in Brevo
3. Review Brevo dashboard for email logs
4. Ensure email templates are configured
5. Check spam folder

### Common Frontend Issues

#### Build Failures

**Symptoms**: Vercel build fails

**Solutions**:
1. Check build logs in Vercel dashboard
2. Ensure all dependencies are in package.json
3. Verify Node.js version compatibility
4. Check for environment variable references
5. Test build locally first

#### API Connection Errors

**Symptoms**: Frontend can't connect to backend

**Solutions**:
1. Verify REACT_APP_API_BASE_URL is correct
2. Check backend is running and accessible
3. Ensure CORS is configured on backend
4. Check browser console for errors
5. Test API endpoint directly

#### Performance Issues

**Symptoms**: Slow page loads, poor user experience

**Solutions**:
1. Optimize bundle size (code splitting)
2. Implement lazy loading
3. Optimize images
4. Use Vercel Analytics to identify bottlenecks
5. Consider implementing service worker

## CI/CD Pipeline

### Automated Backend Deployment

Configure GitHub Actions for automated deployment:

```yaml
name: Deploy to Render

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build with Maven
        run: mvn clean package -DskipTests
      - name: Deploy to Render
        run: |
          curl https://api.render.com/deploy/srv/${RENDER_SERVICE_ID} \
            -X POST \
            -H "Authorization: Bearer ${RENDER_API_KEY}"
```

### Automated Frontend Deployment

Vercel automatically deploys on Git push. Configure:

1. Connect Git repository to Vercel
2. Configure branch protection rules
3. Set up preview deployments for pull requests
4. Configure custom domains

## Maintenance

### Regular Maintenance Tasks

1. **Weekly**
   - Review application logs
   - Check error rates
   - Monitor performance metrics
   - Review security alerts

2. **Monthly**
   - Update dependencies
   - Review and rotate secrets
   - Check database storage usage
   - Review email deliverability

3. **Quarterly**
   - Full security audit
   - Performance optimization review
   - Backup verification
   - Disaster recovery testing

### Dependency Updates

Keep dependencies up to date:

```bash
# Backend - Check for updates
mvn versions:display-dependency-updates

# Frontend - Check for updates
npm outdated

# Update dependencies
npm update
```

### Backup Strategy

- **Database**: Render automatic daily backups
- **Code**: Git repository (GitHub/GitLab)
- **Configuration**: Document environment variables
- **Assets**: Store in cloud storage (AWS S3, etc.)

## Cost Optimization

### Render Cost Optimization

- Use free tier for development
- Optimize connection pool size
- Implement caching to reduce database load
- Monitor resource usage
- Scale down during low-traffic periods

### Vercel Cost Optimization

- Use free tier for development
- Optimize bundle size
- Implement image optimization
- Use edge functions for dynamic content
- Monitor bandwidth usage

### Database Cost Optimization

- Optimize queries
- Implement caching
- Archive old data
- Use connection pooling
- Monitor storage usage

## Disaster Recovery

### Backup Strategy

1. **Database Backups**
   - Render automatic daily backups
   - Manual weekly exports
   - Store backups in multiple locations

2. **Code Backups**
   - Git repository (primary)
   - Additional remote repository (backup)
   - Tag releases for easy rollback

3. **Configuration Backups**
   - Document all environment variables
   - Store configuration in version control
   - Use configuration management tools

### Recovery Procedures

1. **Database Recovery**
   - Restore from Render backup
   - Use pg_restore for manual backups
   - Verify data integrity

2. **Application Recovery**
   - Redeploy from Git repository
   - Restore environment variables
   - Verify functionality

3. **Frontend Recovery**
   - Redeploy from Git repository
   - Restore environment variables
   - Verify functionality

## Support and Resources

### Documentation

- [Render Documentation](https://render.com/docs)
- [Vercel Documentation](https://vercel.com/docs)
- [Brevo Documentation](https://developers.brevo.com/)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [React Documentation](https://react.dev/)

### Community

- Render Community: https://community.render.com
- Vercel Community: https://vercel.com/community
- Stack Overflow: Tag questions with novamart

### Troubleshooting

1. Check logs first
2. Review documentation
3. Search community forums
4. Create support ticket if needed

## Conclusion

NovaMart is designed for easy deployment to modern cloud platforms. By following this guide, you can successfully deploy the backend to Render, frontend to Vercel, and configure all necessary services. The platform is built to scale and can handle growing traffic with proper configuration and monitoring.

Regular maintenance, monitoring, and security updates will ensure the platform remains stable and secure in production. The cloud-native architecture allows for easy scaling and updates as your business grows.

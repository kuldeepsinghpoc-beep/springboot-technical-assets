# Authentication Service Requirements

## Project Overview

Develop a comprehensive Spring Boot authentication service focused on JWT-based authentication, user credential validation, and secure session management. This service handles login, logout, token generation, token validation, and authentication state management.

## Authentication Dependencies

```xml
<!-- Security Framework -->
- spring-boot-starter-security (Core security features)
- spring-security-config (Security configuration)
- spring-security-web (Web security features)

<!-- JWT Implementation -->
- jjwt-api (JWT API)
- jjwt-impl (JWT implementation)
- jjwt-jackson (JWT JSON processing)

<!-- User Service Communication -->
- spring-cloud-starter-openfeign (Service-to-service communication)
- spring-boot-starter-web (REST client capabilities)

<!-- Data & Validation -->
- spring-boot-starter-data-jpa (Token storage)
- spring-boot-starter-validation (Input validation)
```

## Core Authentication Components

### 1. Authentication Controller

#### A. AuthController.java

- POST `/api/auth/login` - User authentication with credentials
- POST `/api/auth/logout` - User logout with token invalidation
- POST `/api/auth/refresh` - Access token refresh using refresh token
- POST `/api/auth/validate` - Token validation for other services
- GET `/api/auth/me` - Current authenticated user information
- POST `/api/auth/revoke` - Token revocation (administrative)
- Comprehensive input validation for all authentication requests
- Rate limiting for brute force attack prevention
- Detailed audit logging for authentication attempts
- Complete OpenAPI/Swagger documentation for authentication endpoints

#### B. AuthInternalController.java

- Internal endpoints for service-to-service authentication validation
- POST `/internal/auth/validate-token` - Token validation for microservices
- POST `/internal/auth/revoke-user-tokens` - Revoke all tokens for specific user
- GET `/internal/auth/token-info` - Token metadata and claims information
- Service authentication and authorization

### 2. JWT Token Management

#### A. JwtUtil.java

Comprehensive JWT utility service for token operations

**Token Generation:**
- Access token generation with user claims and authorities
- Refresh token generation with extended expiry
- Custom claims addition (user ID, username, roles, permissions)
- Token signing with secure secret key
- Configurable token expiration times

**Token Validation:**
- Token signature verification
- Token expiration checking
- Token format and structure validation
- Claims extraction and validation
- Issuer and audience verification

**Token Operations:**
- Token refresh logic with validation
- Token revocation and blacklisting
- Token metadata extraction
- Custom claim processing

#### B. TokenBlacklistService.java

- Revoked token storage and management
- Token blacklist checking for validation
- Expired token cleanup scheduling
- Blacklist storage optimization (Redis recommended)
- Token revocation audit logging

### 3. Security Configuration

#### A. SecurityConfig.java

Comprehensive Spring Security configuration

**Authentication Configuration:**
- JWT authentication filter configuration
- Authentication entry point for unauthorized requests
- Password encoder configuration (BCrypt)
- Authentication manager setup
- Custom authentication providers

**Authorization Configuration:**
- HTTP security configuration with JWT filters
- Public endpoint configuration (login, health checks)
- Protected endpoint authorization rules
- CORS configuration for cross-origin requests
- CSRF protection configuration (disabled for stateless JWT)

**Session Management:**
- Stateless session configuration
- Session creation policy (NEVER/STATELESS)
- Concurrent session control (if needed)

#### B. JwtAuthenticationFilter.java

- Custom authentication filter for JWT processing
- Authorization header extraction and validation
- Token validation and user authentication setup
- Security context population with authenticated user
- Error handling for invalid or expired tokens
- Request/response logging for authentication audit

### 4. Authentication Service Layer

#### A. AuthenticationService.java

**User Authentication:**
- Username/password validation
- User account status verification (active, enabled, not locked)
- Failed login attempt tracking and account locking
- Multi-factor authentication support (optional)
- Authentication success/failure logging and metrics

**Token Management:**
- Access and refresh token generation
- Token validation and verification
- Token refresh logic with security checks
- Token revocation and blacklisting
- Token cleanup and maintenance

**User Service Integration:**
- User credential validation via User Management Service
- User profile information retrieval
- Last login timestamp updates
- User account status synchronization

#### B. UserDetailsServiceImpl.java

- Custom UserDetailsService implementation
- User loading from User Management Service
- UserDetails object creation with authorities
- User account status mapping
- Caching for performance optimization

### 5. Authentication DTOs

#### A. Authentication Request DTOs

- **LoginRequest.java** - User login credentials
  - Username or email (required, validated)
  - Password (required, validated)
  - Remember me option (optional)
  - Device information for tracking (optional)
- **RefreshTokenRequest.java** - Token refresh request
  - Refresh token (required, validated)
  - Device validation (optional)
- **TokenValidationRequest.java** - Token validation request
  - Access token (required)
  - Token type specification
- **LogoutRequest.java** - User logout request
  - Access token for revocation
  - Refresh token for revocation (optional)

#### B. Authentication Response DTOs

- **AuthenticationResponse.java** - Successful authentication response
  - Access token with expiration
  - Refresh token with expiration
  - Token type (Bearer)
  - User basic information (ID, username)
  - Authentication timestamp
- **TokenValidationResponse.java** - Token validation response
  - Validation status (valid/invalid/expired)
  - User information from token claims
  - Token expiration information
  - Validation timestamp
- **AuthErrorResponse.java** - Authentication error response
  - Error code and description
  - Timestamp and request ID
  - Detailed error information for debugging

### 6. User Service Integration

#### A. UserServiceClient.java

- Feign client for User Management Service communication
- User credential validation endpoints
- User profile information retrieval
- User account status checking and updates
- Service discovery and load balancing configuration
- Circuit breaker and retry logic
- Fallback mechanisms for service unavailability

#### B. UserServiceFallback.java

- Fallback implementation for User Service unavailability
- Cached user data utilization
- Graceful degradation of authentication features
- Error handling and user notification

## Authentication Configuration

### A. JWT Configuration Properties

```properties
# JWT Configuration
app.jwt.secret=${JWT_SECRET:mySecretKey12345678901234567890123456789012345678901234567890}
app.jwt.access-token-expiry=900000
app.jwt.refresh-token-expiry=86400000
app.jwt.issuer=authentication-service
app.jwt.audience=wipro-ai-services

# Token Configuration
app.auth.max-failed-attempts=5
app.auth.account-lock-duration=900000
app.auth.token-cleanup-interval=3600000
app.auth.remember-me-expiry=2592000000

# User Service Integration
app.user-service.base-url=http://localhost:8081
app.user-service.timeout=5000
app.user-service.retry-attempts=3
app.user-service.circuit-breaker-enabled=true

# Security Configuration
app.security.cors-allowed-origins=http://localhost:3000,http://localhost:4200
app.security.cors-allowed-methods=GET,POST,PUT,DELETE,OPTIONS
app.security.cors-allowed-headers=*
app.security.rate-limit-requests=100
app.security.rate-limit-window=3600
```

### B. Authentication Security Rules

#### Login Validation

- **Username/Email**: Required, 3-100 characters, valid format
- **Password**: Required, 6-100 characters, BCrypt comparison
- **Account Status**: Active, enabled, not expired, not locked
- **Rate Limiting**: Maximum attempts per IP/user
- **Brute Force Protection**: Account locking after failed attempts
- **Audit Logging**: All authentication attempts logged

#### Token Validation

- **Token Format**: Valid JWT structure with header, payload, signature
- **Token Signature**: Verified with service secret key
- **Token Expiration**: Not expired based on 'exp' claim
- **Token Claims**: Required claims present (sub, iat, exp, iss, aud)
- **Token Blacklist**: Not in revoked token blacklist
- **Issuer/Audience**: Matches configured values

## Authentication API Endpoints

### Endpoint Specifications

- **Base URL**: `/api/auth`
- **Internal Base URL**: `/internal/auth`
- **Content-Type**: `application/json`
- **Authentication**: Bearer token (except login and public endpoints)

#### Public Endpoints

- `POST /api/auth/login` - User authentication
  - Request: username/email and password
  - Response: access token, refresh token, user info
  - Rate limiting and brute force protection
- `POST /api/auth/refresh` - Token refresh
  - Request: refresh token
  - Response: new access token and refresh token
  - Refresh token validation and rotation

#### Protected Endpoints

- `POST /api/auth/logout` - User logout
  - Request: access token (from Authorization header)
  - Response: logout confirmation
  - Token blacklisting and cleanup
- `GET /api/auth/me` - Current user information
  - Request: access token (from Authorization header)
  - Response: user profile from token claims
- `POST /api/auth/validate` - Token validation (public for services)
  - Request: token to validate
  - Response: validation status and user info
- `POST /api/auth/revoke` - Administrative token revocation
  - Request: user ID or token to revoke
  - Response: revocation confirmation
  - Admin privileges required

#### Internal Service Endpoints

- `POST /internal/auth/validate-token` - Token validation for microservices
- `POST /internal/auth/revoke-user-tokens` - Revoke all user tokens
- `GET /internal/auth/token-info` - Token metadata and claims
- Service-to-service authentication required

## Authentication Testing Requirements

### A. Authentication Flow Tests

- User login with valid credentials
- User login with invalid credentials
- Account locking after failed attempts
- Token generation and validation
- Token refresh and rotation
- User logout and token revocation
- Concurrent session handling

### B. Security Tests

- JWT token signature validation
- Token expiration handling
- Token blacklist functionality
- Brute force attack protection
- CORS configuration testing
- Rate limiting validation
- SQL injection and XSS protection

### C. Integration Tests

- User Service integration testing
- Service-to-service authentication
- Database connectivity and operations
- Error handling and fallback mechanisms
- Performance and load testing
- Security vulnerability scanning

## Authentication Error Handling

### A. Authentication Exceptions

- **BadCredentialsException**: Invalid username or password
- **AccountLockedException**: Account locked due to failed attempts
- **AccountExpiredException**: User account has expired
- **DisabledException**: User account is disabled
- **TokenExpiredException**: JWT token has expired
- **InvalidTokenException**: Invalid or malformed JWT token
- **TokenRevokedException**: Token has been revoked/blacklisted
- **AuthenticationServiceException**: General authentication service errors

### B. Error Response Format

```json
{
  "timestamp": "2025-10-05T10:15:30.123Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Invalid credentials provided",
  "path": "/api/auth/login",
  "details": {
    "errorCode": "INVALID_CREDENTIALS",
    "attemptCount": 3,
    "lockoutTime": "2025-10-05T10:30:30.123Z",
    "retryAfter": 900
  }
}
```

## Authentication Service File Structure

```
authentication-service/
├── src/
│   ├── main/
│   │   ├── java/com/wipro/ai/demo/auth/
│   │   │   ├── controller/
│   │   │   │   ├── AuthController.java
│   │   │   │   └── AuthInternalController.java
│   │   │   ├── dto/
│   │   │   │   ├── request/
│   │   │   │   │   ├── LoginRequest.java
│   │   │   │   │   ├── RefreshTokenRequest.java
│   │   │   │   │   ├── TokenValidationRequest.java
│   │   │   │   │   └── LogoutRequest.java
│   │   │   │   └── response/
│   │   │   │       ├── AuthenticationResponse.java
│   │   │   │       ├── TokenValidationResponse.java
│   │   │   │       └── AuthErrorResponse.java
│   │   │   ├── exception/
│   │   │   │   ├── InvalidTokenException.java
│   │   │   │   ├── TokenExpiredException.java
│   │   │   │   ├── TokenRevokedException.java
│   │   │   │   └── AuthenticationServiceException.java
│   │   │   ├── filter/
│   │   │   │   └── JwtAuthenticationFilter.java
│   │   │   ├── model/
│   │   │   │   ├── TokenBlacklist.java
│   │   │   │   └── AuthenticationAudit.java
│   │   │   ├── repository/
│   │   │   │   ├── TokenBlacklistRepository.java
│   │   │   │   └── AuthenticationAuditRepository.java
│   │   │   ├── security/
│   │   │   │   ├── SecurityConfig.java
│   │   │   │   ├── JwtAuthenticationEntryPoint.java
│   │   │   │   └── JwtAccessDeniedHandler.java
│   │   │   ├── service/
│   │   │   │   ├── AuthenticationService.java
│   │   │   │   ├── JwtUtil.java
│   │   │   │   ├── TokenBlacklistService.java
│   │   │   │   ├── UserDetailsServiceImpl.java
│   │   │   │   ├── UserServiceClient.java
│   │   │   │   └── UserServiceFallback.java
│   │   │   └── config/
│   │   │       ├── AuthenticationConfig.java
│   │   │       ├── FeignConfig.java
│   │   │       └── JwtConfig.java
│   │   └── resources/
│   │       ├── application.properties
│   │       └── application-auth.properties
│   └── test/
│       ├── java/com/wipro/ai/demo/auth/
│       │   ├── controller/
│       │   │   ├── AuthControllerTest.java
│       │   │   └── AuthInternalControllerTest.java
│       │   ├── security/
│       │   │   ├── SecurityConfigTest.java
│       │   │   └── JwtAuthenticationFilterTest.java
│       │   ├── service/
│       │   │   ├── AuthenticationServiceTest.java
│       │   │   ├── JwtUtilTest.java
│       │   │   └── TokenBlacklistServiceTest.java
│       │   └── integration/
│       │       ├── AuthenticationIntegrationTest.java
│       │       └── UserServiceIntegrationTest.java
│       └── resources/
│           └── application-test.properties
├── postman/
│   ├── authentication-api-collection.json
│   └── authentication-environment.json
├── scripts/
│   ├── test-auth-api.bat
│   └── test-auth-api.ps1
└── docs/
    ├── AUTHENTICATION-API-GUIDE.md
    └── JWT-IMPLEMENTATION-GUIDE.md
```

## Authentication Documentation

### A. API Documentation

- Complete authentication endpoint documentation
- JWT token format and claims specification
- Authentication flow diagrams and examples
- Error handling and response format documentation
- Rate limiting and security configuration guides
- Service integration and API consumption examples

### B. Security Documentation

- JWT implementation and security considerations
- Token lifecycle management and best practices
- Brute force protection and account locking mechanisms
- CORS configuration and cross-origin security
- Service-to-service authentication protocols

## Expected Authentication Deliverables

1. **JWT Authentication System** with token generation, validation, and refresh
2. **User Credential Validation** integrated with User Management Service
3. **Token Blacklisting** for secure logout and token revocation
4. **Security Configuration** with comprehensive Spring Security setup
5. **Brute Force Protection** with account locking and rate limiting
6. **Service Integration** with User Management Service via Feign client
7. **Authentication Audit** with comprehensive logging and monitoring
8. **API Documentation** with complete endpoint and security guides
9. **Testing Suite** covering authentication flows and security scenarios
10. **Error Handling** with detailed error responses and user guidance

## Authentication Success Criteria

- ✅ JWT token generation and validation working correctly
- ✅ User authentication with credential validation functional
- ✅ Token refresh and rotation mechanism operational
- ✅ Token blacklisting and revocation system working
- ✅ Brute force protection and account locking active
- ✅ CORS and security configuration properly implemented
- ✅ User Service integration with fallback mechanisms functional
- ✅ Rate limiting and DDoS protection configured
- ✅ Authentication audit logging and monitoring operational
- ✅ Comprehensive error handling and user feedback working
- ✅ API documentation complete and accessible
- ✅ Testing coverage comprehensive for all authentication scenarios
- ✅ Performance optimization and caching implemented
- ✅ Production-ready with proper security hardening
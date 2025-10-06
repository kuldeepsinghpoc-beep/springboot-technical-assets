# Authentication Service Requirements

## Project Overview

Develop a comprehensive Spring Boot authentication service focused on JWT-based authentication, user credential validation, and secure session management. This service handles login, logout, token generation, token validation, and authentication state management.

**Service Details:**
- **Authentication Service Port**: 8082
- **Swagger UI URL**: `http://localhost:8082/swagger-ui.html`
- **API Docs URL**: `http://localhost:8082/api-docs`
- **User Management Service**: `http://localhost:8080/api/v1` (integrated)

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

**Public Authentication Endpoints:**

- **POST** `/api/auth/login` - User authentication with credentials
  - **Request Body**: `LoginRequest`
  ```json
  {
    "username": "string (3-100 chars, required)",
    "password": "string (6-100 chars, required)", 
    "rememberMe": "boolean (optional, default: false)",
    "deviceInfo": "string (optional)"
  }
  ```
  - **Response**: `AuthenticationResponse`
  ```json
  {
    "accessToken": "string",
    "refreshToken": "string", 
    "tokenType": "Bearer",
    "accessTokenExpiry": "long (timestamp)",
    "refreshTokenExpiry": "long (timestamp)",
    "userId": "long",
    "username": "string",
    "authenticationTimestamp": "datetime"
  }
  ```

- **POST** `/api/auth/refresh` - Access token refresh using refresh token
  - **Request Body**: `RefreshTokenRequest`
  ```json
  {
    "refreshToken": "string (required)"
  }
  ```
  - **Response**: `AuthenticationResponse`

- **POST** `/api/auth/validate` - Token validation for other services  
  - **Request Body**: `TokenValidationRequest`
  ```json
  {
    "accessToken": "string (required)"
  }
  ```
  - **Response**: `TokenValidationResponse`
  ```json
  {
    "valid": "boolean",
    "userId": "long",
    "username": "string", 
    "expired": "boolean",
    "tokenType": "string"
  }
  ```

**Protected Authentication Endpoints (Require Bearer Token):**

- **POST** `/api/auth/logout` - User logout with token invalidation
  - **Headers**: `Authorization: Bearer <token>` OR
  - **Request Body**: `LogoutRequest` (optional)
  ```json
  {
    "accessToken": "string (optional if header provided)"
  }
  ```
  - **Response**:
  ```json
  {
    "message": "Logout successful"
  }
  ```

- **GET** `/api/auth/me` - Current authenticated user information
  - **Headers**: `Authorization: Bearer <token>` (required)
  - **Response**:
  ```json
  {
    "userId": "long",
    "username": "string",
    "claims": "object",
    "authenticated": "boolean"
  }
  ```

- **POST** `/api/auth/revoke` - Token revocation (administrative)
  - **Query Parameters**: 
    - `userId` (Long, required)
    - `reason` (String, optional, default: "ADMIN_REVOCATION")
  - **Response**:
  ```json
  {
    "message": "All tokens revoked for user: {userId}",
    "reason": "string", 
    "revokedBy": "string"
  }
  ```

#### B. AuthInternalController.java

**Internal Service-to-Service Endpoints:**

- **POST** `/internal/auth/validate-token` - Token validation for microservices
  - **Request Body**:
  ```json
  {
    "token": "string (required)"
  }
  ```
  - **Response**: `TokenValidationResponse`

- **POST** `/internal/auth/revoke-user-tokens` - Revoke all tokens for specific user
  - **Request Body**:
  ```json
  {
    "userId": "long (required)",
    "reason": "string (optional, default: 'INTERNAL_SERVICE_REVOCATION')",
    "revokedBy": "string (optional, default: 'INTERNAL_SERVICE')"
  }
  ```
  - **Response**:
  ```json
  {
    "message": "All tokens revoked for user: {userId}",
    "reason": "string",
    "revokedBy": "string"
  }
  ```

- **GET** `/internal/auth/token-info` - Token metadata and claims information
  - **Query Parameters**: `token` (String, required)
  - **Response**:
  ```json
  {
    "valid": "boolean",
    "expired": "boolean", 
    "username": "string",
    "userId": "long",
    "tokenType": "string",
    "claims": "object"
  }
  ```

- **GET** `/internal/auth/user-service-health` - Check user service health
  - **Response**:
  ```json
  {
    "status": "UP|DOWN", 
    "service": "user-service",
    "message": "string"
  }
  ```

- **GET** `/internal/auth/test-user-service` - Test user service connectivity
  - **Query Parameters**: 
    - `username` (String, optional, default: "testuser")
    - `password` (String, optional, default: "testpass")
  - **Response**:
  ```json
  {
    "status": "SUCCESS|ERROR",
    "message": "string",
    "credentialsValid": "boolean",
    "user": "object|null"
  }
  ```

### 2. User Management Service Integration

#### A. UserServiceClient.java (Feign Client)

**User Management Service API Calls** (via `http://localhost:8080/api/v1`):

- **GET** `/internal/users/validate` - Validate user credentials
  - **Query Parameters**: 
    - `username` (String, required)
    - `password` (String, required)  
  - **Returns**: `Boolean`

- **GET** `/users/{id}` - Get user by ID
  - **Path Variable**: `id` (Long, required)
  - **Returns**: `UserDto`
  ```json
  {
    "id": "long",
    "username": "string", 
    "email": "string",
    "firstName": "string",
    "lastName": "string",
    "active": "boolean",
    "enabled": "boolean"
  }
  ```

- **GET** `/internal/users/by-username/{username}` - Get user by username
  - **Path Variable**: `username` (String, required)  
  - **Returns**: `UserDto`

#### B. UserServiceFallback.java

Fallback implementation providing:
- Cached user data utilization when service is unavailable
- Graceful degradation of authentication features
- Default responses for service unavailability scenarios

### 3. JWT Token Management

#### A. JwtUtil.java

Comprehensive JWT utility service for token operations

**Token Generation:**
- Access token generation with user claims and authorities
- Refresh token generation with extended expiry (24 hours)
- Custom claims addition (user ID, username, roles, permissions)
- Token signing with secure secret key
- Configurable token expiration times (access: 15 min, refresh: 24 hours)

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
- Blacklist storage optimization (H2/Redis)
- Token revocation audit logging

### 4. Security Configuration

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
- Public endpoint configuration (login, health checks, swagger)
- Protected endpoint authorization rules
- CORS configuration for cross-origin requests
- CSRF protection configuration (disabled for stateless JWT)

**Session Management:**
- Stateless session configuration
- Session creation policy (STATELESS)
- Concurrent session control (if needed)

#### B. JwtAuthenticationFilter.java

- Custom authentication filter for JWT processing
- Authorization header extraction and validation
- Token validation and user authentication setup
- Security context population with authenticated user
- Error handling for invalid or expired tokens
- Request/response logging for authentication audit

### 5. Authentication Service Layer

#### A. AuthenticationService.java

**User Authentication:**
- Username/password validation via UserServiceClient
- User account status verification (active, enabled, not locked)
- Failed login attempt tracking and account locking
- Authentication success/failure logging and metrics

**Token Management:**
- Access and refresh token generation
- Token validation and verification
- Token refresh logic with security checks
- Token revocation and blacklisting
- Token cleanup and maintenance

**User Service Integration:**
- User credential validation via User Management Service (`http://localhost:8080/api/v1`)
- User profile information retrieval
- User account status synchronization

#### B. UserDetailsServiceImpl.java

- Custom UserDetailsService implementation
- User loading from User Management Service (`http://localhost:8080/api/v1`)
- UserDetails object creation with authorities
- User account status mapping
- Caching for performance optimization

### 6. Authentication DTOs

#### A. Authentication Request DTOs

- **LoginRequest.java** - User login credentials
  - Username or email (required, validated, 3-100 characters)
  - Password (required, validated, 6-100 characters)
  - Remember me option (optional, default: false)
  - Device information for tracking (optional)
  
- **RefreshTokenRequest.java** - Token refresh request
  - Refresh token (required, validated)
  
- **TokenValidationRequest.java** - Token validation request
  - Access token (required)
  
- **LogoutRequest.java** - User logout request
  - Access token for revocation (optional if header provided)

#### B. Authentication Response DTOs

- **AuthenticationResponse.java** - Successful authentication response
  - Access token with expiration (15 minutes)
  - Refresh token with expiration (24 hours) 
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

## Authentication Configuration

### A. JWT Configuration Properties

```properties
# Server Configuration  
server.port=8082

# JWT Configuration
app.jwt.secret=${JWT_SECRET:mySecretKey12345678901234567890123456789012345678901234567890}
app.jwt.access-token-expiry=900000     # 15 minutes
app.jwt.refresh-token-expiry=86400000  # 24 hours
app.jwt.issuer=authentication-service
app.jwt.audience=wipro-ai-services

# Token Configuration
app.auth.max-failed-attempts=5
app.auth.account-lock-duration=900000     # 15 minutes
app.auth.token-cleanup-interval=3600000   # 1 hour
app.auth.remember-me-expiry=2592000000    # 30 days

# User Service Integration  
app.user-service.base-url=http://localhost:8080/api/v1
app.user-service.timeout=5000
app.user-service.retry-attempts=3
app.user-service.circuit-breaker-enabled=true

# Security Configuration
app.security.cors-allowed-origins=http://localhost:3000,http://localhost:4200
app.security.cors-allowed-methods=GET,POST,PUT,DELETE,OPTIONS
app.security.cors-allowed-headers=*
app.security.rate-limit-requests=100
app.security.rate-limit-window=3600

# OpenAPI Documentation
springdoc.api-docs.path=/api-docs
springdoc.swagger-ui.path=/swagger-ui.html
springdoc.swagger-ui.operationsSorter=method
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
- **Authentication Service**: `http://localhost:8082`
- **Swagger UI**: `http://localhost:8082/swagger-ui.html`
- **Content-Type**: `application/json`
- **Authentication**: Bearer token (except login and public endpoints)

### API Testing Examples

#### 1. User Login
```bash
curl -X POST http://localhost:8082/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "testpass123", 
    "rememberMe": false,
    "deviceInfo": "Browser Chrome"
  }'
```

#### 2. Get Current User Info
```bash
curl -X GET http://localhost:8082/api/auth/me \
  -H "Authorization: Bearer <access_token>"
```

#### 3. Refresh Token
```bash
curl -X POST http://localhost:8082/api/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refreshToken": "<refresh_token>"
  }'
```

#### 4. Validate Token
```bash
curl -X POST http://localhost:8082/api/auth/validate \
  -H "Content-Type: application/json" \
  -d '{
    "accessToken": "<access_token>"
  }'
```

#### 5. User Logout
```bash
curl -X POST http://localhost:8082/api/auth/logout \
  -H "Authorization: Bearer <access_token>"
```

#### 6. Admin Token Revocation
```bash
curl -X POST "http://localhost:8082/api/auth/revoke?userId=123&reason=ADMIN_ACTION" \
  -H "Authorization: Bearer <admin_token>"
```

#### Internal Service Endpoints

#### 7. Internal Token Validation
```bash  
curl -X POST http://localhost:8082/internal/auth/validate-token \
  -H "Content-Type: application/json" \
  -d '{
    "token": "<token_to_validate>"
  }'
```

#### 8. Internal Token Info
```bash
curl -X GET "http://localhost:8082/internal/auth/token-info?token=<token>" \
  -H "Content-Type: application/json"
```

#### 9. Check User Service Health
```bash
curl -X GET http://localhost:8082/internal/auth/user-service-health
```

#### 10. Test User Service Connection  
```bash
curl -X GET "http://localhost:8082/internal/auth/test-user-service?username=testuser&password=testpass"
```

### User Management Service Integration

The authentication service integrates with the User Management Service at `http://localhost:8080/api/v1` through the following calls:

#### UserServiceClient API Calls

1. **Validate Credentials**
   ```java
   GET /internal/users/validate?username={username}&password={password}
   Returns: Boolean
   ```

2. **Get User by ID**  
   ```java
   GET /users/{id}
   Returns: UserDto
   ```

3. **Get User by Username**
   ```java  
   GET /internal/users/by-username/{username}
   Returns: UserDto
   ```

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
  "timestamp": "2025-10-06T10:15:30.123Z",
  "status": 401,
  "error": "Unauthorized",
  "message": "Invalid credentials provided",
  "path": "/api/auth/login",
  "details": {
    "errorCode": "INVALID_CREDENTIALS",
    "attemptCount": 3,
    "lockoutTime": "2025-10-06T10:30:30.123Z",
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

- Complete authentication endpoint documentation with Swagger UI at `http://localhost:8082/swagger-ui.html`
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
2. **User Credential Validation** integrated with User Management Service (`http://localhost:8080/api/v1`)
3. **Token Blacklisting** for secure logout and token revocation
4. **Security Configuration** with comprehensive Spring Security setup
5. **Brute Force Protection** with account locking and rate limiting
6. **Service Integration** with User Management Service via Feign client
7. **Authentication Audit** with comprehensive logging and monitoring
8. **API Documentation** with Swagger UI and complete endpoint guides
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
- ✅ API documentation complete and accessible via Swagger UI
- ✅ Testing coverage comprehensive for all authentication scenarios
- ✅ Performance optimization and caching implemented
- ✅ Production-ready with proper security hardening

## Service URLs Quick Reference

| Service | URL | Description |
|---------|-----|-------------|
| **Authentication Service** | `http://localhost:8082` | Main authentication service |
| **Swagger UI** | `http://localhost:8082/swagger-ui.html` | Interactive API documentation |
| **API Documentation** | `http://localhost:8082/api-docs` | OpenAPI specification |
| **User Management Service** | `http://localhost:8080/api/v1` | Integrated user service |
| **H2 Console** | `http://localhost:8082/h2-console` | Database console (dev only) |
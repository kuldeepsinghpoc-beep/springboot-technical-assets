# User Management Service Requirements

## Project Overview

Create a comprehensive Spring Boot user management service focused on user registration, profile management, and user lifecycle operations. This service handles all user-related data operations while integrating with authentication services for complete user experience management.

## User Management Dependencies

```xml
<!-- User Data & Validation -->
- spring-boot-starter-data-jpa (User entity management)
- spring-boot-starter-validation (Input validation)

<!-- Password Security -->
- spring-security-crypto (Password hashing, independent of Spring Security)

<!-- Communication -->
- spring-boot-starter-web (REST API endpoints)
- spring-cloud-starter-openfeign (Service communication)

<!-- Optional: File Upload -->
- spring-boot-starter-web (Multipart support for profile images)
```

## Core User Management Components

### 1. User Management Controller

#### A. UserController.java

- POST `/api/users/register` - New user registration with validation
- GET `/api/users/profile/{id}` - User profile retrieval
- PUT `/api/users/profile/{id}` - User profile update
- DELETE `/api/users/{id}` - User account deactivation
- GET `/api/users` - User listing with pagination and filtering
- PUT `/api/users/{id}/activate` - User account activation
- PUT `/api/users/{id}/deactivate` - User account deactivation
- POST `/api/users/{id}/password-reset` - Password reset initiation
- PUT `/api/users/{id}/password` - Password change operation
- Complete OpenAPI/Swagger documentation for all user endpoints
- Comprehensive error handling with user-friendly messages

#### B. InternalUserController.java

- Internal endpoints for service-to-service communication
- GET `/internal/users/by-username/{username}` - User lookup for authentication
- GET `/internal/users/by-email/{email}` - User lookup for authentication
- PUT `/internal/users/{id}/last-login` - Last login timestamp update
- GET `/internal/users/{id}/credentials` - Password verification support
- Service authentication and authorization

### 2. User Entity Model

#### A. User.java

JPA entity with comprehensive field mapping and relationships

**Identity Fields:**
- id (Long, auto-generated primary key)
- username (String, unique, 3-50 characters)
- email (String, unique, valid email format)

**Authentication Fields:**
- password (String, BCrypt hashed, excluded from JSON)
- passwordResetToken (String, nullable)
- passwordResetExpiry (LocalDateTime, nullable)

**Profile Fields:**
- firstName (String, required, 1-50 characters)
- lastName (String, required, 1-50 characters)
- phoneNumber (String, optional, max 15 characters)
- dateOfBirth (LocalDate, optional)
- profileImageUrl (String, optional)

**Status Fields:**
- active (Boolean, default true)
- enabled (Boolean, default true)
- accountNonExpired (Boolean, default true)
- credentialsNonExpired (Boolean, default true)
- emailVerified (Boolean, default false)

**Audit Fields:**
- createdAt (LocalDateTime, auto-generated)
- updatedAt (LocalDateTime, auto-updated)
- lastLogin (LocalDateTime, nullable)
- createdBy (String, optional for audit)
- updatedBy (String, optional for audit)

Additional Features:
- Unique constraints on username and email with proper indexing
- Audit fields with @PrePersist and @PreUpdate annotations
- JSON serialization configuration (exclude password)

#### B. UserRole.java (Optional)

- Role-based access control entity (if RBAC needed)
- Many-to-many relationship with User entity
- Role hierarchy and permission mapping
- Dynamic role assignment capabilities

### 3. User Repository Layer

#### A. UserRepository.java

Extended JpaRepository with custom query methods

**User Lookup Methods:**
- `findByUsername(String username)`
- `findByEmail(String email)`
- `findByUsernameOrEmail(String username, String email)`
- `findByUsernameIgnoreCase(String username)`
- `findByEmailIgnoreCase(String email)`

**Existence Check Methods:**
- `existsByUsername(String username)`
- `existsByEmail(String email)`
- `existsByUsernameIgnoreCase(String username)`
- `existsByEmailIgnoreCase(String email)`

**Status-Based Queries:**
- `findByActiveTrue()`
- `findByActiveFalse()`
- `findByEnabledTrue()`
- `findByEmailVerifiedFalse()`

**Custom Query Methods:**
- @Query annotations for complex searches
- Paginated user listing with sorting
- User search by multiple criteria
- Active user count and statistics

**Password Reset Methods:**
- `findByPasswordResetToken(String token)`
- `findByPasswordResetTokenAndPasswordResetExpiryAfter(String token, LocalDateTime now)`

### 4. User Data Transfer Objects

#### A. User Request DTOs

- **UserRegistrationRequest.java** - New user registration
  - All required fields with comprehensive validation
  - Password confirmation field
  - Terms and conditions acceptance
  - Custom validation annotations
- **UserUpdateRequest.java** - User profile updates
  - Partial update support
  - Validation for updatable fields only
  - Version control for optimistic locking
- **PasswordChangeRequest.java** - Password modification
  - Current password verification
  - New password with confirmation
  - Password strength validation
- **PasswordResetRequest.java** - Password reset initiation
  - Email or username identification
  - Security question integration (optional)

#### B. User Response DTOs

- **UserResponse.java** - Complete user profile response
  - All user fields except sensitive data (password)
  - Role and permission information
  - Account status and verification flags
- **UserSummaryResponse.java** - Basic user information
  - Essential fields for listings and references
  - Optimized for performance in bulk operations
- **UserRegistrationResponse.java** - Registration confirmation
  - Registration success confirmation
  - Next steps and verification instructions
  - Basic user information
- **UserCredentialResponse.java** - Internal service response
  - Password hash for authentication service
  - Account status for validation
  - Restricted to internal service calls

### 5. User Service Layer

#### A. UserService.java

**User Registration Operations:**
- Username and email uniqueness validation
- Password encryption with BCrypt
- User account creation with default settings
- Email verification initiation (optional)
- Registration notification and confirmation

**User Profile Management:**
- Profile information retrieval and formatting
- Profile update with validation and conflict resolution
- Profile image upload and management
- Sensitive data filtering for responses

**User Account Management:**
- Account activation and deactivation
- Account status updates and notifications
- Account deletion with data retention policies
- User role assignment and management (if RBAC)

**Password Management:**
- Password change with current password verification
- Password reset token generation and validation
- Password history tracking (optional)
- Password strength validation and enforcement

**User Search and Listing:**
- Paginated user listing with sorting options
- User search by multiple criteria
- User statistics and reporting
- Active user monitoring and analytics

#### B. UserValidationService.java

- Centralized validation logic for user operations
- Username and email format validation
- Password strength and complexity validation
- Business rule validation and enforcement
- Custom validation annotations and validators
- Cross-field validation for registration and updates

#### C. NotificationService.java

User notification management (optional enhancement):
- Registration welcome emails
- Password reset notifications
- Account status change notifications
- Profile update confirmations

## User Management Configuration

### A. User Validation Properties

```properties
# User Registration Configuration
user.registration.enabled=true
user.registration.email-verification-required=true
user.registration.auto-activation=true

# Username Validation
user.username.min-length=3
user.username.max-length=50
user.username.pattern=^[a-zA-Z0-9_]+$
user.username.case-sensitive=false

# Email Validation
user.email.max-length=100
user.email.case-sensitive=false
user.email.domain-whitelist=
user.email.domain-blacklist=

# Password Configuration
user.password.min-length=6
user.password.max-length=100
user.password.require-uppercase=false
user.password.require-lowercase=false
user.password.require-numbers=false
user.password.require-special-chars=false
user.password.bcrypt-strength=12

# Profile Configuration
user.profile.image-max-size=5242880
user.profile.image-allowed-types=jpg,jpeg,png,gif
user.profile.phone-pattern=^[+]?[0-9\s\-()]+$

# Account Management
user.account.default-active=true
user.account.default-enabled=true
user.account.password-reset-expiry=3600000
```

### B. User Validation Rules

#### Registration Validation

- **Username**: 3-50 characters, unique, alphanumeric with underscores, case-insensitive check
- **Email**: Valid email format, unique, max 100 characters, case-insensitive check
- **Password**: 6-100 characters, BCrypt encrypted, confirmation required
- **First Name**: Required, 1-50 characters, letters and spaces only
- **Last Name**: Required, 1-50 characters, letters and spaces only
- **Phone Number**: Optional, max 15 characters, valid phone format
- **Date of Birth**: Optional, valid date, age restrictions if applicable
- **Terms Acceptance**: Required boolean for legal compliance

#### Profile Update Validation

- **Partial Updates**: Only provided fields are validated and updated
- **Username Change**: Restricted or with additional verification
- **Email Change**: Requires re-verification of new email
- **Profile Image**: File type, size, and dimension validation
- **Version Control**: Optimistic locking to prevent concurrent updates

## User Management API Endpoints

### Endpoint Specifications

- **Base URL**: `/api/users`
- **Internal Base URL**: `/internal/users`
- **Content-Type**: `application/json`
- **Authentication**: Required for most endpoints (via Authentication Service)

#### Public Endpoints

- `POST /api/users/register` - User registration
  - Request: complete registration data with validation
  - Response: registration confirmation and user summary
  - Error handling: duplicate username/email, validation errors
- `POST /api/users/password-reset/request` - Password reset request
  - Request: email or username for password reset
  - Response: confirmation (no sensitive information leaked)
  - Rate limiting and security measures
- `POST /api/users/password-reset/confirm` - Password reset confirmation
  - Request: reset token and new password
  - Response: confirmation of password update
  - Token validation and expiry checking

#### Protected Endpoints (Authentication Required)

- `GET /api/users/profile` - Current user profile
- `GET /api/users/profile/{id}` - Specific user profile (admin/self only)
- `PUT /api/users/profile/{id}` - Update user profile
- `PUT /api/users/{id}/password` - Change password
- `PUT /api/users/{id}/activate` - Activate user account (admin only)
- `PUT /api/users/{id}/deactivate` - Deactivate user account
- `DELETE /api/users/{id}` - Delete user account
- `GET /api/users` - List users with pagination and filtering (admin only)
- `POST /api/users/{id}/profile-image` - Upload profile image

#### Internal Service Endpoints

- `GET /internal/users/by-username/{username}` - User lookup for authentication
- `GET /internal/users/by-email/{email}` - User lookup for authentication
- `GET /internal/users/{id}/credentials` - Password hash for verification
- `PUT /internal/users/{id}/last-login` - Update last login timestamp
- `GET /internal/users/{id}/status` - Account status for validation
- Service-to-service authentication and authorization required

## User Management Testing Requirements

### A. User Operation Tests

- User registration with validation testing
- Duplicate username and email handling
- Profile update and partial update scenarios
- Password change and reset workflows
- Account activation and deactivation testing
- User search and pagination functionality

### B. Validation Tests

- Input validation for all user fields
- Custom validation annotation testing
- Cross-field validation scenarios
- Password strength and complexity validation
- Email and username format validation
- File upload validation (profile images)

### C. Repository Tests

- Custom query method testing
- User lookup and existence check methods
- Pagination and sorting functionality
- Database constraint testing (unique constraints)
- Audit field automatic population

## User Management Error Handling

### A. User Management Exceptions

- **UserNotFoundException**: User does not exist
- **DuplicateUsernameException**: Username already exists
- **DuplicateEmailException**: Email already exists
- **InvalidPasswordException**: Password does not meet requirements
- **UserRegistrationException**: General registration errors
- **ProfileUpdateException**: Profile update conflicts or errors
- **PasswordResetException**: Password reset token issues
- **AccountStatusException**: Account status related errors
- **UserValidationException**: Input validation failures

### B. Error Response Format

```json
{
  "timestamp": "2025-10-05T10:15:30.123Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Username already exists",
  "path": "/api/users/register",
  "details": {
    "field": "username",
    "code": "DUPLICATE_USERNAME",
    "rejectedValue": "johndoe",
    "suggestions": ["johndoe123", "johndoe_1"]
  },
  "validationErrors": [
    {
      "field": "password",
      "message": "Password must be at least 8 characters long",
      "code": "PASSWORD_TOO_SHORT"
    }
  ]
}
```

## User Management Service File Structure

```
user-management-service/
├── src/
│   ├── main/
│   │   ├── java/com/wipro/ai/demo/user/
│   │   │   ├── controller/
│   │   │   │   ├── UserController.java
│   │   │   │   └── InternalUserController.java
│   │   │   ├── dto/
│   │   │   │   ├── request/
│   │   │   │   │   ├── UserRegistrationRequest.java
│   │   │   │   │   ├── UserUpdateRequest.java
│   │   │   │   │   ├── PasswordChangeRequest.java
│   │   │   │   │   └── PasswordResetRequest.java
│   │   │   │   └── response/
│   │   │   │       ├── UserResponse.java
│   │   │   │       ├── UserSummaryResponse.java
│   │   │   │       ├── UserRegistrationResponse.java
│   │   │   │       └── UserCredentialResponse.java
│   │   │   ├── entity/
│   │   │   │   ├── User.java
│   │   │   │   └── UserRole.java (optional)
│   │   │   ├── exception/
│   │   │   │   ├── UserNotFoundException.java
│   │   │   │   ├── DuplicateUsernameException.java
│   │   │   │   ├── DuplicateEmailException.java
│   │   │   │   ├── InvalidPasswordException.java
│   │   │   │   ├── UserRegistrationException.java
│   │   │   │   ├── ProfileUpdateException.java
│   │   │   │   ├── PasswordResetException.java
│   │   │   │   └── UserValidationException.java
│   │   │   ├── repository/
│   │   │   │   └── UserRepository.java
│   │   │   ├── service/
│   │   │   │   ├── UserService.java
│   │   │   │   ├── UserValidationService.java
│   │   │   │   └── NotificationService.java
│   │   │   ├── validation/
│   │   │   │   ├── UniqueUsername.java
│   │   │   │   ├── UniqueEmail.java
│   │   │   │   ├── ValidPassword.java
│   │   │   │   └── PasswordMatch.java
│   │   │   └── config/
│   │   │       ├── UserValidationConfig.java
│   │   │       └── FileUploadConfig.java
│   │   └── resources/
│   │       ├── application.properties
│   │       └── application-user.properties
│   └── test/
│       ├── java/com/wipro/ai/demo/user/
│       │   ├── controller/
│       │   │   ├── UserControllerTest.java
│       │   │   └── InternalUserControllerTest.java
│       │   ├── repository/
│       │   │   └── UserRepositoryTest.java
│       │   ├── service/
│       │   │   ├── UserServiceTest.java
│       │   │   └── UserValidationServiceTest.java
│       │   └── validation/
│       │       └── UserValidationTest.java
│       └── resources/
│           └── application-test.properties
├── postman/
│   ├── user-api-collection.json
│   └── user-environment.json
├── scripts/
│   ├── test-user-api.bat
│   └── test-user-api.ps1
└── docs/
    ├── USER-MANAGEMENT-API-GUIDE.md
    └── USER-VALIDATION-GUIDE.md
```

## Integration with Authentication Service

### A. Service Integration

- Internal API endpoints for authentication service consumption
- User credential verification support
- Account status validation for authentication
- Last login timestamp coordination
- User profile data provision for JWT claims
- Service-to-service authentication and authorization

### B. Event-Driven Communication (Optional)

- User registration events for authentication service
- Account status change notifications
- Password change events for token invalidation
- User deletion events for cleanup

## User Management Documentation

### A. API Documentation

- Complete user management endpoint documentation
- Registration and profile management workflows
- Password management and reset procedures
- User search and administration capabilities
- Internal service integration examples
- Error handling and validation guidelines

### B. Validation Documentation

- User input validation rules and patterns
- Custom validation annotation usage
- Password policy and strength requirements
- File upload restrictions and guidelines
- Cross-field validation scenarios

## Expected User Management Deliverables

1. **User Registration System** with comprehensive validation
2. **Profile Management** with update and image upload capabilities
3. **Password Management** including change and reset functionality
4. **User Repository** with optimized queries and indexing
5. **Account Management** with activation/deactivation features
6. **User Search and Listing** with pagination and filtering
7. **Service Integration** with Authentication Service
8. **Validation Framework** with custom annotations
9. **Testing Suite** covering all user operations
10. **Documentation** for user management APIs and workflows

## User Management Success Criteria

- ✅ User registration with comprehensive validation working
- ✅ Profile management with secure updates operational
- ✅ Password management and reset functionality complete
- ✅ User search and administration features functional
- ✅ Account lifecycle management implemented
- ✅ Database optimization and indexing configured
- ✅ Service integration with Authentication Service working
- ✅ Input validation and error handling comprehensive
- ✅ File upload and profile image management operational
- ✅ Testing coverage complete for all user operations
- ✅ User management documentation comprehensive
- ✅ Production-ready with proper security measures
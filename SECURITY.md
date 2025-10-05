# Security Recommendations

This document outlines security improvements that should be considered for the ExpensePilot backend application.

## 🔴 High Priority

### 1. Move Secrets to Environment Variables

**Current Issue:**
- JWT secret is hardcoded in `SecurityConstants.java`
- Database credentials are hardcoded in `application.properties`

**Recommendation:**
Update `application.properties` to use environment variables:
```properties
# JWT Configuration
jwt.secret=${JWT_SECRET:default-secret-for-development-only}
jwt.expiration=${JWT_EXPIRATION:86400000}

# Database Configuration
spring.datasource.url=${DB_URL:jdbc:h2:mem:expensepilot}
spring.datasource.username=${DB_USERNAME:sa}
spring.datasource.password=${DB_PASSWORD:}
```

Update `SecurityConstants.java`:
```java
@Value("${jwt.secret}")
private String jwtSecretString;

@Value("${jwt.expiration}")
public static final long JWT_EXPIRATION;

@PostConstruct
public void init() {
    JWT_SECRET = Keys.hmacShaKeyFor(jwtSecretString.getBytes());
}
```

### 2. Use Strong, Unique Secrets in Production

**Current Issue:**
- The default JWT secret is a placeholder string
- Database password is weak (123456789)

**Recommendation:**
- Generate a cryptographically secure random secret for JWT (at least 64 characters)
- Use strong database passwords (minimum 16 characters with mixed case, numbers, and symbols)
- Never commit production secrets to version control

Example to generate a secure secret:
```bash
openssl rand -base64 64
```

## 🟡 Medium Priority

### 3. Add Application Properties Template

**Recommendation:**
Create `application.properties.template` with placeholder values:
```properties
spring.application.name=ExpensePilot
server.port=${PORT:8080}

# Database Configuration - Update these values
spring.datasource.url=${DB_URL:jdbc:mysql://localhost:3306/expensepilot}
spring.datasource.username=${DB_USERNAME:your-username}
spring.datasource.password=${DB_PASSWORD:your-password}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JWT Configuration - Update these values
jwt.secret=${JWT_SECRET:generate-a-secure-secret-here}
jwt.expiration=${JWT_EXPIRATION:86400000}

# JPA Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=false
spring.jpa.defer-datasource-initialization=true
```

Add `application.properties` to `.gitignore` if it contains sensitive data.

### 4. Enable HTTPS in Production

**Recommendation:**
Configure SSL/TLS for production deployments:
```properties
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=${KEYSTORE_PASSWORD}
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=expensepilot
```

### 5. Implement Rate Limiting

**Recommendation:**
Add rate limiting to prevent brute force attacks on authentication endpoints.

### 6. Add Security Headers

**Recommendation:**
Configure security headers in `SecurityConfig.java`:
```java
http.headers()
    .contentSecurityPolicy("default-src 'self'")
    .and()
    .frameOptions().deny()
    .and()
    .xssProtection().block(true)
    .and()
    .httpStrictTransportSecurity()
        .maxAgeInSeconds(31536000)
        .includeSubDomains(true);
```

## 🟢 Low Priority

### 7. Implement Password Complexity Requirements

**Recommendation:**
Add password validation to enforce:
- Minimum length (currently 6, recommend 8+)
- Mix of uppercase, lowercase, numbers, and special characters

### 8. Add Account Lockout After Failed Login Attempts

**Recommendation:**
Implement account lockout mechanism after N failed login attempts.

### 9. Audit Logging

**Recommendation:**
Log security-relevant events (login attempts, password changes, etc.)

### 10. Regular Dependency Updates

**Recommendation:**
- Regularly update dependencies to patch security vulnerabilities
- Use tools like OWASP Dependency Check or Snyk

## Implementation Priority

1. **Before Production Deployment:**
   - Move all secrets to environment variables (#1)
   - Use strong, unique secrets (#2)
   - Enable HTTPS (#4)

2. **Short Term:**
   - Add rate limiting (#5)
   - Add security headers (#6)
   - Create properties template (#3)

3. **Long Term:**
   - Implement password complexity (#7)
   - Add account lockout (#8)
   - Set up audit logging (#9)
   - Establish dependency update process (#10)

---

**Note:** This is a living document. Update it as security improvements are implemented.

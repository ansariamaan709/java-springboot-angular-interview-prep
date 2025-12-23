# Java Security & Secure Coding — Complete Interview Guide

## Table of Contents

1. [Security Fundamentals](#security-fundamentals)
2. [Authentication & Authorization](#authentication--authorization)
3. [Cryptography in Java](#cryptography-in-java)
4. [Secure Password Storage](#secure-password-storage)
5. [Input Validation & Sanitization](#input-validation--sanitization)
6. [SQL Injection Prevention](#sql-injection-prevention)
7. [XSS Prevention](#xss-prevention)
8. [CSRF Protection](#csrf-protection)
9. [Secure Session Management](#secure-session-management)
10. [JWT Security](#jwt-security)
11. [OWASP Top 10 for Java](#owasp-top-10-for-java)
12. [Secure Coding Best Practices](#secure-coding-best-practices)
13. [Security Testing](#security-testing)
14. [Interview Questions](#interview-questions)

---

## Security Fundamentals

### CIA Triad

- **Confidentiality**: Data accessible only to authorized parties
- **Integrity**: Data not modified by unauthorized parties
- **Availability**: System accessible when needed

### Defense in Depth

Multiple layers of security controls:

```
┌─────────────────────────────────────────────────────────────────┐
│                         Network Security                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Firewall / WAF                        │   │
│   │   ┌─────────────────────────────────────────────────┐   │   │
│   │   │               Application Security               │   │   │
│   │   │   ┌─────────────────────────────────────────┐   │   │   │
│   │   │   │            Input Validation              │   │   │   │
│   │   │   │   ┌─────────────────────────────────┐   │   │   │   │
│   │   │   │   │         Data Security            │   │   │   │   │
│   │   │   │   │   (Encryption, Access Control)   │   │   │   │   │
│   │   │   │   └─────────────────────────────────┘   │   │   │   │
│   │   │   └─────────────────────────────────────────┘   │   │   │
│   │   └─────────────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Principle of Least Privilege

Grant minimum permissions necessary for a task.

```java
// Bad: Overly permissive
public class AdminController {
    @PreAuthorize("isAuthenticated()")  // Any logged-in user can access
    public void deleteAllUsers() { ... }
}

// Good: Specific permission
public class AdminController {
    @PreAuthorize("hasRole('SUPER_ADMIN') and hasAuthority('DELETE_ALL_USERS')")
    public void deleteAllUsers() { ... }
}
```

---

## Authentication & Authorization

### Authentication (AuthN)

**Who are you?** — Verify identity.

### Authorization (AuthZ)

**What can you do?** — Verify permissions.

### Spring Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()))
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasAuthority("DELETE")
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(Customizer.withDefaults()));

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);  // Cost factor 12
    }
}
```

### Method-Level Security

```java
@Service
public class DocumentService {

    // Role-based
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteDocument(Long id) { ... }

    // Permission-based
    @PreAuthorize("hasAuthority('DOCUMENT_WRITE')")
    public Document createDocument(Document doc) { ... }

    // Expression-based with method parameters
    @PreAuthorize("#document.ownerId == authentication.principal.id or hasRole('ADMIN')")
    public void updateDocument(Document document) { ... }

    // Post-authorization (filter result)
    @PostAuthorize("returnObject.ownerId == authentication.principal.id")
    public Document getDocument(Long id) { ... }

    // Filter collections
    @PostFilter("filterObject.public or filterObject.ownerId == authentication.principal.id")
    public List<Document> getDocuments() { ... }
}
```

---

## Cryptography in Java

### Symmetric Encryption (AES)

```java
import javax.crypto.*;
import javax.crypto.spec.*;
import java.security.*;
import java.util.Base64;

public class AESEncryption {

    private static final String ALGORITHM = "AES/GCM/NoPadding";
    private static final int GCM_IV_LENGTH = 12;
    private static final int GCM_TAG_LENGTH = 128;

    // Generate secure key
    public static SecretKey generateKey() throws NoSuchAlgorithmException {
        KeyGenerator keyGen = KeyGenerator.getInstance("AES");
        keyGen.init(256);  // 256-bit key
        return keyGen.generateKey();
    }

    // Encrypt
    public static String encrypt(String plaintext, SecretKey key) throws Exception {
        // Generate random IV
        byte[] iv = new byte[GCM_IV_LENGTH];
        SecureRandom random = new SecureRandom();
        random.nextBytes(iv);

        Cipher cipher = Cipher.getInstance(ALGORITHM);
        GCMParameterSpec gcmSpec = new GCMParameterSpec(GCM_TAG_LENGTH, iv);
        cipher.init(Cipher.ENCRYPT_MODE, key, gcmSpec);

        byte[] ciphertext = cipher.doFinal(plaintext.getBytes("UTF-8"));

        // Combine IV + ciphertext
        byte[] combined = new byte[iv.length + ciphertext.length];
        System.arraycopy(iv, 0, combined, 0, iv.length);
        System.arraycopy(ciphertext, 0, combined, iv.length, ciphertext.length);

        return Base64.getEncoder().encodeToString(combined);
    }

    // Decrypt
    public static String decrypt(String encrypted, SecretKey key) throws Exception {
        byte[] combined = Base64.getDecoder().decode(encrypted);

        // Extract IV
        byte[] iv = new byte[GCM_IV_LENGTH];
        System.arraycopy(combined, 0, iv, 0, iv.length);

        // Extract ciphertext
        byte[] ciphertext = new byte[combined.length - GCM_IV_LENGTH];
        System.arraycopy(combined, GCM_IV_LENGTH, ciphertext, 0, ciphertext.length);

        Cipher cipher = Cipher.getInstance(ALGORITHM);
        GCMParameterSpec gcmSpec = new GCMParameterSpec(GCM_TAG_LENGTH, iv);
        cipher.init(Cipher.DECRYPT_MODE, key, gcmSpec);

        byte[] plaintext = cipher.doFinal(ciphertext);
        return new String(plaintext, "UTF-8");
    }
}
```

### Asymmetric Encryption (RSA)

```java
import java.security.*;
import javax.crypto.Cipher;

public class RSAEncryption {

    // Generate key pair
    public static KeyPair generateKeyPair() throws NoSuchAlgorithmException {
        KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
        keyGen.initialize(2048);  // 2048-bit minimum
        return keyGen.generateKeyPair();
    }

    // Encrypt with public key
    public static byte[] encrypt(byte[] data, PublicKey publicKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        return cipher.doFinal(data);
    }

    // Decrypt with private key
    public static byte[] decrypt(byte[] data, PrivateKey privateKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA/ECB/OAEPWithSHA-256AndMGF1Padding");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        return cipher.doFinal(data);
    }
}
```

### Digital Signatures

```java
public class DigitalSignature {

    public static byte[] sign(byte[] data, PrivateKey privateKey) throws Exception {
        Signature signature = Signature.getInstance("SHA256withRSA");
        signature.initSign(privateKey);
        signature.update(data);
        return signature.sign();
    }

    public static boolean verify(byte[] data, byte[] signatureBytes,
                                 PublicKey publicKey) throws Exception {
        Signature signature = Signature.getInstance("SHA256withRSA");
        signature.initVerify(publicKey);
        signature.update(data);
        return signature.verify(signatureBytes);
    }
}
```

### Secure Hashing

```java
import java.security.MessageDigest;

public class SecureHashing {

    // Simple hash (NOT for passwords)
    public static String sha256(String input) throws Exception {
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hash = digest.digest(input.getBytes("UTF-8"));
        return bytesToHex(hash);
    }

    // HMAC for message authentication
    public static String hmacSha256(String data, SecretKey key) throws Exception {
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(key);
        byte[] hmac = mac.doFinal(data.getBytes("UTF-8"));
        return bytesToHex(hmac);
    }

    private static String bytesToHex(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (byte b : bytes) {
            sb.append(String.format("%02x", b));
        }
        return sb.toString();
    }
}
```

---

## Secure Password Storage

### Never Store Plaintext Passwords!

```java
// ❌ NEVER DO THIS
user.setPassword(plainPassword);  // Storing plaintext

// ❌ DON'T USE WEAK HASHING
String hash = DigestUtils.md5Hex(password);  // MD5 is broken
String hash = DigestUtils.sha1Hex(password); // SHA1 is weak

// ✅ USE BCRYPT, SCRYPT, or ARGON2
```

### BCrypt (Recommended)

```java
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class PasswordService {

    private final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12);

    public String hashPassword(String plainPassword) {
        return encoder.encode(plainPassword);
        // Output: $2a$12$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy
    }

    public boolean verifyPassword(String plainPassword, String hashedPassword) {
        return encoder.matches(plainPassword, hashedPassword);
    }
}
```

### Argon2 (Most Secure, Modern)

```java
import org.springframework.security.crypto.argon2.Argon2PasswordEncoder;

public class Argon2PasswordService {

    // Parameters: saltLength, hashLength, parallelism, memory, iterations
    private final Argon2PasswordEncoder encoder =
        new Argon2PasswordEncoder(16, 32, 1, 65536, 3);

    public String hashPassword(String password) {
        return encoder.encode(password);
    }

    public boolean verifyPassword(String password, String hash) {
        return encoder.matches(password, hash);
    }
}
```

### Password Validation

```java
public class PasswordValidator {

    private static final int MIN_LENGTH = 12;
    private static final Pattern UPPERCASE = Pattern.compile("[A-Z]");
    private static final Pattern LOWERCASE = Pattern.compile("[a-z]");
    private static final Pattern DIGIT = Pattern.compile("\\d");
    private static final Pattern SPECIAL = Pattern.compile("[!@#$%^&*(),.?\":{}|<>]");

    public ValidationResult validate(String password) {
        List<String> errors = new ArrayList<>();

        if (password.length() < MIN_LENGTH) {
            errors.add("Password must be at least " + MIN_LENGTH + " characters");
        }
        if (!UPPERCASE.matcher(password).find()) {
            errors.add("Password must contain at least one uppercase letter");
        }
        if (!LOWERCASE.matcher(password).find()) {
            errors.add("Password must contain at least one lowercase letter");
        }
        if (!DIGIT.matcher(password).find()) {
            errors.add("Password must contain at least one digit");
        }
        if (!SPECIAL.matcher(password).find()) {
            errors.add("Password must contain at least one special character");
        }
        if (isCommonPassword(password)) {
            errors.add("Password is too common");
        }

        return new ValidationResult(errors.isEmpty(), errors);
    }

    private boolean isCommonPassword(String password) {
        // Check against list of common passwords
        return CommonPasswords.contains(password.toLowerCase());
    }
}
```

---

## Input Validation & Sanitization

### Whitelist Validation (Preferred)

```java
public class InputValidator {

    // Whitelist: Only allow expected characters
    private static final Pattern SAFE_USERNAME = Pattern.compile("^[a-zA-Z0-9_]{3,20}$");
    private static final Pattern SAFE_EMAIL = Pattern.compile(
        "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$");

    public void validateUsername(String username) {
        if (username == null || !SAFE_USERNAME.matcher(username).matches()) {
            throw new ValidationException("Invalid username format");
        }
    }

    public void validateEmail(String email) {
        if (email == null || !SAFE_EMAIL.matcher(email).matches()) {
            throw new ValidationException("Invalid email format");
        }
    }
}
```

### Bean Validation (JSR-380)

```java
public class CreateUserRequest {

    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 20, message = "Username must be 3-20 characters")
    @Pattern(regexp = "^[a-zA-Z0-9_]+$", message = "Username can only contain letters, numbers, and underscores")
    private String username;

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 12, message = "Password must be at least 12 characters")
    private String password;

    @Min(value = 0, message = "Age cannot be negative")
    @Max(value = 150, message = "Invalid age")
    private Integer age;
}

@RestController
public class UserController {

    @PostMapping("/users")
    public ResponseEntity<User> createUser(@Valid @RequestBody CreateUserRequest request) {
        // Request is validated before reaching here
        return ResponseEntity.ok(userService.create(request));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, List<String>>> handleValidationErrors(
            MethodArgumentNotValidException ex) {

        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.toList());

        return ResponseEntity.badRequest().body(Map.of("errors", errors));
    }
}
```

---

## SQL Injection Prevention

### Vulnerable Code

```java
// ❌ VULNERABLE TO SQL INJECTION
public User findByUsername(String username) {
    String sql = "SELECT * FROM users WHERE username = '" + username + "'";
    // Input: admin' OR '1'='1
    // Results in: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
    return jdbcTemplate.queryForObject(sql, User.class);
}
```

### Parameterized Queries (Safe)

```java
// ✅ SAFE: Using PreparedStatement
public User findByUsername(String username) {
    String sql = "SELECT * FROM users WHERE username = ?";
    return jdbcTemplate.queryForObject(sql, new Object[]{username},
        (rs, rowNum) -> new User(rs.getLong("id"), rs.getString("username")));
}

// ✅ SAFE: Using named parameters
public User findByUsernameNamed(String username) {
    String sql = "SELECT * FROM users WHERE username = :username";
    MapSqlParameterSource params = new MapSqlParameterSource("username", username);
    return namedParameterJdbcTemplate.queryForObject(sql, params, userRowMapper);
}
```

### JPA (Safe by Default)

```java
// ✅ SAFE: JPA Derived Queries
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}

// ✅ SAFE: JPQL with parameters
@Query("SELECT u FROM User u WHERE u.username = :username")
Optional<User> findByUsernameJpql(@Param("username") String username);

// ❌ VULNERABLE: Concatenating in @Query
@Query("SELECT u FROM User u WHERE u.username = '" + username + "'")  // DON'T DO THIS
```

### Dynamic Queries with Criteria API

```java
// ✅ SAFE: Criteria API for dynamic queries
public List<User> searchUsers(String username, String email, Integer minAge) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> user = query.from(User.class);

    List<Predicate> predicates = new ArrayList<>();

    if (username != null) {
        predicates.add(cb.like(cb.lower(user.get("username")),
            "%" + username.toLowerCase() + "%"));
    }
    if (email != null) {
        predicates.add(cb.equal(user.get("email"), email));
    }
    if (minAge != null) {
        predicates.add(cb.greaterThanOrEqualTo(user.get("age"), minAge));
    }

    query.where(predicates.toArray(new Predicate[0]));
    return entityManager.createQuery(query).getResultList();
}
```

---

## XSS Prevention

### Types of XSS

1. **Reflected XSS**: Malicious script in URL parameters
2. **Stored XSS**: Malicious script stored in database
3. **DOM-based XSS**: Manipulation of client-side JavaScript

### Output Encoding

```java
import org.owasp.encoder.Encode;

public class XSSPrevention {

    // Encode for HTML context
    public String safeHtml(String input) {
        return Encode.forHtml(input);
        // <script>alert('xss')</script>
        // becomes: &lt;script&gt;alert(&#39;xss&#39;)&lt;/script&gt;
    }

    // Encode for JavaScript context
    public String safeJavaScript(String input) {
        return Encode.forJavaScript(input);
    }

    // Encode for URL parameter
    public String safeUrl(String input) {
        return Encode.forUriComponent(input);
    }

    // Encode for CSS
    public String safeCss(String input) {
        return Encode.forCssString(input);
    }
}
```

### Content Security Policy (CSP)

```java
@Configuration
public class SecurityHeadersConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.headers(headers -> headers
            .contentSecurityPolicy(csp -> csp
                .policyDirectives(
                    "default-src 'self'; " +
                    "script-src 'self' 'nonce-{random}'; " +
                    "style-src 'self' 'unsafe-inline'; " +
                    "img-src 'self' data: https:; " +
                    "font-src 'self'; " +
                    "connect-src 'self' https://api.example.com; " +
                    "frame-ancestors 'none'; " +
                    "form-action 'self'"
                ))
            .xssProtection(xss -> xss.headerValue(
                XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK))
        );

        return http.build();
    }
}
```

### HTML Sanitization (for Rich Text)

```java
import org.owasp.html.PolicyFactory;
import org.owasp.html.Sanitizers;

public class HtmlSanitizer {

    private static final PolicyFactory POLICY = Sanitizers.FORMATTING
            .and(Sanitizers.LINKS)
            .and(Sanitizers.BLOCKS)
            .and(Sanitizers.IMAGES);

    public String sanitize(String untrustedHtml) {
        return POLICY.sanitize(untrustedHtml);
        // Removes script tags, event handlers, etc.
        // Keeps safe HTML like <b>, <a>, <p>
    }
}
```

---

## CSRF Protection

### How CSRF Works

Attacker tricks authenticated user into making unwanted requests.

### Spring Security CSRF Protection

```java
@Configuration
public class CsrfConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                // For traditional web apps (form-based)
                .csrfTokenRepository(HttpSessionCsrfTokenRepository())

                // For SPAs with cookies
                // .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())

                // Ignore CSRF for specific paths (use carefully!)
                .ignoringRequestMatchers("/api/webhooks/**")
            );

        return http.build();
    }
}
```

### CSRF Token in Forms (Thymeleaf)

```html
<form th:action="@{/transfer}" method="post">
  <!-- Token automatically added by Spring Security -->
  <input
    type="hidden"
    th:name="${_csrf.parameterName}"
    th:value="${_csrf.token}"
  />
  <input type="text" name="amount" />
  <button type="submit">Transfer</button>
</form>
```

### CSRF for REST APIs with JWT

```java
// For stateless APIs using JWT, CSRF is typically disabled
// because there's no cookie-based session to exploit

@Bean
public SecurityFilterChain apiSecurity(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf.disable())  // Disable for stateless JWT API
        .sessionManagement(session -> session
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));

    return http.build();
}
```

---

## Secure Session Management

### Session Configuration

```java
@Configuration
public class SessionConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .invalidSessionUrl("/login?invalid-session")
                .maximumSessions(1)  // Prevent concurrent sessions
                .maxSessionsPreventsLogin(true)  // Block new login
                .expiredUrl("/login?session-expired")
                .sessionRegistry(sessionRegistry())
            );

        return http.build();
    }

    @Bean
    public SessionRegistry sessionRegistry() {
        return new SessionRegistryImpl();
    }
}
```

### Secure Cookie Configuration

```yaml
server:
  servlet:
    session:
      cookie:
        http-only: true # Prevent JavaScript access
        secure: true # Only send over HTTPS
        same-site: strict # Prevent CSRF
        max-age: 3600 # 1 hour
        name: SESSIONID # Custom cookie name
      timeout: 30m # Session timeout
```

### Session Fixation Prevention

```java
http.sessionManagement(session -> session
    .sessionFixation()
    .migrateSession()  // Create new session, copy attributes (default)
    // .newSession()    // Create new session, don't copy
    // .changeSessionId() // Just change ID (Servlet 3.1+)
);
```

---

## JWT Security

### Secure JWT Generation

```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;

public class JwtService {

    // Use strong key (minimum 256 bits for HS256)
    private final SecretKey key = Keys.secretKeyFor(SignatureAlgorithm.HS256);

    public String generateToken(User user) {
        Instant now = Instant.now();

        return Jwts.builder()
            .setSubject(user.getId().toString())
            .setIssuedAt(Date.from(now))
            .setExpiration(Date.from(now.plus(1, ChronoUnit.HOURS)))  // Short expiry
            .setIssuer("my-app")
            .setAudience("my-app-api")
            .claim("roles", user.getRoles())
            .claim("email", user.getEmail())
            .signWith(key)
            .compact();
    }

    public Claims validateToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(key)
            .requireIssuer("my-app")
            .requireAudience("my-app-api")
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
}
```

### JWT Security Best Practices

```java
// ✅ DO:
- Use strong signing algorithms (RS256 or HS256 with 256+ bit key)
- Set short expiration times (15-60 minutes)
- Validate issuer (iss) and audience (aud)
- Use refresh tokens for long-lived sessions
- Store tokens securely (httpOnly cookies or secure storage)

// ❌ DON'T:
- Use "none" algorithm
- Store sensitive data in payload (it's only base64 encoded)
- Use long expiration times without refresh mechanism
- Store tokens in localStorage (vulnerable to XSS)
```

### Refresh Token Implementation

```java
@Service
public class TokenService {

    public TokenPair generateTokenPair(User user) {
        String accessToken = generateAccessToken(user);  // 15 min expiry
        String refreshToken = generateRefreshToken(user);  // 7 days expiry

        // Store refresh token hash in database
        RefreshTokenEntity entity = new RefreshTokenEntity();
        entity.setUserId(user.getId());
        entity.setTokenHash(hashToken(refreshToken));
        entity.setExpiresAt(Instant.now().plus(7, ChronoUnit.DAYS));
        refreshTokenRepository.save(entity);

        return new TokenPair(accessToken, refreshToken);
    }

    public TokenPair refreshTokens(String refreshToken) {
        // Validate refresh token
        Claims claims = validateRefreshToken(refreshToken);
        String userId = claims.getSubject();

        // Check if refresh token exists and not revoked
        String tokenHash = hashToken(refreshToken);
        RefreshTokenEntity entity = refreshTokenRepository
            .findByUserIdAndTokenHash(userId, tokenHash)
            .orElseThrow(() -> new InvalidTokenException("Invalid refresh token"));

        // Revoke old refresh token (token rotation)
        refreshTokenRepository.delete(entity);

        // Generate new token pair
        User user = userRepository.findById(Long.parseLong(userId)).orElseThrow();
        return generateTokenPair(user);
    }

    public void revokeAllTokens(Long userId) {
        refreshTokenRepository.deleteAllByUserId(userId);
    }
}
```

---

## OWASP Top 10 for Java

### 1. Broken Access Control

```java
// ❌ Vulnerable: IDOR (Insecure Direct Object Reference)
@GetMapping("/orders/{orderId}")
public Order getOrder(@PathVariable Long orderId) {
    return orderRepository.findById(orderId).orElseThrow();  // No ownership check!
}

// ✅ Secure: Verify ownership
@GetMapping("/orders/{orderId}")
public Order getOrder(@PathVariable Long orderId, @AuthenticationPrincipal User user) {
    Order order = orderRepository.findById(orderId).orElseThrow();
    if (!order.getUserId().equals(user.getId())) {
        throw new AccessDeniedException("Not your order");
    }
    return order;
}
```

### 2. Cryptographic Failures

```java
// ❌ Weak: MD5, SHA1, DES
// ✅ Strong: SHA-256+, AES-256-GCM, BCrypt/Argon2 for passwords
```

### 3. Injection

```java
// Use parameterized queries (covered in SQL Injection section)
```

### 4. Insecure Design

```java
// Implement rate limiting, account lockout, proper error handling
```

### 5. Security Misconfiguration

```yaml
# ✅ Disable debug in production
spring:
  profiles:
    active: prod
# ✅ Remove default credentials
# ✅ Keep dependencies updated
# ✅ Disable unnecessary features
```

### 6. Vulnerable Components

```xml
<!-- Use dependency check plugins -->
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>8.4.0</version>
</plugin>
```

### 7. Authentication Failures

```java
// Implement:
// - Strong password policies
// - Multi-factor authentication
// - Account lockout after failed attempts
// - Secure password reset
```

### 8. Software and Data Integrity Failures

```java
// Verify signatures, use secure CI/CD, integrity checks
```

### 9. Security Logging Failures

```java
// Log security events (login, access denied, etc.)
// Don't log sensitive data (passwords, tokens)
```

### 10. Server-Side Request Forgery (SSRF)

```java
// ❌ Vulnerable
@GetMapping("/fetch")
public String fetchUrl(@RequestParam String url) {
    return restTemplate.getForObject(url, String.class);  // Can access internal resources!
}

// ✅ Secure: Whitelist allowed domains
public String fetchUrl(String url) {
    URL parsedUrl = new URL(url);
    if (!ALLOWED_HOSTS.contains(parsedUrl.getHost())) {
        throw new SecurityException("Host not allowed");
    }
    return restTemplate.getForObject(url, String.class);
}
```

---

## Interview Questions

### Fundamentals

1. **Difference between authentication and authorization?**

   - Authentication: Verify identity (who are you?). Authorization: Verify permissions (what can you do?).

2. **What is the principle of least privilege?**
   - Grant minimum permissions necessary. Reduces attack surface and limits damage from compromises.

### Cryptography

3. **When would you use symmetric vs asymmetric encryption?**

   - Symmetric (AES): Fast, for bulk data, requires shared key. Asymmetric (RSA): Key exchange, digital signatures, slower.

4. **Why should you never use MD5 or SHA1 for password hashing?**

   - Too fast (allows brute force), no salt, known collisions. Use BCrypt/Argon2 with built-in salt and work factor.

5. **What is a salt and why is it important?**
   - Random value added to password before hashing. Prevents rainbow table attacks and ensures identical passwords have different hashes.

### Web Security

6. **How does SQL injection work and how do you prevent it?**

   - Attacker injects SQL via user input. Prevent with parameterized queries, never concatenate user input into SQL.

7. **Explain XSS and prevention strategies.**

   - Attacker injects malicious scripts. Prevent with output encoding, Content Security Policy, input validation.

8. **What is CSRF and when is protection needed?**
   - Attacker tricks user into making unwanted requests. Needed for state-changing operations with session cookies. Use CSRF tokens.

### JWT

9. **What security considerations apply to JWT?**

   - Strong signing algorithm, short expiry, validate claims (iss, aud, exp), don't store sensitive data in payload, secure storage.

10. **Why use refresh tokens with JWT?**
    - Short-lived access tokens limit exposure window. Refresh tokens allow session extension without re-authentication. Can be revoked.

### Spring Security

11. **How does Spring Security filter chain work?**

    - Chain of filters processing each request. Each filter handles specific concern (CSRF, auth, etc.). Order matters.

12. **Difference between @Secured, @PreAuthorize, @PostAuthorize?**
    - @Secured: Simple role check. @PreAuthorize: SpEL expressions, method parameters. @PostAuthorize: Check after method execution, access return value.

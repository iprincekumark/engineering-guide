# 06 — Spring Security

> **Goal:** Understand authentication, authorization, JWT, OAuth2, CORS, CSRF, and how Spring Security's filter chain protects your backend.

---

## Table of Contents

1. [What is Spring Security?](#1-what-is-spring-security)
2. [Authentication vs Authorization](#2-authentication-vs-authorization)
3. [Security Filter Chain](#3-security-filter-chain)
4. [SecurityFilterChain Configuration](#4-securityfilterchain-configuration)
5. [JWT Authentication](#5-jwt-authentication)
6. [OAuth2 Basics](#6-oauth2-basics)
7. [Password Encoding](#7-password-encoding)
8. [CORS & CSRF](#8-cors--csrf)
9. [Method-Level Security & RBAC](#9-method-level-security--rbac)
10. [Interview Notes](#10-interview-notes)
11. [Summary](#11-summary)
12. [References](#12-references)

---

## 1. What is Spring Security?

**Definition:**
Spring Security is a powerful, customizable authentication and access-control framework for Spring-based applications. It provides comprehensive security services: authentication, authorization, and protection against common exploits (CSRF, session fixation, clickjacking).

**Why it exists:**
Security is non-negotiable. Without a framework, developers implement ad-hoc security — leading to vulnerabilities. Spring Security provides battle-tested, standards-based security.

**Real-life analogy:**
A multi-layered building security system: front gate (authentication — who are you?), ID verification (credentials check), floor access cards (authorization — what can you access?), and CCTV/alarms (exploit protection).

**Key characteristics:**
- Filter-based architecture integrating with the Servlet container
- Supports form login, HTTP Basic, JWT, OAuth2, SAML
- Method-level security with SpEL expressions
- Active defense against CSRF, XSS, session fixation

---

## 2. Authentication vs Authorization

**Definition:**
- **Authentication:** Verifying identity — "who are you?" (Login, JWT validation, OAuth2 token exchange)
- **Authorization:** Verifying permissions — "what can you access?" (Role checks, endpoint restrictions)

```
Request → AUTHENTICATION (identity check) → AUTHORIZATION (permission check) → Controller
```

---

## 3. Security Filter Chain

**Definition:**
An ordered list of servlet filters applied to every HTTP request. Each filter handles a specific security concern.

```
Request → SecurityContextFilter → CorsFilter → CsrfFilter → LogoutFilter
  → AuthenticationFilter (JWT/Form) → ExceptionTranslationFilter
  → AuthorizationFilter → Controller
```

---

## 4. SecurityFilterChain Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .cors(cors -> cors.configurationSource(corsConfigSource()))
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .requestMatchers(HttpMethod.GET, "/api/v1/products/**").permitAll()
                .requestMatchers("/api/v1/**").authenticated()
                .anyRequest().authenticated()
            )
            .authenticationProvider(authenticationProvider)
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
            .build();
    }

    @Bean
    public CorsConfigurationSource corsConfigSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("https://example.com"));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
        config.setAllowedHeaders(List.of("*"));
        config.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}
```

---

## 5. JWT Authentication

**Definition:**
JWT (JSON Web Token) is a compact, self-contained token for stateless authentication. The server generates a JWT after login; the client sends it with every request in the `Authorization: Bearer <token>` header.

**Why it exists:**
Session-based auth stores state on the server. In distributed microservices, sharing sessions is complex. JWT is stateless — the token itself contains identity claims.

**Real-life analogy:**
A concert wristband. Verified once at the gate (login), you wear it at every stage (request) — no one calls the ticket booth again.

**JWT structure:** `Header.Payload.Signature`

```java
// JWT Service
@Service
public class JwtService {
    @Value("${jwt.secret}") private String secretKey;
    @Value("${jwt.expiration}") private long expirationMs;

    public String generateToken(UserDetails userDetails) {
        return Jwts.builder()
            .subject(userDetails.getUsername())
            .claim("roles", userDetails.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority).toList())
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + expirationMs))
            .signWith(getSigningKey())
            .compact();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        return extractUsername(token).equals(userDetails.getUsername())
            && !isTokenExpired(token);
    }

    public String extractUsername(String token) {
        return Jwts.parser().verifyWith(getSigningKey()).build()
            .parseSignedClaims(token).getPayload().getSubject();
    }

    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(secretKey));
    }
}

// JWT Filter
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
            HttpServletResponse response, FilterChain chain) throws Exception {
        String authHeader = request.getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            chain.doFilter(request, response);
            return;
        }
        String jwt = authHeader.substring(7);
        String username = jwtService.extractUsername(jwt);
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails user = userDetailsService.loadUserByUsername(username);
            if (jwtService.isTokenValid(jwt, user)) {
                var authToken = new UsernamePasswordAuthenticationToken(
                    user, null, user.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        chain.doFilter(request, response);
    }
}

// Auth Controller
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody @Valid LoginRequest req) {
        authManager.authenticate(new UsernamePasswordAuthenticationToken(req.email(), req.password()));
        UserDetails user = userDetailsService.loadUserByUsername(req.email());
        return ResponseEntity.ok(new AuthResponse(jwtService.generateToken(user)));
    }
}
```

---

## 6. OAuth2 Basics

**Definition:**
OAuth2 is an authorization framework enabling third-party apps to access user resources via access tokens without exposing credentials ("Login with Google/GitHub").

| Flow | Use Case |
|------|----------|
| Authorization Code | Server-side web apps (most secure) |
| PKCE | SPAs, mobile apps |
| Client Credentials | Machine-to-machine |

```yaml
# Spring Boot as OAuth2 Resource Server
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://accounts.google.com
```

---

## 7. Password Encoding

**Definition:**
One-way hashing of passwords before database storage. Never store plaintext passwords.

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12); // ~250ms per hash
}
```

| Encoder | Recommended? |
|---------|-------------|
| `BCryptPasswordEncoder` | ✅ Default choice |
| `Argon2PasswordEncoder` | ✅ Best (memory-hard) |
| `SCryptPasswordEncoder` | ✅ Good alternative |
| `NoOpPasswordEncoder` | ❌ NEVER in production |

---

## 8. CORS & CSRF

**CORS:** Browser security restricting cross-origin requests. Backend must explicitly allow trusted origins.

**CSRF:** Attack tricking browsers into making unwanted authenticated requests. **Disable for stateless JWT APIs** (the attacker can't access the `Authorization` header). **Keep for session-based apps.**

```java
http.csrf(csrf -> csrf.disable()) // Safe for stateless JWT APIs
```

---

## 9. Method-Level Security & RBAC

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }

@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
public UserDto getUser(Long userId) { ... }

@PostAuthorize("returnObject.email == authentication.principal.username")
public UserDto getCurrentUser() { ... }
```

**RBAC:** Users → Roles → Permissions hierarchy. Assign roles to users; permissions to roles.

```java
@Entity
public class User {
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(name = "user_roles")
    private Set<Role> roles = new HashSet<>();
}

@Entity
public class Role {
    @Enumerated(EnumType.STRING)
    private RoleName name; // ROLE_USER, ROLE_ADMIN
    @ManyToMany(fetch = FetchType.EAGER)
    private Set<Permission> permissions;
}
```

---

## 10. Interview Notes

1. **How does the Security filter chain work?** — Ordered chain of filters. Each handles a concern (CORS, CSRF, auth). Request passes through all filters.
2. **How does JWT auth work?** — Login → validate → generate JWT. Requests → extract from header → validate → set SecurityContext.
3. **`hasRole()` vs `hasAuthority()`?** — `hasRole("ADMIN")` auto-prefixes `ROLE_`. `hasAuthority("ROLE_ADMIN")` checks the exact string.
4. **Why disable CSRF for REST APIs?** — CSRF exploits cookie-based auth. Stateless JWTs in headers are immune.
5. **`@PreAuthorize` vs `@PostAuthorize`?** — Pre: checked before execution. Post: checked after (can inspect return value).
6. **How to secure Actuator endpoints?** — `requestMatchers("/actuator/**").hasRole("ADMIN")`. Expose only `/health` publicly.

---

## 11. Summary

| Topic | Key Takeaway |
|-------|-------------|
| Authentication | Verify identity (JWT, OAuth2, form login) |
| Authorization | Verify permissions (roles, authorities) |
| Filter Chain | Ordered filters process every request |
| JWT | Stateless, self-contained — ideal for REST APIs |
| OAuth2 | Delegated auth via external providers |
| CORS | Explicitly allow cross-origin requests |
| CSRF | Disable for stateless APIs |
| RBAC | Users → Roles → Permissions |

---

## 12. References

- [Spring Security Documentation](https://docs.spring.io/spring-security/reference/)
- [Baeldung — Spring Security](https://www.baeldung.com/security-spring)
- [JWT.io](https://jwt.io/)
- [RFC 7519 — JSON Web Token](https://datatracker.ietf.org/doc/html/rfc7519)
- [OAuth 2.0 Specification](https://oauth.net/2/)
- [OWASP Security Cheat Sheets](https://cheatsheetseries.owasp.org/)

---

> **Previous:** [← 05 — Spring Data JPA & Hibernate](./05_Spring_Data_JPA_Hibernate.md)
> **Next:** [07 — Database Design & Persistence →](./07_Database_Design_Persistence.md)

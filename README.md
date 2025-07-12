🔐 JWT Authentication

1-> When a client attempts to log in by sending credentials (username and password) to the /api/public/auth endpoint, Spring Security first passes the request through the configured SecurityFilterChain. Since /api/public/** is marked as publicly accessible, the request bypasses authentication and reaches the AuthController.

2-> In AuthController, the AuthenticationManager authenticates the user using a UsernamePasswordAuthenticationToken. Internally, this triggers the CustomUserDetailsService to load user details from the database. If authentication succeeds, the user details are passed to the JwtUtils class, which generates a JWT token. This token includes claims like the username and roles, and is signed using a secret key via HMAC-SHA256. The token is then returned to the client in the response.

3-> For subsequent requests to protected endpoints (like /api/admin/**), the client includes the JWT in the Authorization header in the format Bearer <token>. These requests are intercepted by the JwtAuthenticationFilter, a OncePerRequestFilter that runs before the request hits the controller.

4-> The JwtAuthenticationFilter extracts the token from the header and uses JwtUtils to parse and validate it. If valid, it extracts the username from the token and uses CustomUserDetailsService again to load the full user. Then it creates an Authentication object and sets it in SecurityContextHolder, which makes the user authenticated for the current request.

5-> Finally, Spring Security applies any authorization rules defined in the SecurityFilterChain. If the user has the required roles or permissions, the request proceeds to the controller. Otherwise, Spring returns a 403 Forbidden or 401 Unauthorized error, depending on the situation.

### 🔐 JWT Authentication Flow

1. **Login** → User sends username & password to `/api/public/auth`.
2. **Authentication** → Spring authenticates using `AuthenticationManager` & `UserDetailsService`.
3. **Token Generation** → JWT is created using `JwtUtils` and returned to the client.
4. **Request with Token** → Client sends JWT in the `Authorization` header.
5. **Token Validation** → `JwtAuthenticationFilter` validates the token.
6. **Set Security Context** → Authenticated user is set in `SecurityContextHolder`.
7. **Authorization Check** → Roles are checked, controller is accessed.

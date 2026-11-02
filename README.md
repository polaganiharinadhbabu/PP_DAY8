# Day 8 — JWT, OAuth 2.0, Security & System Design

**Name:** Polagani Harinadh Babu  
**Date:** 02 November 2026  
**Topic:** JWT, OAuth 2.0, Security & System Design  
**Technologies:** Python, FastAPI, JWT, OAuth 2.0, SQLAlchemy, SQLite, Argon2, pytest

---

## 1. Objective

The objective of this assignment is to develop a secure FastAPI application with user authentication, JWT access tokens, refresh-token rotation, OAuth 2.0 integration, role-based access control (RBAC), audit logging, and automated testing.

The assignment also covers Authorization Code + PKCE and the high-level design of a scalable product API.

## 2. Task A — FastAPI Authentication (40 Minutes)

### Requirements

- Implement user registration and login.
- Hash passwords securely.
- Implement access-token validation.
- Create a protected user profile route.

### Implementation

The application provides registration and login endpoints. Passwords are hashed using Argon2 before being stored in the database.

After successful login, the application issues an access token and a refresh token. Protected endpoints validate the access token before returning user information.

### Authentication Flow

```text
User Registration
       |
       v
Validate User Details
       |
       v
Hash Password
       |
       v
Save User in Database
       |
       v
User Login
       |
       v
Verify Password
       |
       v
Generate Access Token
and Refresh Token
       |
       v
Access Protected Profile
```

### API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/register` | Register a new user |
| POST | `/login` | Authenticate a user |
| POST | `/refresh` | Rotate refresh token |
| POST | `/logout` | Revoke refresh token |
| GET | `/profile` | Access protected user profile |
| GET | `/health` | Check application health |

### Security Measures

- Passwords are not stored as plain text.
- Access tokens have an expiry time.
- JWT validation checks the token signature and configured claims.
- Protected routes require valid authentication.
- Refresh tokens are stored as hashes in the database.
- Logout revokes refresh-token credentials.

## 3. Task B — OAuth Integration (35 Minutes)

### Requirements

- Draw the Authorization Code + PKCE flow.
- Implement a provider callback using a test OAuth application.

### Authorization Code + PKCE Flow

```text
       User / Client
             |
             v
   Generate Code Verifier
   and Code Challenge
             |
             v
     Authorization Request
             |
             v
    OAuth Authorization
          Server
             |
             v
       User Login
       and Consent
             |
             v
     Authorization Code
             |
             v
       Callback URL
             |
             v
 Exchange Code + Verifier
             |
             v
      Token Response
             |
             v
    Authenticated Client
```

### Implementation

The application includes optional Google OAuth login and callback routes using an OAuth client library.

To test the integration:

1. Create a test OAuth application with the identity provider.
2. Configure the client ID and client secret.
3. Register the correct callback URL.
4. Store credentials in the local `.env` file.
5. Start the application and test the login flow.

For the local configuration, the callback URL is:

`http://127.0.0.1:8000/auth/google/callback`

The configured redirect URL must match the one registered with Google.

**PKCE note:** The diagram describes the required Authorization Code + PKCE flow. Verify that the provider integration actually generates and validates a code verifier and code challenge before claiming PKCE implementation is complete.

## 4. Task C — RBAC and Audit (25 Minutes)

### Requirements

- Create roles and permissions.
- Protect administrator routes.
- Record audit events for role changes.

### Role-Based Access Control

The application uses role-based access control to restrict access to protected resources.

| Role | Permission |
|---|---|
| `admin` | Access administrator-only functionality |
| `support` | Access permitted support functionality |
| `customer` | Access their own profile and permitted customer functionality |

### RBAC Flow

```text
User Request
     |
     v
Validate Access Token
     |
     v
Identify User and Role
     |
     v
Check Required Permission
     |
     +-------------------+
     |                   |
     v                   v
 Permission Granted   Permission Denied
     |                   |
     v                   v
 Return Resource      HTTP 403
```

### Protected Endpoints

- `GET /admin/users` — list users; admin access required.
- `PATCH /admin/users/{user_id}/role` — change a user's role; admin access required.
- `GET /audit` — view recent audit events; admin access required.

### Audit Logging

Audit logs provide a record of important security-related actions.

A role-change audit event should contain:

- Actor who performed the change.
- User whose role was changed.
- Previous role.
- New role.
- Event timestamp.
- Success or failure outcome.

Passwords, raw access tokens, refresh tokens, and other secrets must not be recorded in audit logs.

## 5. Task D — HLD Design (40 Minutes)

### Requirements

Draw a scalable product API architecture containing:

- CDN
- Load balancer
- Reverse proxy
- FastAPI instances
- Cache
- Database
- Queue

### High-Level Design

```text
                Client / Browser
                       |
                       v
                      CDN
                       |
                       v
                Load Balancer
                       |
                       v
                Reverse Proxy
                       |
          +------------+------------+
          |            |            |
          v            v            v
      FastAPI 1    FastAPI 2    FastAPI N
          |            |            |
          +------------+------------+
                       |
          +------------+------------+
          |            |            |
          v            v            v
        Cache       Database       Queue
                                    |
                                    v
                            Background Worker
```

### Component Explanation

**CDN:** Delivers static assets and suitable cacheable public content closer to users.

**Load Balancer:** Distributes incoming requests across healthy application instances.

**Reverse Proxy:** Routes requests to backend services and can handle TLS termination.

**FastAPI Instances:** Process API requests, authentication, authorization, and business logic.

**Cache:** Stores frequently accessed data to reduce repeated database queries.

**Database:** Stores persistent data such as users, refresh-token records, and audit events.

**Queue:** Holds background jobs that do not need to finish during the API request.

**Background Worker:** Processes queued jobs independently of the API instances.

Private user data and authenticated profile responses must not be placed in a public cache.

## 6. Project Structure

```text
day8_secure_backend/
├── app/
│   ├── __init__.py
│   ├── database.py
│   ├── main.py
│   ├── models.py
│   ├── schemas.py
│   └── security.py
├── tests/
│   ├── conftest.py
│   └── test_auth.py
├── .env.example
├── .gitignore
├── requirements.txt
└── README.md
```

## 7. Main API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Health check |
| POST | `/register` | Register a customer |
| POST | `/login` | Authenticate and issue tokens |
| POST | `/refresh` | Rotate refresh token |
| POST | `/logout` | Revoke refresh credentials |
| GET | `/profile` | Get authenticated user's profile |
| GET | `/admin/users` | List users; admin only |
| PATCH | `/admin/users/{user_id}/role` | Change user role; admin only |
| GET | `/audit` | View audit events; admin only |
| GET | `/auth/google` | Start Google OAuth when configured |
| GET | `/auth/google/callback` | Handle Google OAuth callback |

Open `/docs` after starting the application to inspect the available endpoints in Swagger UI.

## 8. Traffic Estimate and Scalability

For an example system, assume:

- Daily active users: 100,000
- Requests per user per day: 20
- Total daily requests: 100,000 × 20 = 2,000,000
- Average requests per second: 2,000,000 ÷ 86,400 ≈ 23
- Peak traffic at 5× average: approximately 115 requests per second

These figures are planning assumptions, not measured performance results.

### Scalability Improvements

- Run multiple FastAPI instances behind a load balancer.
- Add database indexes for frequently queried fields.
- Use Redis for suitable cache data and distributed rate limiting.
- Use a queue for background tasks.
- Monitor request latency, error rates, and database performance.
- Perform load testing before production deployment.


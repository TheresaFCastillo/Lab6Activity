# Node.js TypeScript MySQL Boilerplate API

A fully functional authentication API built with Node.js, TypeScript, Express, Sequelize (MySQL), and JWTs.

## Features

- **User Registration & Email Verification** — Users sign up and must verify via email link
- **JWT Authentication** — Short-lived JWT access tokens (15 min) + long-lived Refresh Tokens (7 days)
- **Refresh Token Rotation** — Each use of a refresh token generates a new one for security
- **Role-Based Access Control (RBAC)** — Admin and User roles
- **Account Management** — Forgot/reset password, full CRUD
- **Swagger UI** — Auto-generated API docs at `/api-docs`

## Project Structure

```
node-mysql-api/
├── _helpers/
│   ├── db.ts               # Sequelize DB connection & model init
│   ├── role.ts             # Role enum (Admin / User)
│   ├── send-email.ts       # Nodemailer email wrapper
│   └── swagger.ts          # Swagger UI route handler
├── _middleware/
│   ├── authorize.ts        # JWT auth + role-based authorization
│   ├── error-handler.ts    # Global error handler
│   └── validate-request.ts # Joi request body validation
├── accounts/
│   ├── account.model.ts       # Sequelize Account model
│   ├── refresh-token.model.ts # Sequelize RefreshToken model
│   ├── account.service.ts     # Business logic
│   └── accounts.controller.ts # Express routes
├── config.json             # DB, JWT secret, SMTP settings
├── swagger.yaml            # OpenAPI 3.0 spec
├── server.ts               # Express app entry point
├── tsconfig.json
└── package.json
```

## Setup

### 1. Install dependencies

```bash
npm install
```

### 2. Configure `config.json`

Update the following values:

```json
{
  "database": {
    "host": "localhost",
    "port": 3306,
    "user": "root",
    "password": "YOUR_MYSQL_PASSWORD",
    "database": "node_mysql_api"
  },
  "secret": "YOUR_JWT_SECRET_KEY",
  "emailFrom": "info@your-domain.com",
  "smtpOptions": {
    "host": "smtp.ethereal.email",
    "port": 587,
    "auth": {
      "user": "YOUR_ETHEREAL_USER",
      "pass": "YOUR_ETHEREAL_PASS"
    }
  }
}
```

> **Tip:** Get free test SMTP credentials at [https://ethereal.email](https://ethereal.email)

### 3. Run the server

```bash
# Development (with auto-reload)
npm run start:dev

# Production
npm start
```

### 4. Open Swagger UI

```
http://localhost:4000/api-docs
```

## API Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/accounts/authenticate` | Public | Login, returns JWT + refresh token cookie |
| POST | `/accounts/refresh-token` | Cookie | Get new JWT using refresh token |
| POST | `/accounts/revoke-token` | JWT | Revoke a refresh token |
| POST | `/accounts/register` | Public | Register new account |
| POST | `/accounts/verify-email` | Public | Verify email with token |
| POST | `/accounts/forgot-password` | Public | Request password reset email |
| POST | `/accounts/validate-reset-token` | Public | Validate reset token |
| POST | `/accounts/reset-password` | Public | Reset password |
| GET | `/accounts` | Admin JWT | Get all accounts |
| GET | `/accounts/:id` | JWT | Get account by ID |
| POST | `/accounts` | Admin JWT | Create account |
| PUT | `/accounts/:id` | JWT | Update account |
| DELETE | `/accounts/:id` | JWT | Delete account |

## Authentication Flow

1. **Register** → POST `/accounts/register` (sends verification email)
2. **Verify Email** → POST `/accounts/verify-email` with token from email
3. **Login** → POST `/accounts/authenticate` → get `jwtToken` + `refreshToken` cookie
4. **Access protected routes** → Add `Authorization: Bearer <jwtToken>` header
5. **Refresh JWT** → POST `/accounts/refresh-token` (uses cookie automatically)
6. **Logout** → POST `/accounts/revoke-token`

## Security Notes

- JWT tokens expire after **15 minutes**
- Refresh tokens expire after **7 days**
- Refresh tokens are stored in **HttpOnly cookies** (XSS protection)
- **Refresh Token Rotation**: each refresh issues a new token and revokes the old one
- Password hashing uses **bcrypt** (10 salt rounds)
- The first registered account is automatically assigned the **Admin** role

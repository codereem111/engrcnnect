# EngrConnect API Contract

### Overview

All endpoints should be prefixed with `/api`. For example, a request to
log in will be sent to `POST /api/login`. The API uses **JSON** for all
request and response bodies.

------------------------------------------------------------------------

### Authentication Endpoints

These endpoints handle user registration, login, and session management.

#### 1. Signup

- **Endpoint**: `POST /api/signup`
- **Purpose**: Registers a new user account.
- **Request Body**: A JSON object containing the following fields:
  - **`userId`**: `string` (The user's matriculation number or email)
  - **`password`**: `string` (The user's chosen password)
  - **`role`**: `string` (Either `"student"` or `"admin"`)
  - **`department`**: `string` (Required for students)
  - **`securityQuestion`**: `string` (Required for students)
  - **`securityAnswer`**: `string` (Required for students)
  - **`adminRole`**: `string` (Required for admins)
  - **`adminLevel`**: `string` (Required for admins)
- **Expected Response**:
  - **Success (201 Created)**:
    `{ "message": "User created successfully." }`
  - **Failure (400 Bad Request)**:
    `{ "error": "User ID already exists." }` or
    `{ "error": "Missing required fields." }`

#### 2. Login

- **Endpoint**: `POST /api/login`
- **Purpose**: Authenticates a user and provides a session token.
- **Request Body**: A JSON object containing:
  - **`userId`**: `string`
  - **`password`**: `string`
  - **`role`**: `string` (Optional, could be inferred from the user ID)
  - **`level`**: `string` (Required for student login)
- **Expected Response**:
  - **Success (200 OK)**:
    `{ "message": "Login successful.", "token": "your_jwt_token_here", "user": { "userId": "...", "role": "...", "department": "...", "level": "..." } }`
  - **Failure (400 Bad Request)**: `{ "error": "Invalid credentials." }`

------------------------------------------------------------------------

### Notification Endpoints

These endpoints handle the creation and retrieval of notifications.

#### 1. Fetch Notifications

- **Endpoint**: `GET /api/notifications`
- **Purpose**: Retrieves a list of all active notifications.
- **Request Parameters**:
  - **`department`**: `string` (Optional. Filter notifications by
    department. The default should be all notifications.)
- **Authentication**: No authentication required for viewing
  notifications.
- **Expected Response**:
  - **Success (200 OK)**: A JSON array of notification objects. Each
    object should have the following structure:
    - **`id`**: `string`
    - **`title`**: `string`
    - **`body`**: `string`
    - **`department`**: `string`
    - **`posterName`**: `string`
    - **`timestamp`**: `string` (ISO 8601 format)
    - **`expirationDate`**: `string` (Date string in "YYYY-MM-DD"
      format, can be `null`)
  - **Failure (500 Internal Server Error)**:
    `{ "error": "Server error." }`

#### 2. Post New Notification

- **Endpoint**: `POST /api/notifications`
- **Purpose**: Creates a new notification. This action is only for
  authenticated administrators.
- **Authentication**: Requires a valid **JWT** in the `Authorization`
  header (`Bearer <token>`). The backend should verify the token and the
  user's role to ensure they are an admin.
- **Request Body**: A JSON object containing the notification data:
  - **`title`**: `string`
  - **`body`**: `string`
  - **`department`**: `string`
  - **`expirationDate`**: `string` (Date string in "YYYY-MM-DD" format,
    optional)
- **Expected Response**:
  - **Success (201 Created)**:
    `{ "message": "Notification posted successfully." }`
  - **Failure (401 Unauthorized)**: `{ "error": "Access denied." }`
  - **Failure (400 Bad Request)**:
    `{ "error": "Missing required fields." }`

------------------------------------------------------------------------

### Other Endpoints

#### 1. Update Admin Role

- **Endpoint**: `PATCH /api/users/:userId/role`
- **Purpose**: Allows an admin to update their role and level.
- **Authentication**: Requires a valid **JWT**.
- **Request Body**: A JSON object containing:
  - **`adminRole`**: `string`
  - **`adminLevel`**: `string`
- **Expected Response**:
  - **Success (200 OK)**:
    `{ "message": "Admin role updated successfully." }`
  - **Failure (401 Unauthorized)**: `{ "error": "Access denied." }`

# EngrConnect Backend Package

This document provides a comprehensive overview of the EngrConnect API, including the contract and a reference implementation.

---

## Section 1: API Contract

### Overview

All endpoints should be prefixed with `/api`. The API uses **JSON** for all request and response bodies.

### Authentication Endpoints

#### 1. Signup

- **Endpoint**: `POST /api/signup`
- **Purpose**: Registers a new user account.
- **Request Body**: A JSON object containing the user's details.
    - **For Students:**
        ```json
        {
          "userId": "string",
          "password": "string",
          "role": "student",
          "department": "string",
          "securityQuestion": "string",
          "securityAnswer": "string"
        }
        ```
    - **For Admins:**
        ```json
        {
          "userId": "string",
          "password": "string",
          "role": "admin",
          "adminRole": "string",
          "adminLevel": "string"
        }
        ```
- **Expected Response**:
    - **Success (201 Created)**: `{ "message": "User created successfully." }`
    - **Failure (400 Bad Request)**: `{ "error": "User ID already exists." }` or `{ "error": "Missing required fields." }`

#### 2. Login

- **Endpoint**: `POST /api/login`
- **Purpose**: Authenticates a user and provides a session token.
- **Request Body**: A JSON object with login credentials.
    - **For Students:**
        ```json
        {
          "userId": "string",
          "password": "string",
          "role": "student",
          "level": "string"
        }
        ```
    - **For Admins:**
        ```json
        {
          "userId": "string",
          "password": "string",
          "role": "admin"
        }
        ```
- **Expected Response**:
    - **Success (200 OK)**:
        ```json
        {
          "message": "Login successful.",
          "token": "your_unique_jwt_token_here",
          "user": {
            "userId": "string",
            "role": "string",
            "department": "string",
            "level": "string",
            "adminRole": "string",
            "adminLevel": "string"
          }
        }
        ```
    - **Failure (400 Bad Request)**: `{ "error": "Invalid credentials." }`

---

### Notification Endpoints

#### 1. Fetch Notifications

- **Endpoint**: `GET /api/notifications`
- **Purpose**: Retrieves a list of active notifications.
- **Request**: This endpoint can accept an optional query parameter to filter by department.
    - `GET /api/notifications?department=Computer Engineering`
- **Expected Response**:
    - **Success (200 OK)**: An array of notification objects.
        ```json
        [
          {
            "id": "string",
            "title": "string",
            "body": "string",
            "department": "string",
            "posterName": "string",
            "timestamp": "string",
            "expirationDate": "string"
          }
        ]
        ```
    - **Failure (500 Internal Server Error)**: `{ "error": "Server error." }`

#### 2. Post New Notification

- **Endpoint**: `POST /api/notifications`
- **Purpose**: Creates a new notification. This action is only for authenticated administrators.
- **Authentication**: Requires a valid **JWT** in the `Authorization` header (`Bearer <token>`).
- **Request Body**: A JSON object containing the notification data.
    ```json
    {
      "title": "string",
      "body": "string",
      "department": "string",
      "expirationDate": "string"
    }
    ```
- **Expected Response**:
    - **Success (201 Created)**: `{ "message": "Notification posted successfully." }`
    - **Failure (401 Unauthorized)**: `{ "error": "Access denied. Admin role required." }`
    - **Failure (400 Bad Request)**: `{ "error": "Missing required fields." }`

---

### Other Endpoints

#### 1. Update Admin Role

- **Endpoint**: `PATCH /api/users/:userId/role`
- **Purpose**: Allows an admin to update their role and level. The `:userId` is a path parameter.
- **Authentication**: Requires a valid **JWT** in the `Authorization` header.
- **Request Body**: A JSON object containing:
    ```json
    {
      "adminRole": "string",
      "adminLevel": "string"
    }
    ```
- **Expected Response**:
    - **Success (200 OK)**: `{ "message": "Admin role updated successfully." }`
    - **Failure (400 Bad Request)**: `{ "error": "Missing required fields." }`
    - **Failure (401 Unauthorized)**: `{ "error": "Access denied. Admin role required." }`

---

## Section 2: Backend Code Reference

### **File: `app.py`**

```python
from flask import Flask, request, jsonify
from datetime import datetime, date
from flask_cors import CORS

app = Flask(__name__)
CORS(app)

# --- In-memory "databases" for demonstration purposes ---
users = {
    "student": {
        "22/30GR023": {
            "password": "password123",
            "securityQuestion": "What is your favorite color?",
            "securityAnswer": "blue",
            "level": "300",
            "department": "Computer Engineering"
        }
    },
    "admin": {
        "admin@engrconnect.edu": {
            "password": "password123",
            "adminRole": "HOD",
            "adminLevel": "All"
        }
    }
}

notifications = [
    {
        "id": "1",
        "title": "Exam Timetable",
        "body": "The final exam timetable has been posted on the faculty portal.",
        "department": "All",
        "posterName": "Engr Zakriya",
        "timestamp": datetime.now().isoformat(),
        "expirationDate": "2025-10-30"
    },
    {
        "id": "2",
        "title": "Lab Session Cancelled",
        "body": "The Thermodynamics lab session for MECH-301 is cancelled today due to maintenance.",
        "department": "Mechanical Engineering",
        "posterName": "Engr K.O abdulrahman",
        "timestamp": datetime.now().isoformat(),
        "expirationDate": "2025-08-17"
    }
]

# --- Helper function for authentication check ---
def is_admin(auth_header):
    return auth_header and "Bearer " in auth_header and auth_header.split(" ")[1] == "your_admin_jwt_token"

# --- API Endpoints ---
@app.route("/api/signup", methods=["POST"])
def signup():
    data = request.get_json()
    user_id = data.get("userId")
    password = data.get("password")
    role = data.get("role")
    department = data.get("department")
    security_question = data.get("securityQuestion")
    security_answer = data.get("securityAnswer")
    admin_role = data.get("adminRole")
    admin_level = data.get("adminLevel")

    if not user_id or not password or not role:
        return jsonify({"error": "Missing required fields."}), 400

    if role == "student":
        if not department or not security_question or not security_answer:
            return jsonify({"error": "Missing required fields for student."}), 400
        if user_id in users["student"]:
            return jsonify({"error": "User ID already exists."}), 400
        users["student"][user_id] = {
            "password": password,
            "department": department,
            "securityQuestion": security_question,
            "securityAnswer": security_answer,
            "level": user_id[3:5] + "00"
        }
    elif role == "admin":
        if not admin_role or not admin_level:
            return jsonify({"error": "Missing required fields for admin."}), 400
        if user_id in users["admin"]:
            return jsonify({"error": "User ID already exists."}), 400
        users["admin"][user_id] = {
            "password": password,
            "adminRole": admin_role,
            "adminLevel": admin_level
        }
    else:
        return jsonify({"error": "Invalid role specified."}), 400

    return jsonify({"message": "User created successfully."}), 201

@app.route("/api/login", methods=["POST"])
def login():
    data = request.get_json()
    user_id = data.get("userId")
    password = data.get("password")
    role = data.get("role")
    level = data.get("level")

    if not user_id or not password or not role:
        return jsonify({"error": "Invalid credentials."}), 400

    if role == "student":
        if not level:
            return jsonify({"error": "Invalid credentials."}), 400
        if user_id in users["student"] and users["student"][user_id]["password"] == password and users["student"][user_id]["level"] == level:
            token = "your_student_jwt_token"
            return jsonify({
                "message": "Login successful.",
                "token": token,
                "user": {
                    "userId": user_id,
                    "role": role,
                    "department": users["student"][user_id]["department"],
                    "level": users["student"][user_id]["level"]
                }
            }), 200
    elif role == "admin":
        if user_id in users["admin"] and users["admin"][user_id]["password"] == password:
            token = "your_admin_jwt_token"
            return jsonify({
                "message": "Login successful.",
                "token": token,
                "user": {
                    "userId": user_id,
                    "role": role,
                    "adminRole": users["admin"][user_id]["adminRole"],
                    "adminLevel": users["admin"][user_id]["adminLevel"]
                }
            }), 200

    return jsonify({"error": "Invalid credentials."}), 400

@app.route("/api/notifications", methods=["GET"])
def get_notifications():
    department = request.args.get("department", "All")
    today_str = date.today().isoformat()

    filtered_notifications = []
    for notif in notifications:
        is_active = notif.get("expirationDate") is None or notif.get("expirationDate") >= today_str
        matches_department = department == "All" or notif["department"] == "All" or notif["department"] == department
        if is_active and matches_department:
            filtered_notifications.append(notif)

    return jsonify(filtered_notifications), 200

@app.route("/api/notifications", methods=["POST"])
def post_notification():
    auth_header = request.headers.get("Authorization")
    if not is_admin(auth_header):
        return jsonify({"error": "Access denied. Admin role required."}), 401
    
    data = request.get_json()
    title = data.get("title")
    body = data.get("body")
    department = data.get("department")
    expiration_date = data.get("expirationDate")

    if not all([title, body, department]):
        return jsonify({"error": "Missing required fields."}), 400

    new_id = str(len(notifications) + 1)
    new_notification = {
        "id": new_id,
        "title": title,
        "body": body,
        "department": department,
        "posterName": "Mock Admin",
        "timestamp": datetime.now().isoformat(),
        "expirationDate": expiration_date
    }
    notifications.append(new_notification)

    return jsonify({"message": "Notification posted successfully."}), 201

@app.route("/api/users/<user_id>/role", methods=["PATCH"])
def update_admin_role(user_id):
    auth_header = request.headers.get("Authorization")
    if not is_admin(auth_header):
        return jsonify({"error": "Access denied. Admin role required."}), 401

    if user_id not in users["admin"]:
         return jsonify({"error": "Access denied. User not found or not an admin."}), 401

    data = request.get_json()
    new_admin_role = data.get("adminRole")
    new_admin_level = data.get("adminLevel")

    if not new_admin_role or not new_admin_level:
        return jsonify({"error": "Missing required fields."}), 400

    users["admin"][user_id]["adminRole"] = new_admin_role
    users["admin"][user_id]["adminLevel"] = new_admin_level

    return jsonify({"message": "Admin role updated successfully."}), 200

if __name__ == "__main__":
    app.run(debug=True)
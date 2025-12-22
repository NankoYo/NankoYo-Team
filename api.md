# Nankoyo System - Comprehensive API Documentation

# Nankoyo System - Comprehensive API Documentation

## Document Overview

This comprehensive document integrates core API specifications, test guidelines, permission control rules, and change logs for the Nankoyo closed-source team website and management system. It is designed for internal developers, testers, and system administrators to ensure consistent API usage, efficient testing, and smooth version iteration. All content is aligned with the system’s PHP-based architecture and `nankoyo.com` domain configuration.

---

## 1. API Core Specifications

### 1.1 Basic Information

- **API Prefixes**:

    - Management System APIs: `/api/admin/` (for internal administrative operations, requiring authentication)

    - Public Website APIs: `/api/web/` (for external public data access, most without authentication)

- **Request Methods**:

    - GET: Retrieve resource data (e.g., list queries, detail queries)

    - POST: Create or update resource data (e.g., add products, submit forms)

    - DELETE: Remove resource data (e.g., delete articles, logical deletion of products)

- **Data Format**:

    - Request: JSON (POST/PUT) or query parameters (GET/DELETE)

    - Response: Unified JSON structure with `code` (status), `msg` (message), and `data` (business data)

- **Authentication Mechanism**:

    - Management System APIs: Token-based authentication (carried in `Authorization` header: `Authorization: Bearer [Token]`)

    - Public APIs: No authentication required (except for sensitive interfaces like user registration/login)

- **Request Headers (Required for Management APIs)**:

|    Header Key|    Value Example|    Description|
|---|---|---|
|    Authorization|    Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...|    Authentication Token|
|    Content-Type|    application/json|    For JSON request bodies|
|    X-Request-Id|    uuid-1234-5678-90ab-cdef|    Optional, for request tracing|
### 1.2 Response Structure Standard

#### Success Response

```json

{
  "code": 200,
  "msg": "Operation successful",
  "data": {
    // Business-specific data (object/array/string)
  },
  "timestamp": 1718765432100
}
```

#### Error Response

```json

{
  "code": 400,
  "msg": "Invalid parameter: product title is required",
  "data": {},
  "timestamp": 1718765432100,
  "requestId": "uuid-1234-5678-90ab-cdef" // For troubleshooting
}
```

---

## 2. Core API Examples

### 2.1 Management System APIs (Requiring Authentication)

#### 2.1.1 Admin Login (Token Acquisition)

- **Endpoint**: `POST /api/admin/login`

- **Description**: Obtain authentication Token for admin operations

- **Request Body**:

    ```json
    
    {
      "username": "admin",  // Required, admin username (string)
      "password": "123456"  // Required, admin password (string)
    }
    ```

- **Response**:

    ```json
    
    {
      "code": 200,
      "msg": "Login successful",
      "data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...", // Valid for 2 hours
        "role": "super_admin", // Role: super_admin/content_admin/hr_admin
        "expireAt": 1718772632100 // Token expiration timestamp
      },
      "timestamp": 1718765432100
    }
    ```

- **Notes**: Reset default password immediately after first login; re-login when Token expires.

#### 2.1.2 Product Management APIs

|Operation|Endpoint|Method|Key Parameters|
|---|---|---|---|
|List Products|`/api/admin/product/list`|GET|`page` (default:1), `size`(default:10), `category_id`, `keyword`|
|Get Product Detail|`/api/admin/product/detail`|GET|`id` (required, product ID)|
|Create Product|`/api/admin/product/create`|POST|`title`, `summary`, `category_id`, `cover`, `status` (required)|
|Update Product|`/api/admin/product/update`|POST|`id` (required), `title`, `summary`, `status` (optional)|
|Delete Product|`/api/admin/product/delete`|DELETE|`id` (required, product ID)|
**Example: Create Product**

- Request Body:

    ```json
    
    {
      "title": "Nankoyo Project Management System", // 1-100 chars
      "summary": "A lightweight PM tool for small teams", // 1-500 chars
      "category_id": 3, // Valid category ID from /api/admin/category/list
      "cover": "/static/uploads/product/pm-tool-cover.png", // From media upload API
      "details": "# Product Details\nSupports task tracking and team collaboration", // Markdown/HTML
      "status": 1, // 1=published, 0=draft
      "specs": [
        {"name": "Version", "value": "V2.1"},
        {"name": "Support Users", "value": "Up to 50"}
      ]
    }
    ```

- Response:

    ```json
    
    {
      "code": 200,
      "msg": "Product created successfully",
      "data": {"id": 105},
      "timestamp": 1718766000000
    }
    ```

#### 2.1.3 Media Library Upload API

- **Endpoint**: `POST /api/admin/media/upload`

- **Description**: Upload images for products, cases, or blogs

- **Request Body**: Form-data

|    Field|    Type|    Requirements|
|---|---|---|
|    file|    File|    JPG/PNG/GIF, max 5MB|
|    type|    String|    Optional (default: "common"; "product"/"case"/"blog" for classification)|
- **Response**:

    ```json
    
    {
      "code": 200,
      "msg": "Upload successful",
      "data": {
        "url": "/static/uploads/media/20250618/pm-tool-cover.png",
        "fileName": "pm-tool-cover.png",
        "size": 124500 // File size in bytes
      },
      "timestamp": 1718765800000
    }
    ```

### 2.2 Public Website APIs (No Authentication)

#### 2.2.1 Public Data APIs

|Purpose|Endpoint|Method|Key Parameters|
|---|---|---|---|
|Get Public Product List|`/api/web/product/list`|GET|`page`, `size`, `category_id`|
|Get Product Detail|`/api/web/product/detail`|GET|`slug` (required, product slug)|
|Get Case Studies|`/api/web/case/list`|GET|`page`, `size`, `industry`|
|Get Latest Blogs|`/api/web/blog/latest`|GET|`limit` (default:5)|
|Submit Contact Form|`/api/web/contact/submit`|POST|`name`, `email`, `message` (required)|
**Example: Submit Contact Form**

- Request Body:

    ```json
    
    {
      "name": "John Doe",
      "email": "john@example.com",
      "phone": "+1234567890", // Optional
      "message": "Interested in your project management system. Please contact me.",
      "subject": "Product Inquiry" // Optional
    }
    ```

- Response:

    ```json
    
    {
      "code": 200,
      "msg": "Your message has been received. We will contact you soon!",
      "data": {},
      "timestamp": 1718767000000
    }
    ```

---

## 3. API Testing Guidelines

### 3.1 Testing Environment

- **Base URL (Development)**: `https://dev.admin.nankoyo.com/api`

- **Base URL (Production)**: `https://admin.nankoyo.com/api`

- **Test Accounts**:

    - Super Admin: `super_admin@nankoyo.com` (password: [Internal Team Password])

    - Content Admin: `content_admin@nankoyo.com` (password: [Internal Team Password])

### 3.2 Testing Tools

- Recommended: Postman, Insomnia, or curl commands

- Postman Collection: Import via [Internal Team Shared Link]

### 3.3 Testing Workflow

1. **Authenticate**: First call `POST /api/admin/login` to get Token

2. **Set Headers**: Add `Authorization: Bearer [Token]` to all management API requests

3. **Test Cases**:

    - Positive Test: Use valid parameters to verify successful responses

    - Negative Test: Test missing required fields, invalid formats, and permission restrictions

    - Boundary Test: Test maximum/minimum values (e.g., 100-character product title)

4. **Validation Criteria**:

    - Status code matches expected result (200 for success, 4xx for client errors)

    - Response `msg` is clear and accurate

    - `data` contains expected fields and values

    - Public APIs return consistent data with the website frontend

### 3.4 curl Example (Product List)

```bash

# Get product list (replace [TOKEN] with actual value)
curl -X GET "https://dev.admin.nankoyo.com/api/admin/product/list?page=1&size=10&category_id=3" \
  -H "Authorization: Bearer [TOKEN]" \
  -H "Content-Type: application/json"
```

---

## 4. Permission Control for APIs

### 4.1 Role Definitions

|Role|Permissible API Modules|Restrictions|
|---|---|---|
|Super Admin|All APIs (management + system settings)|No restrictions|
|Content Admin|Product, Case, Blog, Media APIs|Cannot access user management or system settings|
|HR Admin|Career, Resume APIs|Cannot modify products/cases/blogs|
|Viewer|Read-only APIs (list/detail)|Cannot create/update/delete any resources|
### 4.2 Permission Verification Logic

- APIs under `/api/admin/` check the `role` field in the Token

- Unauthorized requests return `code: 403` with `msg: "Insufficient permissions"`

- Example of 403 Response:

    ```json
    
    {
      "code": 403,
      "msg": "Insufficient permissions: cannot access system settings",
      "data": {},
      "timestamp": 1718768000000,
      "requestId": "uuid-abc1-23de-45fg-67hi"
    }
    ```

### 4.3 Role Assignment

- Only Super Admins can assign roles via the management system UI (`/admin/user/role`)

- No API for role assignment (to prevent unauthorized access)

---

## 5. Error Code Reference

### 5.1 Common Error Codes

|Code|Description|Handling Suggestion|
|---|---|---|
|200|Success|-|
|400|Invalid request parameters|Check parameter format/required fields|
|401|Unauthorized (invalid/missing Token)|Re-login to obtain valid Token|
|403|Insufficient permissions|Contact Super Admin for role adjustment|
|404|Resource not found|Verify resource ID/slug exists|
|409|Resource conflict|e.g., duplicate product slug; modify and retry|
|500|Server internal error|Check PHP error logs or contact developers|
### 5.2 Module-Specific Error Codes

|Module|Code|Description|
|---|---|---|
|Product|4001|Product title already exists|
|Product|4002|Invalid category ID|
|Media|4101|File size exceeds 5MB|
|Media|4102|Unsupported file format (only JPG/PNG/GIF)|
|Auth|4201|Incorrect username/password|
|Auth|4202|Account disabled|
---

## 6. API Change Log

|Version|Date|Changed By|Cha

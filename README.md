# API Documentation

This document provides detailed information about the API endpoints for the LinqUpEG application.

## Table of Contents

1.  [Authentication](#authentication)
    *   [POST /api/register](#post-apiregister)
    *   [POST /api/login](#post-apilogin)
    *   [POST /api/auth/google](#post-apiauthgoogle)
    *   [DELETE /api/deleteAccount](#delete-apideleteaccount)
2.  [User Data](#user-data)
    *   [GET /api/get/socials/:handle](#get-apigetsocialshandle)
    *   [GET /api/get/:handle](#get-apigethandle)
    *   [POST /api/data/dashboard](#post-apidatadashboard)
3.  [Save Items](#save-items)
    *   [POST /api/save/socials](#post-apisavesocials)
    *   [POST /api/save/profile](#post-apisaveprofile)
    *   [POST /api/save/links](#post-apisavelinks)
4.  [Load Items](#load-items)
    *   [POST /api/load/socials](#post-apiloadsocials)
    *   [POST /api/load/links](#post-apiloadlinks)
5.  [Link Resolution](#link-resolution)
    *   [POST /api/resolve-url](#post-apiresolve-url)
    *   [GET /api/l/:linkId](#get-apillinkid)
6.  [Password Management](#password-management)
    *   [POST /api/resetPassword](#post-apiresetpassword)
    *   [POST /api/validateToken](#post-apivalidatetoken)
    *   [POST /api/changePassword](#post-apichangepassword)
    *   [POST /api/user/update-password](#post-apiuserupdate-password)
7.  [Products](#products)
    *   [GET /api/get/products](#get-apigetproducts)
8.  [Analytics](#analytics)
    *   [POST /api/view/:handle](#post-apiviewhandle)
9.  [Member Management (Admin Only)](#member-management-admin-only)
    *   [POST /api/members/add](#post-apimembersadd)
    *   [GET /api/members/get](#get-apimembersget)
    *   [DELETE /api/members/remove](#post-apimembersremove)
    *   [POST /api/members/generate-invite](#post-apimembersgenerate-invite)
    *   [POST /api/members/send-invite](#post-apimemberssend-invite)
    *   [POST /api/members/toggle-activation](#post-apimemberstoggle-activation)

---

## Authentication

### POST /api/register

*   **Description:** Registers a new user. Can also handle member registration via an invite token.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Body:**
    ```json
    {
      "name": "Test User",
      "email": "test@example.com",
      "password": "Password123!",
      "handle": "testuser",
      "inviteToken": "optional_jwt_invite_token" // Optional: For registering as a member under an admin
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "message": "User created",
      "status": "success",
      "token": "jwt_token",
      "id": "user_id",
      "role": "ADMIN", // or "MEMBER" if registered via inviteToken
      "joinedAdmin": false // or true if registered via inviteToken and linked to an admin
    }
    ```
*   **Response (Error 400 - Bad Request):**
    *   If required fields are missing:
        ```json
        {
          "status": "error",
          "message": "Missing required fields"
        }
        ```
    *   If password format is invalid:
        ```json
        {
          "status": "error",
          "message": "Invalid password format."
        }
        ```
    *   If email or handle is a duplicate (Note: actual HTTP status might be 200 with error in body):
        ```json
        {
          "message": "Duplicate username or email. Try a different one.",
          "status": "error"
        }
        ```
*   **Response (Error 500 - Server Error):**
    ```json
    {
      "message": "Specific error message",
      "status": "error"
    }
    ```
*   **Logic:**
    1.  Validates input fields: `name`, `email`, `password`, `handle`.
    2.  Password must meet complexity: at least 6 characters, 1 lowercase, 1 uppercase, 1 digit, 1 special character.
    3.  If `inviteToken` is provided and valid:
        *   Verifies the JWT `inviteToken`.
        *   If token is a valid invite from an existing ADMIN, the new user's `role` is set to 'MEMBER' and `adminId` is linked.
        *   The new member's ID is added to the inviting admin's `members` array.
    4.  Otherwise, the user's `role` defaults to 'ADMIN'.
    5.  Checks if email or handle already exists in the database.
    6.  Hashes the password.
    7.  Creates a new user in the database with provided details, role, and a default link (`{ url: "https://linqupeg.com", title: "LINQ UP", icon: "/logoname.png" }`).
    8.  Generates a JWT token for the new user.
    9.  No email verification step is currently implemented in this endpoint.

---

### POST /api/login

*   **Description:** Logs in an existing user.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Body:**
    ```json
    {
      "email": "test@example.com",
      "password": "password123"
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "message": "User found",
      "status": "success",
      "token": "jwt_token",
      "id": "user_id"
    }
    ```
*   **Response (Error - Invalid Credentials, HTTP 200 with error in body):**
    ```json
    {
      "status": "not found",
      "error": "Invalid Credentials"
    }
    ```
*   **Response (Error - Other, HTTP 200 with error in body):**
    ```json
    {
      "message": "Specific error message",
      "status": "error"
    }
    ```
*   **Logic:**
    1.  Converts the provided email to lowercase.
    2.  Finds the user by email.
    3.  Compares the provided password with the stored hashed password.
    4.  If credentials are valid, generates a JWT token containing user ID and email.

---

### POST /api/auth/google

*   **Description:** Authenticates a user via Google OAuth using an authorization code. Can also handle member registration via an invite token.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Body:**
    ```json
    {
      "code": "google_authorization_code",
      "inviteToken": "optional_jwt_invite_token" // Optional: For registering/logging in as a member under an admin
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "token": "jwt_token",
      "id": "user_id",
      "role": "ADMIN", // or "MEMBER"
      "joinedAdmin": false // or true if linked to an admin
    }
    ```
*   **Response (Error 400):**
    *   If Google authentication fails (e.g., invalid code):
        ```json
        {
          "error": "Google authentication failed"
        }
        ```
    *   If the email is already registered with a password (local account):
        ```json
        {
          "error": "This email is registered with a password. Please use your email and password to log in."
        }
        ```
*   **Logic:**
    1.  Exchanges the Google authorization `code` for an `id_token` and `access_token` with Google servers.
    2.  Verifies the `id_token` using `google-auth-library`.
    3.  Extracts user information (email, name, picture) from the Google token.
    4.  If `inviteToken` is provided and valid:
        *   Verifies the JWT `inviteToken`.
        *   If token is a valid invite from an existing ADMIN, the user's `role` is set to 'MEMBER' and `adminId` is linked.
    5.  Checks if a user with the Google email already exists:
        *   If the user exists and their `authType` is 'local', returns an error instructing them to log in with their password.
        *   If the user exists and `authType` is 'google' (or not set), proceeds to log them in. If an `inviteToken` was processed and they weren't previously a member of that admin, this link might be established.
        *   If the user does not exist:
            *   A new user is created with the handle derived from the email prefix, Google email, name, and profile picture.
            *   `authType` is set to 'google'.
            *   A default link (`{ url: "https://linqupeg.com", title: "LINQ UP", icon: "/logoname.png" }`) is added.
            *   If an `inviteToken` was processed, their `role` is set to 'MEMBER', `adminId` is linked, and the admin's `members` array is updated.
            *   Otherwise, their `role` defaults to 'ADMIN'.
    6.  Generates a JWT token for the user.

---

### DELETE /api/deleteAccount

*   **Description:** Deletes the authenticated user's account. **Note:** This endpoint currently requires email and password in the body for re-authentication, despite the presence of JWT-based authentication via middleware.
*   **Authentication:** Required (JWT Bearer Token via `authenticateUser` middleware).
*   **Access Control:** User themselves (intended, but re-authentication logic is key).
*   **Request Body:**
    ```json
    {
      "email": "user@example.com",
      "password": "currentPassword123"
    }
    ```
*   **Response (Success 204 - No Content):**
    *   Body (Note: 204 responses should ideally not have a body, but it's sent):
    ```json
    {
      "message": "User deleted",
      "status": "success"
    }
    ```
*   **Response (Error 400 - Bad Request):**
    ```json
    {
      "status": "error",
      "error": "Email and password are required"
    }
    ```
*   **Response (Error 401 - Unauthorized):**
    ```json
    {
      "status": "not found", // Note: status says "not found" but implies unauthorized
      "error": "Invalid Credentials"
    }
    ```
*   **Response (Error 404 - Not Found):**
    ```json
    {
      "status": "not found",
      "error": "User not found"
    }
    ```
*   **Response (Error 500 - Server Error):**
    ```json
    {
      "message": "Specific error message",
      "status": "error"
    }
    ```
*   **Logic:**
    1.  The `authenticateUser` middleware verifies the JWT and attaches `req.user.id`.
    2.  The controller then requires `email` and `password` in the request body.
    3.  It finds the user by the provided `email` (not by `req.user.id` from JWT).
    4.  It verifies the provided `password` against the stored hash for the user found by email.
    5.  If email and password are correct, the user account associated with that email is deleted.

---

## User Data

### GET /api/get/socials/:handle

*   **Description:** Retrieves social media links for a user by their handle.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Params:**
    *   `handle`: The user's unique handle.
*   **Response (Success 200):**
    ```json
    {
      "message": "found",
      "socials": {
        "facebook": "https://facebook.com/user",
        "twitter": "https://twitter.com/user",
        "linkedin": "https://linkedin.com/in/user",
        "instagram": "https://instagram.com/user",
        "youtube": "https://youtube.com/user",
        "tiktok": "https://tiktok.com/@user",
        "github": "https://github.com/user"
      },
      "status": "success"
    }
    ```
*   **Response (Error - User not found, HTTP 200 with error in body):**
    ```json
    {
      "status": "error",
      "error": "User not found or user has no social media data"
    }
    ```
*   **Response (Error - Other, HTTP 200 with error in body):**
    ```json
    {
      "status": "error",
      "error": "Specific error message"
    }
    ```
*   **Logic:**
    1.  Finds the user by their `handle`.
    2.  Returns the `socialMedia` object from the user document.

---

### GET /api/get/:handle

*   **Description:** Retrieves public user data (profile details, links, etc.) for a given handle.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Params:**
    *   `handle`: The user's unique handle.
*   **Response (Success 200):**
    ```json
    {
      "message": "found",
      "userData": {
        "name": "Test User",
        "bio": "User bio",
        "avatar": "url_to_image.png",
        "background": "url_to_background_image.png",
        "logo": "url_to_logo_image.png",
        "handle": "testuser",
        "role": "ADMIN", // or "MEMBER"
        "phoneNumber": "1234567890",
        "secondaryPhoneNumber": "0987654321",
        "whatsapp": "1234567890",
        "userEmail": "test@example.com",
        "links": [
          { "url": "https://example.com", "title": "My Website", "icon": "icon_url.png", "type": "link", "clicks": 0 }
        ],
        "title": "User Title",
        "views": 150
      },
      "socials": {
        "facebook": "https://facebook.com/user",
        "twitter": "https://twitter.com/user"
        // ... other social platforms
      },
      "status": "success"
    }
    ```
*   **Response (Error 404 - Not Found):**
    ```json
    {
      "status": "error",
      "message": "User not found or profile has been deactivated"
    }
    ```
*   **Response (Error - Other, HTTP 200 with error in body):**
    ```json
    {
      "status": "error",
      "error": "Specific error message"
    }
    ```
*   **Logic:**
    1.  Finds the user by their `handle`. The user must also not have `activated` set to `false`.
    2.  Selects and returns a range of user fields: `name`, `bio`, `avatar` (defaults if not set), `background`, `logo`, `handle`, `role`, `phoneNumber`, `secondaryPhoneNumber`, `whatsapp`, `userEmail` (which is the primary `email`), `links`, `title`, and `views`.
    3.  Also returns the `socialMedia` object.

---

### POST /api/data/dashboard

*   **Description:** Retrieves data for the authenticated user's dashboard.
*   **Authentication:** Required (JWT Bearer Token via `authenticateUser` middleware).
*   **Access Control:** User themselves.
*   **Request Body:** None (expects token in header).
*   **Response (Success 200):**
    ```json
    {
      "userData": {
        "name": "Test User",
        "role": "ADMIN", // or "MEMBER"
        "bio": "User bio",
        "avatar": "url_to_image.png",
        "background": "url_to_background_image.png",
        "logo": "url_to_logo_image.png",
        "handle": "testuser",
        "phoneNumber": "1234567890",
        "secondaryPhoneNumber": "0987654321",
        "whatsapp": "1234567890",
        "userEmail": "test@example.com",
        "linkCount": 5,
        "pdfCount": 2,
        "actualLinks": [
          { "url": "https://example.com", "title": "My Website", "icon": "icon_url.png", "type": "link", "clicks": 0 },
          { "url": "path/to/file.pdf", "title": "My PDF", "icon": "pdf_icon_url.png", "type": "pdf", "clicks": 0 }
        ],
        "views": 150,
        "memberCount": 3 // Only present if role is ADMIN
      },
      "status": "Okay"
    }
    ```
*   **Response (Error 404 - Not Found):**
    ```json
    {
      "status": "error",
      "error": "User not found"
    }
    ```
*   **Response (Error - Other, HTTP 200 with error in body):**
    ```json
    {
      "status": "error",
      "error": "Specific error message"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware to get authenticated user details (`req.user`).
    2.  Finds the user in the database by the email from `req.user.email`.
    3.  Counts the number of links with `type: 'pdf'` and `type: 'link'` separately.
    4.  Returns a comprehensive set of user data including: `name`, `role`, `bio`, `avatar`, `background`, `logo`, `handle`, contact details (`phoneNumber`, `secondaryPhoneNumber`, `whatsapp`, `userEmail`), counts for links and PDFs (`linkCount`, `pdfCount`), the full `actualLinks` array, `views`, and `memberCount` (if the user is an ADMIN, this is the length of their `members` array).

---

## Save Items

These endpoints require authentication. The user ID is extracted from the JWT token.

### POST /api/save/socials

*   **Description:** Saves or updates the authenticated user's social media links.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** User themselves.
*   **Request Body:**
    ```json
    {
      "socials": [
        { "platform": "twitter", "url": "https://twitter.com/newuser", "isActive": true },
        { "platform": "linkedin", "url": "https://linkedin.com/in/newuser", "isActive": false }
      ]
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Socials saved successfully",
      "socials": [
        { "platform": "twitter", "url": "https://twitter.com/newuser", "isActive": true },
        { "platform": "linkedin", "url": "https://linkedin.com/in/newuser", "isActive": false }
      ]
    }
    ```
*   **Response (Error 400/404/500):**
    ```json
    {
      "status": "error",
      "message": "Invalid data" // or "User not found" or "Failed to save socials"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware.
    2.  Validates the incoming `socials` array.
    3.  Updates the `socials` field for the authenticated user in the database.

---

### POST /api/save/profile

*   **Description:** Saves or updates the authenticated user's profile information (name, bio, profile picture).
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** User themselves.
*   **Request Body:**
    ```json
    {
      "name": "Updated Name",
      "bio": "Updated bio.",
      "profilePicture": "new_image_url.jpg", // Optional
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Profile saved successfully",
      "profile": {
        "name": "Updated Name",
        "bio": "Updated bio.",
        "profilePicture": "new_image_url.jpg",
      }
    }
    ```
*   **Response (Error 400/404/500):**
    ```json
    {
      "status": "error",
      "message": "Invalid data" // or "User not found" or "Failed to save profile"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware.
    2.  Updates the `name`, `bio`, `profilePicture` fields for the authenticated user.

---

### POST /api/save/links

*   **Description:** Saves or updates the authenticated user's shareable links.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** User themselves.
*   **Request Body:**
    ```json
    {
      "links": [
        { "title": "My Portfolio", "url": "https://portfolio.example.com", "isActive": true, "type": "link" },
        { "title": "Blog Post", "url": "https://blog.example.com/post1", "isActive": false, "type": "link" }
      ]
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Links saved successfully",
      "links": [
        { "title": "My Portfolio", "url": "https://portfolio.example.com", "isActive": true, "type": "link" },
        { "title": "Blog Post", "url": "https://blog.example.com/post1", "isActive": false, "type": "link" }
      ]
    }
    ```
*   **Response (Error 400/404/500):**
    ```json
    {
      "status": "error",
      "message": "Invalid data" // or "User not found" or "Failed to save links"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware.
    2.  Validates the incoming `links` array.
    3.  Updates the `links` field for the authenticated user. Each link object is expected to have `id`, `title`, `url`, `isActive`, and `type`.

---

## Load Items

These endpoints require authentication and are used to load existing data for the authenticated user, typically for populating editing interfaces.

### POST /api/load/socials

*   **Description:** Loads the authenticated user's saved social media links.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** User themselves.
*   **Request Body:** None.
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "socials": [
        { "platform": "twitter", "url": "https://twitter.com/user", "isActive": true }
      ]
    }
    ```
*   **Response (Error 404/500):**
    ```json
    {
      "status": "error",
      "message": "User not found" // or "Failed to load socials"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware.
    2.  Finds the user and returns their `socials` array.

---

### POST /api/load/links

*   **Description:** Loads the authenticated user's saved shareable links.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** User themselves.
*   **Request Body:** None.
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "links": [
        { "id": "link_id_1", "title": "My Website", "url": "https://example.com", "isActive": true, "type": "link" }
      ]
    }
    ```
*   **Response (Error 404/500):**
    ```json
    {
      "status": "error",
      "message": "User not found" // or "Failed to load links"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware.
    2.  Finds the user and returns their `links` array.

---

## Link Resolution

### POST /api/resolve-url

*   **Description:** Fetches metadata (like title and favicon) for a given URL. Used for enhancing link previews.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** Authenticated users.
*   **Request Body:**
    ```json
    {
      "url": "https://example.com"
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "metadata": {
        "title": "Example Domain",
        "favicon": "https://www.iana.org/_img/2022/iana_logo_black.svg" // or null
      }
    }
    ```
*   **Response (Error 400/500):**
    ```json
    {
      "status": "error",
      "message": "Invalid URL" // or "Failed to resolve URL"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware.
    2.  Takes a URL as input.
    3.  Fetches the HTML content of the URL.
    4.  Parses the HTML to extract the page title and favicon URL.
    5.  Returns the extracted metadata.

---

### GET /api/l/:linkId

*   **Description:** Redirects a short link ID to its original URL and tracks the click.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Params:**
    *   `linkId`: The unique ID of the link to redirect.
*   **Response:**
    *   **Success (302 Redirect):** Redirects to the original URL.
    *   **Error (404 Not Found):** If the link ID is not found or the link is inactive.
        ```json
        { "status": "error", "message": "Link not found or inactive" }
        ```
*   **Logic:**
    1.  Finds a user who has a link with the given `linkId` in their `links` array.
    2.  Ensures the link is active.
    3.  If found and active, increments the click count for that specific link and the user's total link clicks.
    4.  Redirects the client to the link's original URL.

---

## Password Management

### POST /api/resetPassword

*   **Description:** Initiates the password reset process for a user by sending a reset token to their email.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Body:**
    ```json
    {
      "email": "user@example.com"
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Password reset email sent. Please check your inbox."
    }
    ```
*   **Response (Error 400/404/500):**
    ```json
    {
      "status": "error",
      "message": "Email not provided" // or "User not found" or "Error sending email"
    }
    ```
*   **Logic:**
    1.  Finds the user by email.
    2.  Generates a unique password reset token and sets an expiry time.
    3.  Saves the token and expiry to the user document.
    4.  Sends an email to the user containing a link with the reset token (using `ResetPasswordEmail` template and `sendEmail` utility).

---

### POST /api/validateToken

*   **Description:** Validates a password reset token.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Body:**
    ```json
    {
      "token": "reset_token_string"
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Token is valid"
    }
    ```
*   **Response (Error 400):**
    ```json
    {
      "status": "error",
      "message": "Invalid or expired token"
    }
    ```
*   **Logic:**
    1.  Finds a user whose `resetPasswordToken` matches the provided token and whose `resetPasswordExpires` is in the future.

---

### POST /api/changePassword

*   **Description:** Changes the user's password using a valid reset token. This is for the unauthenticated password reset flow.
*   **Authentication:** Not required (relies on valid token).
*   **Access Control:** Public (with valid token).
*   **Request Body:**
    ```json
    {
      "token": "valid_reset_token_string",
      "newPassword": "newSecurePassword123"
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Password has been changed successfully."
    }
    ```
*   **Response (Error 400):**
    ```json
    {
      "status": "error",
      "message": "Invalid or expired token" // or "Password update failed"
    }
    ```
*   **Logic:**
    1.  Finds the user by the `resetPasswordToken` and checks if the token is still valid (not expired).
    2.  Hashes the `newPassword`.
    3.  Updates the user's password with the new hashed password.
    4.  Clears the `resetPasswordToken` and `resetPasswordExpires` fields from the user document.

---

### POST /api/user/update-password

*   **Description:** Allows an authenticated user to change their own password.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** User themselves.
*   **Request Body:**
    ```json
    {
      "currentPassword": "oldPassword123",
      "newPassword": "newSecurePassword456"
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Password updated successfully"
    }
    ```
*   **Response (Error 400/401/500):**
    ```json
    {
      "status": "error",
      "message": "Incorrect current password" // or "User not found" or "Failed to update password"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` middleware.
    2.  Retrieves the authenticated user.
    3.  Verifies `currentPassword` against the stored hashed password.
    4.  Hashes `newPassword`.
    5.  Updates the user's password in the database.

---

## Products

### GET /api/get/products

*   **Description:** Retrieves a list of available products or services (e.g., subscription plans).
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "products": [
        { "id": "prod_1", "name": "Basic Plan", "price": 0, "features": ["Feature A", "Feature B"] },
        { "id": "prod_2", "name": "Pro Plan", "price": 10, "features": ["Feature A", "Feature B", "Feature C"] }
      ]
    }
    ```
*   **Response (Error 500):**
    ```json
    {
      "status": "error",
      "message": "Failed to retrieve products"
    }
    ```
*   **Logic:**
    1.  Currently returns a hardcoded list of products. In a real application, this would fetch from a database or a payment provider like Stripe.

---

## Analytics

### POST /api/view/:handle

*   **Description:** Increments the profile view count for a user.
*   **Authentication:** Not required.
*   **Access Control:** Public.
*   **Request Params:**
    *   `handle`: The user's unique handle whose profile was viewed.
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Profile view tracked"
    }
    ```
*   **Response (Error 404/500):**
    ```json
    {
      "status": "error",
      "message": "User not found" // or "Failed to track profile view"
    }
    ```
*   **Logic:**
    1.  Finds the user by `handle`.
    2.  Increments the `profileViews` count for the user.

---

## Member Management (Admin Only)

These endpoints require the authenticated user to have an 'admin' role.

### POST /api/members/add

*   **Description:** Adds a new member to the admin's team/organization.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** Admin only.
*   **Request Body:**
    ```json
    {
      "name": "New Member",
      "email": "member@example.com",
      "role": "editor" // e.g., editor, viewer
    }
    ```
*   **Response (Success 201):**
    ```json
    {
      "status": "success",
      "message": "Member added successfully. Invitation sent.",
      "member": { /* member details */ }
    }
    ```
*   **Response (Error 400/403/404/500):**
    ```json
    {
      "status": "error",
      "message": "User with this email already exists as a member or primary account" // or "Admin not found" or "Failed to add member"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` and `adminOnly` middleware.
    2.  Checks if an account with the given email already exists.
    3.  Creates a new user document for the member with a temporary password and specified role.
    4.  Links the member to the admin (e.g., by storing `adminId` in the member's document or adding to an admin's `members` array).
    5.  Sends an invitation email (likely with a link to set their password).

---

### GET /api/members/get

*   **Description:** Retrieves a list of members associated with the authenticated admin.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** Admin only.
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "members": [
        { "id": "member_id_1", "name": "Member One", "email": "member1@example.com", "role": "editor", "isActive": true, "addedOn": "timestamp" },
        { "id": "member_id_2", "name": "Member Two", "email": "member2@example.com", "role": "viewer", "isActive": false, "addedOn": "timestamp" }
      ]
    }
    ```
*   **Response (Error 403/404/500):**
    ```json
    {
      "status": "error",
      "message": "Admin not found" // or "Failed to retrieve members"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` and `adminOnly` middleware.
    2.  Finds all user documents where `adminId` matches the authenticated admin's ID.
    3.  Returns relevant details for each member.

---

### DELETE /api/members/remove

*   **Description:** Removes a member from the admin's team.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** Admin only.
*   **Request Body:**
    ```json
    {
      "memberId": "member_id_to_remove"
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Member removed successfully"
    }
    ```
*   **Response (Error 403/404/500):**
    ```json
    {
      "status": "error",
      "message": "Member not found or not associated with this admin" // or "Failed to remove member"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` and `adminOnly` middleware.
    2.  Verifies that the `memberId` belongs to a member managed by the authenticated admin.
    3.  Deletes the member's user document or disassociates them from the admin.

---

### POST /api/members/generate-invite

*   **Description:** Generates an invitation link for a new member.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** Admin only.
*   **Request Body:**
    ```json
    {
      "email": "newmember@example.com",
      "role": "editor" // e.g., editor, viewer
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "inviteLink": "https://app.linqupeg.com/invite?token=invite_token_string"
    }
    ```
*   **Response (Error 400/403/500):**
    ```json
    {
      "status": "error",
      "message": "Email is required" // or "Failed to generate invite link"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` and `adminOnly` middleware.
    2.  Generates a unique invitation token, potentially storing it with the admin's ID, target email, and role.
    3.  Constructs an invitation URL containing this token.

---

### POST /api/members/send-invite

*   **Description:** Sends an invitation email to a potential new member.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** Admin only.
*   **Request Body:**
    ```json
    {
      "email": "potentialmember@example.com",
      "name": "Potential Member Name", // Optional, for personalizing email
      "role": "viewer" // Role to be assigned upon acceptance
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Invitation email sent successfully to potentialmember@example.com"
    }
    ```
*   **Response (Error 400/403/500):**
    ```json
    {
      "status": "error",
      "message": "Email is required" // or "Failed to send invite" or "Error sending email"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` and `adminOnly` middleware.
    2.  Generates an invitation token (similar to `/generate-invite` or reuses its logic).
    3.  Stores the pending invitation details (email, role, token, adminId).
    4.  Constructs an invitation link.
    5.  Sends an email using `InviteMemberEmail` template and `sendEmail` utility, including the admin's name and the invitation link.

---

### POST /api/members/toggle-activation

*   **Description:** Activates or deactivates a member account associated with the admin.
*   **Authentication:** Required (JWT Bearer Token).
*   **Access Control:** Admin only.
*   **Request Body:**
    ```json
    {
      "memberId": "member_id_to_toggle",
      "isActive": true // or false
    }
    ```
*   **Response (Success 200):**
    ```json
    {
      "status": "success",
      "message": "Member activation status updated successfully.",
      "member": { /* updated member details, including new isActive status */ }
    }
    ```
*   **Response (Error 400/403/404/500):**
    ```json
    {
      "status": "error",
      "message": "MemberId and isActive status are required" // or "Member not found" or "Failed to update member status"
    }
    ```
*   **Logic:**
    1.  Uses `authenticateUser` and `adminOnly` middleware.
    2.  Finds the member by `memberId`, ensuring they are managed by the authenticated admin.
    3.  Updates the `isActive` status of the member in the database.

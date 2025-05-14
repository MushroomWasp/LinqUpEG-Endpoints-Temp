# LinqUp API Documentation

This document provides an overview of the available endpoints in the LinqUp API.

`baseUrl`: `https://linqupeg.com/api/`

## Authentication

### Register User
**POST** `/register`
- **Request Body**
  ```json
  {
    "name": "John Doe",
    "handle": "johndoe",
    "email": "john@example.com",
    "password": "Password1!"
  }
  ```
- **Response**
  ```json
  {
    "message": "User created",
    "status": "success",
    "token": "jwt_token_here",
    "id": "user_id_here"
  }
  ```

### Login User
**POST** `/login`
- **Request Body**
  ```json
  {
    "email": "john@example.com",
    "password": "Password1!"
  }
  ```
- **Response**
  ```json
  {
    "message": "User found",
    "status": "success",
    "token": "jwt_token_here",
    "id": "user_id_here"
  }
  ```

### Google Authentication
**POST** `/auth/google`
- **Request Body**
  ```json
  {
    "code": "google_auth_code"
  }
  ```
- **Response**
  ```json
  {
    "status": "success",
    "token": "jwt_token_here",
    "id": "user_id_here"
  }
  ```

### Delete Account
**DELETE** `/deleteAccount`
- **Request Body**
  ```json
  {
    "email": "john@example.com",
    "password": "Password1!"
  }
  ```
- **Response**
  ```json
  {
    "message": "User deleted",
    "status": "success"
  }
  ```

## Password Management

### Reset Password
**POST** `/resetPassword`
- **Request Body**
  ```json
  {
    "email": "john@example.com"
  }
  ```
- **Response**
  ```json
  {
    "message": "success"
  }
  ```

### Validate Token
**POST** `/validateToken`
- **Request Body**
  ```json
  {
    "token": "reset_token_here"
  }
  ```
- **Response**
  ```json
  {
    "message": "success",
    "user": { /* user details */ }
  }
  ```

### Change Password
**POST** `/changePassword`
- **Request Body**
  ```json
  {
    "resetPasswordToken": "reset_token_here",
    "password": "NewPassword1!"
  }
  ```
- **Response**
  ```json
  {
    "message": "Password changed successfully"
  }
  ```

## User Data

### Get User Data
**GET** `/get/:handle`
- **Response**
  ```json
  {
    "message": "found",
    "userData": {
      "name": "John Doe",
      "bio": "User bio",
      "avatar": "avatar_url",
      "handle": "johndoe",
      "links": [/* array of links */]
    },
    "socials": {/* social media links */},
    "status": "success"
  }
  ```

### Get User Socials
**GET** `/get/socials/:handle`
- **Response**
  ```json
  {
    "message": "found",
    "socials": {/* social media links */},
    "status": "success"
  }
  ```

### Get Dashboard Data

**POST** `/data/dashboard`

#### Request Body
```json
{
  "tokenMail": "jwt_token_here"
}
````

#### Response

```json
{
  "userData": {
    "name": "John Doe",
    "role": "Developer",
    "bio": "Loves building cool stuff.",
    "avatar": "https://cdn-icons-png.flaticon.com/128/4140/4140048.png",
    "background": "",
    "logo": "",
    "handle": "@johndoe",
    "phoneNumber": "+1234567890",
    "secondaryPhoneNumber": "",
    "whatsapp": "+1234567890",
    "userEmail": "john@example.com",
    "links": 3,
    "pdfs": 1
  },
  "status": "Okay"
}
```

## Data Management

### Save Profile
**POST** `/save/profile`
- **Request Body**
  ```json
  {
    "tokenMail": "jwt_token_here",
    "name": "John Doe",
    "bio": "Updated bio",
    "title": "Developer",
    "company": "LinqUp",
    "avatar": "avatar_url",
    "background": "background_url",
    "logo": "logo_url",
    "phoneNumber": "+123456789",
    "secondaryPhoneNumber": "+987654321",
    "whatsapp": "+123456789",
    "userEmail": "contact@example.com"
  }
  ```
- **Response**
  ```json
  {
    "message": "saved",
    "status": "success"
  }
  ```

### Save Socials
**POST** `/save/socials`
- **Request Body**
  ```json
  {
    "tokenMail": "jwt_token_here",
    "socials": {
      "facebook": "facebook_url",
      "instagram": "instagram_url",
      "twitter": "twitter_url"
    }
  }
  ```
- **Response**
  ```json
  {
    "message": "saved",
    "status": "success"
  }
  ```

### Save Links
**POST** `/save/links`
- **Request Body**
  ```json
  {
    "tokenMail": "jwt_token_here",
    "links": [
      {
        "url": "https://example.com",
        "title": "My Website",
        "icon": "icon_url",
        "type": "link"
      }
    ]
  }
  ```
- **Response**
  ```json
  {
    "message": "saved",
    "status": "success"
  }
  ```

### Load Socials
**POST** `/load/socials`
- **Request Body**
  ```json
  {
    "tokenMail": "jwt_token_here"
  }
  ```
- **Response**
  ```json
  {
    "message": "found",
    "socials": {/* social media links */},
    "status": "success"
  }
  ```

### Load Links
**POST** `/load/links`
- **Request Body**
  ```json
  {
    "tokenMail": "jwt_token_here"
  }
  ```
- **Response**
  ```json
  {
    "message": "found",
    "links": [/* array of links */],
    "status": "success"
  }
  ```

## Utilities

### Resolve URL
**POST** `/resolve-url`
- **Request Body**
  ```json
  {
    "url": "https://shortened-url.com/abc"
  }
  ```
- **Response**
  ```json
  {
    "resolvedUrl": "https://original-url.com/image.jpg"
  }
  ```

### Get Products
**GET** `/get/products`
- **Response**
  ```json
  {
    "data": {
      "products": {
        "edges": [/* array of product edges */]
      }
    }
  }
  ```

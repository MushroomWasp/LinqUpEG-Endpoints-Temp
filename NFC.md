# NFC Backend API Documentation

## Base URL
All API endpoints are relative to the base URL of your server.

## Authentication
Most endpoints require authentication using a JWT token. Include the token in the Authorization header:
```
Authorization: Bearer <your_token>
```

## API Endpoints

### Authentication (`/Auth`)

#### Login
- **POST** `/Auth/login`
- **Request Body:**
  ```json
  {
    "email": "string (required)",
    "password": "string (required)",
    "NotificationToken": "string (optional)"
  }
  ```

#### Register
- **POST** `/Auth/register`
- **Request Body:**
  ```json
  {
    "email": "string (required)",
    "handle": "string (required)",
    "category": "string (required)",
    "purchaseCode": "string (required)",
    "password": "string (required)"
  }
  ```

#### Find User (Admin Only)
- **GET** `/Auth/FindUser/:id`
- **Headers:** Requires Admin Authentication
- **Parameters:** `id` - User ID

#### Get All Users (Admin Only)
- **GET** `/Auth/GetAllUsers`
- **Headers:** Requires Admin Authentication

#### Add Notification Token
- **POST** `/Auth/NotificationToken/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Search User
- **POST** `/Auth/getUser/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID
- **Request Body:**
  ```json
  {
    "name": "string (required)"
  }
  ```

### Profile (`/Profile`)

#### Get Profile Data
- **GET** `/Profile/profiledata/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - User ID

#### Update Profile
- **PATCH** `/Profile/UpdateProfile/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - User ID

#### Update Profile Image
- **PATCH** `/Profile/UpdateProfileImage/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - User ID
- **Form Data:** `ProfileImage` - Image file

#### Send Friend Request
- **POST** `/Profile/SendRequest/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - Target User ID

#### Get Friend Requests
- **GET** `/Profile/GetRequests/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Accept Friend Request
- **POST** `/Profile/AcceptFriendRequest`
- **Headers:** Requires Authentication

#### Reject Friend Request
- **DELETE** `/Profile/RemoveRequest/User/:userId/Request/:requesId`
- **Headers:** Requires Authentication
- **Parameters:** 
  - `userId` - User ID
  - `requesId` - Request ID

#### Delete Friend
- **DELETE** `/Profile/RemoveFriend/User/:userId/Friend/:FriendId`
- **Headers:** Requires Authentication
- **Parameters:**
  - `userId` - User ID
  - `FriendId` - Friend ID

#### Get All Friends
- **GET** `/Profile/GetFriends/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Add Custom Link
- **POST** `/Profile/AddLink/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID
- **Form Data:**
  - `Social` - Icon file
  - `url` - Link URL (required)
  - `title` - Link title (required)

#### Delete Custom Link
- **DELETE** `/Profile/DeleteLink/:userId/SocialIndex/:linkIndex`
- **Headers:** Requires Authentication
- **Parameters:**
  - `userId` - User ID
  - `linkIndex` - Link index to delete

#### Update Custom Link Text
- **PATCH** `/Profile/UpdateLink/:userId/SocialIndex/:linkIndex`
- **Headers:** Requires Authentication
- **Parameters:**
  - `userId` - User ID
  - `linkIndex` - Link index to update

#### Update Custom Link Icon
- **PATCH** `/Profile/UpdateImage/:userId/SocialIndex/:linkIndex`
- **Headers:** Requires Authentication
- **Parameters:**
  - `userId` - User ID
  - `linkIndex` - Link index to update
- **Form Data:** `Social` - New icon file

#### Add Social Link
- **POST** `/Profile/AddSocial/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Update Social Link
- **PATCH** `/Profile/UpdateSocial/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Delete Social Link
- **DELETE** `/Profile/DeleteSocial/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Add Health Status
- **POST** `/Profile/AddHealth/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Update Health Status
- **PATCH** `/Profile/UpdateHealth/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Delete Health Status
- **DELETE** `/Profile/DeleteHealth/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

### Chat (`/chat`)

#### Create Chat
- **POST** `/chat/StartChating`
- **Headers:** Requires Authentication
- **Request Body:** (Validated by CreateChatValidation)

#### Get User Chats
- **GET** `/chat/UserChats/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - User ID

#### Get User Specific Chat
- **POST** `/chat/UserOneChat`
- **Headers:** Requires Authentication

#### Delete Chat
- **DELETE** `/chat/DeleteChat/User/:id/Chat/:chatId`
- **Headers:** Requires Authentication
- **Parameters:**
  - `id` - User ID
  - `chatId` - Chat ID

### Messages (`/Messages`)

#### Create Message
- **POST** `/Messages/CreateMessage`
- **Headers:** Requires Authentication
- **Request Body:** (Validated by SendMessageSchemaValidation)

#### Get All Messages
- **GET** `/Messages/AllMessages/:chatId`
- **Headers:** Requires Authentication
- **Parameters:** `chatId` - Chat ID

#### Mark Message as Seen
- **PATCH** `/Messages/Seen/:chatId/:senderId`
- **Headers:** Requires Authentication
- **Parameters:**
  - `chatId` - Chat ID
  - `senderId` - Sender ID

#### Send Image
- **POST** `/Messages/SendImage`
- **Headers:** Requires Authentication
- **Form Data:** `ChatImage` - Image file

### Analysis (`/Analysis`)

#### Add Tab View
- **POST** `/Analysis/LinkTap/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Get Tabs by Time
- **POST** `/Analysis/TabTime/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Get Views by Time
- **POST** `/Analysis/ViewTime/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Add View
- **POST** `/Analysis/AddView/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Add Social Tab
- **POST** `/Analysis/SocialTab/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

#### Get Social Tabs by Time
- **POST** `/Analysis/SocialTime/:userId`
- **Headers:** Requires Authentication
- **Parameters:** `userId` - User ID

### Notifications (`/Notification`)

#### Get User Notifications
- **GET** `/Notification/getAll/:recipientId`
- **Headers:** Requires Authentication
- **Parameters:** `recipientId` - Recipient User ID

#### Create Notification
- **POST** `/Notification/addNotification`
- **Request Body:** (Validated by CreateNotificationValidation)

#### Delete Selected Notifications
- **DELETE** `/Notification/DeleteSelected/:recipientId`
- **Headers:** Requires Authentication
- **Parameters:** `recipientId` - Recipient User ID

#### Clear All Notifications
- **DELETE** `/Notification/Clearselected/:recipientId`
- **Headers:** Requires Authentication
- **Parameters:** `recipientId` - Recipient User ID

#### Read All Notifications
- **PATCH** `/Notification/ReadAllNotification/:recipientId`
- **Headers:** Requires Authentication
- **Parameters:** `recipientId` - Recipient User ID

#### Read Single Notification
- **PATCH** `/Notification/ReadNotification/:recipientId`
- **Headers:** Requires Authentication
- **Parameters:** `recipientId` - Recipient User ID

#### Get Notification Count
- **GET** `/Notification/GetNotification/:recipientId`
- **Headers:** Requires Authentication
- **Parameters:** `recipientId` - Recipient User ID

### Products (`/Products`)

#### Create Product
- **POST** `/Products/CreateProduct`
- **Headers:** Requires Authentication
- **Form Data:**
  - `Products` - Product image file
  - Request body validated by createProductValidation

#### Delete Product
- **DELETE** `/Products/DeleteProduct/:productId`
- **Headers:** Requires Authentication
- **Parameters:** `productId` - Product ID

#### Update Product
- **PATCH** `/Products/UpdateProduct/:productId`
- **Headers:** Requires Authentication
- **Parameters:** `productId` - Product ID

#### Update Product Image
- **PATCH** `/Products/UpdateProductImage/:productId`
- **Headers:** Requires Authentication
- **Parameters:** `productId` - Product ID
- **Form Data:** `Products` - New product image file

#### Get All Products
- **GET** `/Products/getProducts`
- **Headers:** Requires Authentication

#### Get One Product
- **GET** `/Products/getOneProduct/:productId`
- **Headers:** Requires Authentication
- **Parameters:** `productId` - Product ID

#### Activate Products
- **POST** `/Products/addActiveProducts/users/:userId/Product/:ProductId`
- **Headers:** Requires Authentication
- **Parameters:**
  - `userId` - User ID
  - `ProductId` - Product ID

#### Remove Activated Products
- **DELETE** `/Products/removeActiveProducts/users/:userId/Product/:ProductId`
- **Headers:** Requires Authentication
- **Parameters:**
  - `userId` - User ID
  - `ProductId` - Product ID

### Serials (`/Serials`)

#### Add Serial Code
- **POST** `/Serials/AddSerial`
- **Headers:** Requires Authentication
- **Request Body:** (Validated by CreateCustomLinkValidation)

#### Delete Serial Code
- **DELETE** `/Serials/DeleteCode/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - Serial ID

#### Update Serial Code
- **PATCH** `/Serials/UpdateSerial/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - Serial ID

#### Get Specific Serial
- **GET** `/Serials/GetSerial/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - Serial ID

#### Get All Serials
- **GET** `/Serials/GetSerials`
- **Headers:** Requires Authentication

#### Activate Card
- **POST** `/Serials/Activiate`
- **Headers:** Requires Authentication

#### Change Sold Status
- **PATCH** `/Serials/TurnToSold/:id`
- **Headers:** Requires Authentication
- **Parameters:** `id` - Serial ID

### Blocks (`/Blocks`)

#### Block User
- **POST** `/Blocks/BlockUser`
- **Headers:** Requires Authentication

#### Get Blocked Users
- **GET** `/Blocks/GetBlockedUsers`
- **Headers:** Requires Authentication

#### Remove User from Block List
- **DELETE** `/Blocks/RemoveUserFromBlockList/:BlockedUserId`
- **Headers:** Requires Authentication
- **Parameters:** `BlockedUserId` - ID of the blocked user

## Error Handling
All endpoints return errors in the following format:
```json
{
  "success": false,
  "message": "Error message",
  "stack": "Error stack trace (only in development)"
}
```

## File Upload
For endpoints that require file uploads:
- Files are stored in the `uploads` directory
- Profile images are stored in the `uploads/icons` directory
- Maximum file size and allowed file types are configured in the server settings 

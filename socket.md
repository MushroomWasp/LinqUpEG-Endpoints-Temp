# Socket API Documentation

This document outlines the Socket.IO events used in the application.

## Server-Side Events (Listening)

### `connection`
- **Description**: Triggered when a new client connects to the socket server.
- **Payload**: None.
- **Internal Actions**:
    - Logs the connection of the new socket ID.
    - Sets up listeners for other events from this client.

### `OpenApp`
- **Description**: Triggered when a user opens the application. Registers the user as online and notifies them of their online friends.
- **Payload**: `{ userId: string }`
    - `userId`: The ID of the user opening the app.
- **Emits**:
    - `GetOnlineFriends` to the connected client (`socket.id`):
        - **Payload**: `OnlineFriendForUser: Array<{ id: string, ...otherFriendData }>` - An array of online friends for the user.
- **Internal Actions**:
    - Adds the user to the `OnlineUsers` map with their socket ID.
    - Fetches and emits the list of online friends for the connecting user.

### `OnlineFiends`
- **Description**: Triggered when a client requests the list of their online friends. (Note: There's a typo in the event name, likely intended to be `OnlineFriends`).
- **Payload**: `{ userId: string }`
    - `userId`: The ID of the user requesting their online friends.
- **Emits**:
    - `GetOnlineFriends` to the connected client (`socket.id`):
        - **Payload**: `onlineFriendsForUser: Array<{ id: string, ...otherFriendData }>` - An array of online friends for the user.
- **Internal Actions**:
    - Ensures the user is in the `OnlineUsers` map.
    - Fetches and emits the list of online friends for the user.

### `Chats`
- **Description**: Triggered when a client requests their list of chats.
- **Payload**: `{ userId: string }`
    - `userId`: The ID of the user requesting their chats.
- **Emits**:
    - `GetChats` to the connected client (`socket.id`):
        - **Payload**: `{ Chats: Array<ChatObject> }` - An array of chat objects for the user, including online status of participants.
- **Internal Actions**:
    - Ensures the user is in the `OnlineUsers` map.
    - Fetches and emits the user's chats.

### `GetMessages`
- **Description**: Triggered when a client requests messages for a specific chat.
- **Payload**: `{ userId: string, chatId: string }`
    - `userId`: The ID of the user requesting messages (used to find their socket).
    - `chatId`: The ID of the chat for which messages are requested.
- **Emits**:
    - `getMessage` to the requesting user's socket(s) (`recipient.socketId`):
        - **Payload**: `Messages: Array<MessageObject>` - An array of messages for the specified chat.
- **Internal Actions**:
    - Fetches all messages for the given `chatId` and `userId` (to determine read status).
    - Emits the messages to the user if they are online.

### `SeenMessage`
- **Description**: Triggered when a user has seen messages in a chat. Updates message status and notifies relevant users.
- **Payload**: `{ userId: string, chatId: string, reciverId?: string }`
    - `userId`: The ID of the user who has seen the messages.
    - `chatId`: The ID of the chat where messages were seen.
    - `reciverId`: (Optional) The ID of the other user in the chat, to update their view as well.
- **Emits**:
    - To the `userId`'s socket(s):
        - `getMessage`: **Payload**: `Messages: Array<MessageObject>` (updated messages for the chat).
        - `GetChats`: **Payload**: `{ Chats: Array<ChatObject> }` (updated chat list).
    - If `reciverId` is provided and online, to the `reciverId`'s socket(s):
        - `getMessage`: **Payload**: `Messages: Array<MessageObject>` (updated messages for the chat).
        - `GetChats`: **Payload**: `{ Chats: Array<ChatObject> }` (updated chat list).
- **Internal Actions**:
    - Marks messages in the specified chat as seen by the `userId`.
    - Fetches updated messages and chats for both the sender (`userId`) and receiver (`reciverId`, if online) and emits them.

### `Message`
- **Description**: Triggered when a user sends a new message.
- **Payload**: `{ senderId: string, reciverId: string, text: string, chatId: string }`
    - `senderId`: The ID of the user sending the message.
    - `reciverId`: The ID of the user receiving the message.
    - `text`: The content of the message.
    - `chatId`: The ID of the chat the message belongs to.
- **Emits**:
    - To the `senderId`'s socket(s):
        - `getMessage`: **Payload**: `Messages: Array<MessageObject>` (updated messages for the chat, including the new one).
    - If `reciverId` is online, to the `reciverId`'s socket(s):
        - `getMessage`: **Payload**: `Messages: Array<MessageObject>` (updated messages for the chat).
        - `GetChats`: **Payload**: `{ Chats: Array<ChatObject> }` (updated chat list for the receiver).
- **Internal Actions**:
    - Saves the new message to the database.
    - Fetches all messages for the chat.
    - Emits the updated messages to both sender and receiver (if online).
    - Emits updated chat list to the receiver (if online).
    - Sends a push notification to the `reciverId`.

### `AllFriends`
- **Description**: Triggered when a client requests their complete list of friends (both online and offline).
- **Payload**: `{ userId: string }`
    - `userId`: The ID of the user requesting their friends list.
- **Emits**:
    - `GetAllFriends` to the connected client (`socket.id`):
        - **Payload**: `AllUserFriends: Array<FriendObject>` - An array of all friends, with their online status.
- **Internal Actions**:
    - Fetches all friends of the `userId` and their current online status based on `OnlineUsers`.
    - Emits the comprehensive list to the user.

### `Logout`
- **Description**: Triggered when a user explicitly logs out.
- **Payload**: `{ userId: string }`
    - `userId`: The ID of the user logging out.
- **Emits** (to affected online friends of the logged-out user):
    - `GetOnlineFriends`: **Payload**: `OnlineFriendsForFriend: Array<{ id: string, ...otherFriendData }>` (updated list of their online friends).
    - `GetChats`: **Payload**: `{ Chats: Array<ChatObject> }` (updated chat list, reflecting the user's new offline status).
- **Internal Actions**:
    - Removes the `userId` from the `OnlineUsers` map.
    - Identifies online friends of the logged-out user.
    - For each affected online friend:
        - Fetches their updated list of online friends.
        - Fetches their updated chat list.
        - Emits these updates to the friend's socket(s).

### `disconnect`
- **Description**: Triggered automatically when a client's socket connection is lost.
- **Payload**: None.
- **Emits** (to affected online friends of the disconnected user, if the user had no other active sockets):
    - `GetOnlineFriends`: **Payload**: `OnlineFriendsForFriend: Array<{ id: string, ...otherFriendData }>` (updated list of their online friends).
    - `GetChats`: **Payload**: `{ Chats: Array<ChatObject> }` (updated chat list, reflecting the user's new offline status).
- **Internal Actions**:
    - Identifies the `userId` associated with the disconnected `socket.id`.
    - Removes the `socket.id` from the user's list of active sockets in `OnlineUsers`.
    - If the user has no more active sockets, removes the `userId` from `OnlineUsers`.
    - If a user was fully disconnected (all their sockets closed):
        - Identifies online friends of the disconnected user.
        - For each affected online friend:
            - Fetches their updated list of online friends.
            - Fetches their updated chat list.
            - Emits these updates to the friend's socket(s).

## Client-Side Events (Emitting to Server)

Clients should emit events with the payloads as described in the "Server-Side Events (Listening)" section.

## Client-Side Events (Listening From Server)

### `GetOnlineFriends`
- **Description**: Received by the client when their list of online friends is updated.
- **Payload**: `OnlineFriendForUser: Array<{ id: string, ...otherFriendData }>` or `OnlineFriendsForFriend: Array<{ id: string, ...otherFriendData }>`
    - An array of user objects representing friends who are currently online.

### `GetChats`
- **Description**: Received by the client when their list of chats is updated.
- **Payload**: `{ Chats: Array<ChatObject> }`
    - `Chats`: An array of chat objects, potentially with updated last messages, unread counts, or participant online statuses.

### `getMessage`
- **Description**: Received by the client when new messages arrive for a chat or when messages in a chat are updated (e.g., seen status).
- **Payload**: `Messages: Array<MessageObject>`
    - An array of message objects for a specific chat.

### `GetAllFriends`
- **Description**: Received by the client when they request their full list of friends.
- **Payload**: `AllUserFriends: Array<FriendObject>`
    - An array of friend objects, including their online status.

## Notes

- The server maintains an `OnlineUsers` Map to track which users are currently connected and their socket IDs.
- Users can have multiple socket connections (multiple devices/tabs), so socket IDs are stored in arrays.
- The server uses ping/pong with 10-second intervals and 5-second timeouts for connection health.
- Push notifications are sent when users receive messages and are offline.

# Global Chat Service Design

## Overview
This document outlines the design of a global chat service similar to WhatsApp or Facebook Messenger. The service enables users to communicate over the internet, supporting both one-on-one and group chats. Messages are stored for better viewing, and encryption is implemented to ensure security and privacy.

## Solution Architecture

### Overview
The global chat service follows a microservices architecture deployed on cloud infrastructure. It utilizes various cloud-native services for scalability, reliability, and security. The system incorporates message encryption to safeguard user communication and ensures seamless one-on-one and group chat experiences.

### Components

1. **Microservices:**
    - User Authentication Service
    - Message Service
    - Chat Service
    - Group Management Service
    - Encryption Service

2. **Data Storage:**
    - Apache Kafka for message streaming and persistence.
    - Relational Database for user data and settings.
    - NoSQL Database for chat history data.

3. **Media Storage:**
    - Cloud-native object storage service for storing media and document files.

### Communication Flow

1. **User Authentication Service:**
    - Handles user authentication and authorization.
    - Generates authentication tokens for authenticated users.

2. **Message Service:**
    - Manages the storage and retrieval of messages.
    - Publishes messages to Kafka topics for persistence and real-time processing.
    - Provides APIs for sending, receiving, and querying messages.

3. **Chat Service:**
    - Facilitates one-on-one and group chats.
    - Subscribes to Kafka topics for real-time message updates.
    - Manages real-time communication between users using WebSockets or SSE (Server-Sent Events).

4. **Group Management Service:**
    - Handles the creation, modification, and deletion of groups.
    - Manages group memberships and permissions.
    - Utilizes Kafka for group-related event streaming and persistence.

5. **Encryption Service:**
    - Provides end-to-end encryption for messages.
    - Encrypts messages before publishing them to Kafka.
    - Decrypts messages for authorized users upon retrieval.

### Data Flow User Registration and Authentication
- Users register and authenticate through the User Authentication Service.
- Upon successful authentication, users receive authentication tokens for accessing the chat service.

### Data Flow for One-on-One Chat

#### Sending a Message:

1. **User Initiates Message:**
    - A user initiates a chat from the chat interface by selecting another user and composing a message.
    - The user's client application sends a message to the Message Service via REST API or Kafka Producer.

2. **Message Processing:**
    - The Message Service processes the message, including encryption if necessary.
    - The processed message is published to the appropriate Kafka topic for persistence.
    - After message processing, the Message Service stores the message in a relational database for persistence. 
    - The Chat Service retrieves past messages from the database when a user requests message history.
3. **Real-time Communication:**
    - The Chat Service subscribes to the Kafka topic for one-on-one messages.
    - Upon receiving new messages, the Chat Service notifies the recipient in real-time via WebSockets or SSE.

4. **Message Delivery:**
    - The recipient's client application receives the real-time notification of the new message.
    - The message is displayed in the recipient's chat interface.

#### Retrieving Messages:

1. **User Requests Messages:**
    - A user opens a chat with another user in the chat interface and requests to view previous messages.
    - The client application sends a request to the Message Service via REST API or Kafka Consumer.

2. **Message Retrieval:**
    - The Message Service first checks the database for any locally stored past messages based on user IDs.
    - If the messages are not found in the local database, the Message Service retrieves them from Kafka.
    - Retrieved messages are sent back to the client application for display.

This ensures that message retrieval is optimized for performance by first checking the local database before resorting to Kafka for fetching historical data. Let me know if you need further adjustments!

#### Message Data Structure:

```json
{
  "messageId": "123456",
  "senderId": "user1",
  "recipientId": "user2",
  "content": "Hello, how are you?",
  "timestamp": "2024-04-16T12:00:00Z"
}
```
#### One-on-One Message Flow Diagram:
![One-on-One Message Flow Diagram](https://github.com/narmidm/system-design/blob/master/%20Global%20Chat%20Service%20Design/Cloud-Native%20way/image3.png)


### Data Flow for Group Messaging

#### Sending a Group Message:

1. **User Initiates Message:**
    - A user initiates a group message from the chat interface by selecting a group chat and composing a message.
    - The user's client application sends a message to the Message Service via REST API or Kafka Producer.

2. **Message Processing:**
    - The Message Service processes the message, including encryption if necessary.
    - The processed message is published to the appropriate Kafka topic for persistence.
    - Similar to one-on-one chat, group messages are stored in a relational database after processing by the Message Service. 
    - When a user requests group message history, the Message Service retrieves past messages from the database.
3. **Real-time Communication:**
    - The Chat Service subscribes to the Kafka topic for group messages.
    - Upon receiving new messages, the Chat Service distributes them to all group members in real-time.

4. **Message Delivery:**
    - Each group member's client application receives the real-time notification of the new message.
    - The message is displayed in the group chat interface for all members.

#### Retrieving Group Messages:

1. **User Requests Group Messages:**
    - A user opens a group chat in the chat interface and requests to view previous messages.
    - The client application sends a request to the Message Service via REST API or Kafka Consumer.

2. **Message Retrieval:**
    - The Message Service first checks the database for any locally stored past group messages based on the group ID.
    - If the messages are not found in the local database, the Message Service retrieves them from Kafka based on the group ID.
    - Retrieved messages are sent back to the client application for display.

By checking the local database first, we optimize the retrieval process and minimize the reliance on Kafka for fetching historical group messages. Let me know if you need further adjustments!

#### Group Message Data Structure:

```json
{
  "messageId": "789012",
  "groupId": "group1",
  "senderId": "user1",
  "content": "Let's meet at 3 PM.",
  "timestamp": "2024-04-16T12:05:00Z"
}
```

#### Group Message Flow Diagram:
![Group Message Flow Diagram](https://github.com/narmidm/system-design/blob/master/%20Global%20Chat%20Service%20Design/Cloud-Native%20way/image2.png)


### Data Flow for Message Attachments
- Message attachments (e.g., images, files) are stored in a cloud-native object storage service.
- The Message Service stores references to attachments along with messages in Kafka.
- Users retrieve attachments through pre-signed URLs provided by the object storage service.

### Security

1. **Authentication and Authorization:**
    - User Authentication Service verifies user credentials and issues authentication tokens.
    - Authentication tokens are used to authorize access to chat services and APIs.

2. **End-to-End Encryption:**
    - The Encryption Service ensures messages are encrypted before storage and decrypted for authorized recipients.
    - Utilizes strong cryptographic algorithms to protect message content.

## API Endpoints

### User Authentication Service

#### Endpoint: /auth/register

- **Method:** POST
- **Description:** Register a new user.
- **Request:**
    - **Body:**
        - `username` (string, required): The username of the new user.
        - `password` (string, required): The password of the new user.
- **Response:**
    - **Success (201 Created):**
        - Body: Confirmation message or newly created user data.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

#### Endpoint: /auth/login

- **Method:** POST
- **Description:** Authenticate a user and generate an authentication token.
- **Request:**
    - **Body:**
        - `username` (string, required): The username of the user.
        - `password` (string, required): The password of the user.
- **Response:**
    - **Success (200 OK):**
        - Body: Authentication token.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

### Chat Service

#### Endpoint: /chat/send

- **Method:** POST
- **Description:** Send a message.
- **Request:**
    - **Headers:**
        - Authorization: Bearer token for authentication.
    - **Body:**
        - `recipient` (string, required): Username of the message recipient.
        - `message` (string, required): Message content.
- **Response:**
    - **Success (200 OK):**
        - Body: Confirmation message.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

#### Endpoint: /chat/messages

- **Method:** GET
- **Description:** Get messages.
- **Request:**
    - **Headers:**
        - Authorization: Bearer token for authentication.
    - **Query Parameters:**
        - `recipient` (string, optional): Username of the message recipient.
- **Response:**
    - **Success (200 OK):**
        - Body: List of messages.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

### Group Management Service

#### Endpoint: /groups/create

- **Method:** POST
- **Description:** Create a new group.
- **Request:**
    - **Headers:**
        - Authorization: Bearer token for authentication.
    - **Body:**
        - `name` (string, required): Name of the group.
        - `members` (array of strings, required): Usernames of group members.
- **Response:**
    - **Success (201 Created):**
        - Body: Confirmation message or group data.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

#### Endpoint: /groups/{groupId}/add-member

- **Method:** PUT
- **Description:** Add a member to a group.
- **Request:**
    - **Headers:**
        - Authorization: Bearer token for authentication.
    - **Path Parameter:**
        - `groupId` (string, required): ID of the group.
    - **Body:**
        - `member` (string, required): Username of the member to add.
- **Response:**
    - **Success (200 OK):**
        - Body: Confirmation message.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

#### Endpoint: /groups/{groupId}/remove-member
- **Method:** DELETE
- **Description:** Remove a member from a group.
- **Request:**
    - **Headers:**
        - Authorization: Bearer token for authentication.
    - **Path Parameter:**
        - `groupId` (string, required): ID of the group.
    - **Body:**
        - `member` (string, required): Username of the member to remove.
- **Response:**
    - **Success (200 OK):**
        - Body: Confirmation message.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

### Group Messaging

#### Endpoint: /groups/{groupId}/send

- **Method:** POST
- **Description:** Send a message to a group.
- **Request:**
    - **Headers:**
        - Authorization: Bearer token for authentication.
    - **Path Parameter:**
        - `groupId` (string, required): ID of the group.
    - **Body:**
        - `message` (string, required): Message content.
- **Response:**
    - **Success (200 OK):**
        - Body: Confirmation message.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

#### Endpoint: /groups/{groupId}/messages

- **Method:** GET
- **Description:** Get messages from a group.
- **Request:**
    - **Headers:**
        - Authorization: Bearer token for authentication.
    - **Path Parameter:**
        - `groupId` (string, required): ID of the group.
- **Response:**
    - **Success (200 OK):**
        - Body: List of messages.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

### Encryption

#### Endpoint: /encrypt

- **Method:** POST
- **Description:** Encrypt a message.
- **Request:**
    - **Body:**
        - `message` (string, required): Message content to encrypt.
- **Response:**
    - **Success (200 OK):**
        - Body: Encrypted message.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

#### Endpoint: /decrypt

- **Method:** POST
- **Description:** Decrypt a message.
- **Request:**
    - **Body:**
        - `encryptedMessage` (string, required): Encrypted message content.
- **Response:**
    - **Success (200 OK):**
        - Body: Decrypted message.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure.

## Deployment Options

### Apache Kafka Microservices Architecture

#### Explanation

- **User Authentication Service:** Deployed as a standalone service handling user authentication and token generation.
- **Message Service:** Implemented as a microservice subscribing to Kafka topics for message processing and persistence.
- **Group Management Service:** Deployed as a microservice managing group-related operations and events.
- **Encryption Service:** Implemented as a microservice handling message encryption and decryption.

### Apache Kafka
- **Why Kafka?** Provides fault-tolerant, scalable, and distributed messaging capabilities.
- **Configuration:** Set up Kafka clusters for message streaming and persistence.

### Sequence Diagram for Deployment (Updated)
![sequence diagram illustrating the deployment process for the chat service](https://github.com/narmidm/system-design/blob/master/%20Global%20Chat%20Service%20Design/Cloud-Native%20way/image1.png)

## Real-time Communication Options

- **Polling:** Periodically requesting data from the server, generating a high number of requests which is inefficient.
- **Long Polling:** Holding the connection open until new data is available, reducing the number of requests and latency.
- **WebSocket:** A bidirectional communication protocol enabling real-time communication between the client and server over a single, long-lived connection, providing the lowest latency. It is the most common solution for sending async updates from server to client.
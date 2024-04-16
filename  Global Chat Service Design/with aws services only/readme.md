# Global Chat Service Design

## Overview
This document outlines the design of a global chat service similar to WhatsApp or Facebook Messenger. The service enables users to communicate over the internet, supporting both one-on-one and group chats. Messages are stored for better viewing, and encryption is implemented to ensure security and privacy.

## Solution Architecture

### Overview
The global chat service follows a microservices architecture deployed on cloud infrastructure. It utilizes various AWS services for scalability, reliability, and security. The system incorporates message encryption to safeguard user communication and ensures seamless one-on-one and group chat experiences.

### Components

1. **Microservices:**
    - User Authentication Service
    - Message Service
    - Chat Service
    - Group Management Service
    - Encryption Service

2. **Database:**
    - DynamoDB for storing user profiles, group information, and messages.

3. **Cloud Infrastructure:**
    - AWS Lambda for serverless execution of microservices.
    - AWS API Gateway for managing RESTful API endpoints.
    - Amazon S3 for storing message attachments (e.g., images, files).
    - Amazon CloudFront for content delivery and caching.

### Communication Flow

1. **User Authentication Service:**
    - Handles user authentication and authorization.
    - Generates authentication tokens for authenticated users.

2. **Message Service:**
    - Manages the storage and retrieval of messages.
    - Stores encrypted messages in DynamoDB.
    - Provides APIs for sending, receiving, and querying messages.

3. **Chat Service:**
    - Facilitates one-on-one and group chats.
    - Utilizes the Message Service for message storage and retrieval.
    - Manages real-time communication between users using WebSockets.

4. **Group Management Service:**
    - Handles the creation, modification, and deletion of groups.
    - Manages group memberships and permissions.
    - Utilizes DynamoDB for storing group information.

5. **Encryption Service:**
    - Provides end-to-end encryption for messages.
    - Implements cryptographic protocols to ensure data security.
    - Encrypts messages before storing them in the database and decrypts them for authorized users.

### Data Flow User Registration and Authentication
 - Users register and authenticate through the User Authentication Service.
 - Upon successful authentication, users receive authentication tokens for accessing the chat service.

### Data Flow for One-on-One Chat

#### Sending a Message:

1. **User Initiates Message:**
    - A user initiates a chat from the chat interface by selecting another user and composing a message.
    - The user's client application sends a POST request to the `/chat/send` endpoint with the recipient's username and message content.

2. **Authorization and Validation:**
    - The backend service receives the request and validates the user's authentication token to ensure they are logged in.
    - The service also validates the recipient's username to ensure it corresponds to an existing user.

3. **Message Processing:**
    - If the user and recipient are validated, the message content is processed by the backend service.
    - Additional operations such as encryption of the message content may be performed before storing it in the database.

4. **Storing the Message:**
    - The processed message, along with metadata such as sender ID, recipient ID, and timestamp, is stored in the database.
    - DynamoDB is queried to insert the message into the appropriate table, maintaining data consistency and integrity.

5. **Notification to Recipient:**
    - Once the message is successfully stored, the backend service sends a notification to the recipient user.
    - This notification can be delivered asynchronously through a messaging queue or WebSocket connection to minimize latency.

#### Retrieving Messages:

1. **User Requests Messages:**
    - A user opens a chat with another user in the chat interface and requests to view previous messages.
    - The client application sends a GET request to the `/chat/messages` endpoint with the recipient's username.

2. **Authorization and Validation:**
    - The backend service receives the request and validates the user's authentication token.

3. **Fetching Messages from Database:**
    - Upon successful validation, the backend service queries the database (DynamoDB) to retrieve all messages exchanged between the two users.
    - The retrieved messages are sorted by timestamp to ensure they are displayed in chronological order.

4. **Response to User:**
    - The backend service constructs a response containing the retrieved messages and sends it back to the user's client application.

5. **Displaying Messages:**
    - The client application receives the response from the backend service and displays the messages in the chat interface.
    - Messages may be rendered with sender information, timestamps, and other relevant metadata for better user experience.

#### Message Data Structure:

- **Message Schema:**
    - Each message in the chat is represented by a document in the database with the following structure:
      ```json
      {
        "messageId": "unique_message_id",
        "senderId": "sender_user_id",
        "recipientId": "recipient_user_id",
        "content": "message_content",
        "timestamp": "timestamp",
        // Additional metadata if needed
      }
      ```
    - **Explanation:**
        - `messageId`: Unique identifier for the message.
        - `senderId`: ID of the user who sent the message.
        - `recipientId`: ID of the user who received the message.
        - `content`: Content of the message.
        - `timestamp`: Timestamp indicating when the message was sent.

#### One-on-One Message Flow Diagram:

![One-on-One Message Flow Diagram](https://github.com/narmidm/system-design/blob/master/Global%20Chat%20Service%20Design/image4.png?raw=true)


### Data Flow for Group Messaging

#### Sending a Group Message:

1. **User Initiates Message:**
    - A user initiates a group message from the chat interface by selecting a group chat and composing a message.
    - The user's client application sends a POST request to the `/groups/{groupId}/send` endpoint with the message content and the group ID.

2. **Authorization and Validation:**
    - The backend service receives the request and validates the user's authentication token to ensure they have permission to send messages to the group.
    - The service also validates the group ID to ensure it exists and that the user is a member of the group.

3. **Message Processing:**
    - If the user and group are validated, the message content is processed by the backend service.
    - The service may perform additional operations such as encryption of the message content before storing it in the database.

4. **Storing the Message:**
    - The processed message, along with metadata such as sender ID, timestamp, and group ID, is stored in the database.
    - DynamoDB is queried to insert the message into the appropriate table, ensuring data integrity and consistency.

5. **Notification to Group Members:**
    - Once the message is successfully stored, the backend service notifies all members of the group about the new message.
    - This notification can be sent asynchronously through a messaging queue or WebSocket connection to minimize latency.

#### Retrieving Group Messages:

1. **User Requests Group Messages:**
    - A user opens a group chat in the chat interface and requests to view previous messages.
    - The client application sends a GET request to the `/groups/{groupId}/messages` endpoint with the group ID.

2. **Authorization and Validation:**
    - The backend service receives the request and validates the user's authentication token and group ID.

3. **Fetching Messages from Database:**
    - Upon successful validation, the backend service queries the database (DynamoDB) to retrieve all messages associated with the group ID.
    - The retrieved messages are sorted by timestamp to ensure they are displayed in chronological order.

4. **Response to User:**
    - The backend service constructs a response containing the retrieved messages and sends it back to the user's client application.

5. **Displaying Messages:**
    - The client application receives the response from the backend service and displays the group messages in the chat interface.
    - Messages may be rendered with sender information, timestamps, and other relevant metadata for better user experience.

#### Group Message Data Structure:

- **Message Schema:**
    - Each message in the group chat is represented by a document in the database with the following structure:
      ```json
      {
        "messageId": "unique_message_id",
        "groupId": "group_id",
        "senderId": "user_id",
        "content": "message_content",
        "timestamp": "timestamp",
        // Additional metadata if needed
      }
      ```
    - **Explanation:**
        - `messageId`: Unique identifier for the message.
        - `groupId`: ID of the group to which the message belongs.
        - `senderId`: ID of the user who sent the message.
        - `content`: Content of the message.
        - `timestamp`: Timestamp indicating when the message was sent.

#### Group Message Flow Diagram:

![Group Message Flow Diagram](https://github.com/narmidm/system-design/blob/master/Global%20Chat%20Service%20Design/image3.png?raw=true)

### Data Flow for Message Attachments
- Message attachments (e.g., images, files) are stored in Amazon S3.
- The Message Service stores references to attachments along with encrypted messages in DynamoDB.
- Users retrieve attachments through pre-signed URLs provided by Amazon S3.

### Security

1. **Authentication and Authorization:**
    - User Authentication Service verifies user credentials and issues authentication tokens.
    - Authentication tokens are used to authorize access to chat services and APIs.

2. **End-to-End Encryption:**
    - The Encryption Service ensures messages are encrypted before storage and decrypted for authorized recipients.
    - Utilizes strong cryptographic algorithms to protect message content.

3. **Data Protection:**
    - DynamoDB data is encrypted at rest to protect user data.
    - Access to AWS services is controlled through IAM roles and policies.

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

### AWS Lambda Microservices Architecture

#### Explanation

- **User Authentication Service:** Deployed as a Lambda function triggered by API Gateway endpoints /auth/register and /auth/login.
- **Message Service:** Implemented as a Lambda function handling /chat/send and /chat/messages endpoints.
- **Group Management Service:** Deployed as a Lambda function triggered by API Gateway endpoints /groups/create and /groups/{groupId}/add-member.
- **Encryption Service:** Implemented as a Lambda function providing /encrypt and /decrypt endpoints.

### DynamoDB
- **Why DynamoDB?** Offers seamless integration with AWS Lambda and provides scalable, fully managed NoSQL database capabilities.
- **Configuration:** Set up DynamoDB tables for storing user profiles, messages, and group information.
- **Scalability:** DynamoDB automatically scales to handle varying workloads and storage requirements.

### Amazon S3 and CloudFront
- **Storage:** Use Amazon S3 for storing message attachments.
- **Content Delivery:** Utilize Amazon CloudFront for content delivery and caching to improve performance.

#### Sequence diagram illustrating the deployment process:

![sequence diagram illustrating the deployment process for the chat service](https://github.com/narmidm/system-design/blob/master/Global%20Chat%20Service%20Design/image5.png?raw=true)

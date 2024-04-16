# User Management System

## Overview
This repository contains a user management system capable of registering, authenticating, and controlling customers. It's designed with a microservices architecture and can be deployed using AWS Lambda or Kubernetes.


## Solution Architecture and Low-Level Design

### Solution Architecture

#### Overview
The User Management System is designed to handle user registration, authentication, profile management, and deletion functionalities. It consists of microservices deployed on a cloud infrastructure, with communication facilitated through RESTful APIs. The system can use either MongoDB or DynamoDB for data storage, depending on the specific requirements and preferences.

#### Components
1. **Microservices:**
    - User Registration Service
    - Authentication Service
    - Profile Update Service
    - User Deletion Service

2. **Database:**
    - Option 1: MongoDB for storing user data.
    - Option 2: DynamoDB for storing user data.

3. **Cloud Infrastructure:**
    - AWS Lambda for serverless execution of microservices.
    - AWS API Gateway for managing RESTful API endpoints.
    - AWS IAM for managing access and authentication.
4. **Cloud Native Infrastructure:**
    - Kubernetes cluster for containerized deployment of microservices.

#### Communication Flow

**AWS Lambda Deployment:**
1. Clients interact with the system through RESTful API endpoints exposed by AWS API Gateway.
2. API Gateway triggers the respective Lambda functions based on incoming requests.
3. Lambda functions execute the business logic, interact with the selected database (MongoDB or DynamoDB) for data operations, and return responses to clients.

**Kubernetes Deployment:**
1. Clients interact with the system through RESTful API endpoints exposed by Kubernetes Ingress or Service.
2. Ingress or Service routes the requests to the respective microservices deployed in the Kubernetes cluster.
3. Microservices execute the business logic, interact with the selected database (MongoDB or DynamoDB) for data operations, and return responses to clients.

### Low-Level Design

#### Microservices Design
Each microservice follows a similar architecture pattern:

1. **User Registration Service:**
    - Receives registration requests.
    - Validates input data (e.g., username uniqueness, password strength).
    - Stores user data in the selected database (MongoDB or DynamoDB).

2. **Authentication Service:**
    - Receives authentication requests.
    - Validates user credentials against the selected database (MongoDB or DynamoDB).
    - Generates and returns JWT token for authenticated users.

3. **Profile Update Service:**
    - Handles profile update requests.
    - Validates user identity and input data.
    - Updates user details in the selected database (MongoDB or DynamoDB).

4. **User Deletion Service:**
    - Processes user deletion requests.
    - Validates user identity.
    - Deletes user record from the selected database (MongoDB or DynamoDB).

#### Data Storage
User data can be stored in either MongoDB or DynamoDB, depending on the specific requirements and preferences of the system. Both databases offer scalability, reliability, and flexibility in managing user-related data.

MongoDB can be preferred if vendor locking is a concern, as it is a popular NoSQL database that can be deployed on various cloud providers or on-premises. On the other hand, DynamoDB is a fully managed service provided by AWS, making it a convenient choice for organizations heavily invested in the AWS ecosystem or those seeking seamless integration with other AWS services. However, it's important to note that DynamoDB is AWS-specific and may result in vendor lock-in.

#### Security
- Access to microservices and the selected database (MongoDB or DynamoDB) is controlled using AWS IAM roles and policies.
- Authentication Service generates JWT tokens for authorized users, ensuring secure access to protected endpoints.
- Communication between clients and microservices is encrypted using HTTPS.

#### Error Handling
- Each microservice implements error handling mechanisms to handle exceptions gracefully.
- Error responses include appropriate status codes and error messages to guide clients.

#### Scalability
- The system is designed to be horizontally scalable, with AWS Lambda automatically scaling based on demand.
- Both MongoDB and DynamoDB support automatic scaling to handle increased throughput or storage requirements.

#### Monitoring and Logging
- AWS CloudWatch is used for monitoring Lambda function invocations, errors, and performance metrics.
- Logging is implemented within microservices to capture relevant information for troubleshooting and auditing purposes.



## Microservices in Detail
The system is broken down into the following microservices:

### User Registration Service

- **Description:** Handles new user registrations.
- **Technology Used:**
    - **Database:** (MongoDB or DynamoDB)
    - **Authentication:** JSON Web Tokens (JWT)
- **Responsibilities:**
    - Receives registration requests from clients.
    - Validates input data (e.g., username uniqueness, password strength).
    - Stores user data in (MongoDB or DynamoDB).

### User Update Service

- **Description:** Allows updating existing user details.
- **Technology Used:**
    - **Database:** (MongoDB or DynamoDB)
- **Responsibilities:**
    - Receives update requests from clients.
    - Validates input data and user identity.
    - Updates user details in (MongoDB or DynamoDB).

### User Deletion Service

- **Description:** Facilitates user deletions.
- **Technology Used:**
    - **Database:** (MongoDB or DynamoDB)
- **Responsibilities:**
    - Receives deletion requests from clients.
    - Validates user identity.
    - Deletes user record from (MongoDB or DynamoDB).

### User Retrieval Service

- **Description:** Retrieves all users from the database.
- **Technology Used:**
    - **Database:** (MongoDB or DynamoDB)
- **Responsibilities:**
    - Retrieves user data from (MongoDB or DynamoDB).
    - Provides user data to clients.

### Authentication Service

- **Description:** Authenticates users with their credentials.
- **Technology Used:**
    - **Authentication:** JSON Web Tokens (JWT)
- **Responsibilities:**
    - Receives authentication requests from clients.
    - Validates user credentials against (MongoDB or DynamoDB).
    - Generates JWT token for authenticated users.


## API Endpoints

### Register User

- **Endpoint:** `POST /users`
- **Description:** Register a new user.
- **Request:**
    - **Body:**
        - `username` (string, required): The username of the new user.
        - `password` (string, required): The password of the new user.
        - Additional fields for user details (e.g., email, name) can be included if needed.
- **Response:**
    - **Success (201 Created):**
        - Body: Confirmation message or newly created user data.
    - **Error (4xx):**
        - Body: Error message indicating the reason for failure (e.g., invalid input, username already exists).

### Update User

- **Endpoint:** `PUT /users/{username}`
- **Description:** Update a user's details.
- **Request:**
    - **Body:**
        - New user details to be updated.
    - **Path Parameter:**
        - `username` (string, required): The username of the user to be updated.
- **Response:**
    - **Success (200 OK):**
        - Body: Confirmation message or updated user data.
    - **Error (4xx or 5xx):**
        - Body: Error message indicating the reason for failure (e.g., user not found, invalid input).

### Delete User

- **Endpoint:** `DELETE /users/{username}`
- **Description:** Delete a user.
- **Request:**
    - **Path Parameter:**
        - `username` (string, required): The username of the user to be deleted.
- **Response:**
    - **Success (204 No Content):**
        - Body: No content.
    - **Error (4xx or 5xx):**
        - Body: Error message indicating the reason for failure (e.g., user not found, deletion failed).

### Get Users

- **Endpoint:** `GET /users`
- **Description:** Retrieve all users.
- **Request:**
    - No request parameters.
- **Response:**
    - **Success (200 OK):**
        - Body: List of user data.
    - **Error (4xx or 5xx):**
        - Body: Error message indicating the reason for failure (e.g., database error).

### Authenticate User

- **Endpoint:** `POST /users/authenticate`
- **Description:** Authenticate a user.
- **Request:**
    - **Body:**
        - `username` (string, required): The username of the user to be authenticated.
        - `password` (string, required): The password of the user to be authenticated.
- **Response:**
    - **Success (200 OK):**
        - Body: Authentication token or success message.
    - **Error (4xx or 5xx):**
        - Body: Error message indicating the reason for failure (e.g., invalid credentials).


## Data Flow:

### Register User:

1. User submits registration details through the user interface.
2. Frontend sends the registration request to the backend server.
3. Backend validates the input data (e.g., username uniqueness, password strength).
4. If validation passes, the backend stores the user data in the user collection in the database.
5. Confirmation message is sent to the frontend for display to the user.

### Update User:

1. User requests to update their profile information.
2. Frontend sends the update request to the backend with the new user data and the username to be updated.
3. Backend verifies the user's identity and checks for the existence of the specified username.
4. If the user is authenticated and the username exists, the backend updates the user's information in the database.
5. Confirmation message is sent to the frontend.

### Delete User:

1. User requests to delete their account.
2. Frontend sends the delete request to the backend along with the username to be deleted.
3. Backend verifies the user's identity and checks for the existence of the specified username.
4. If authenticated and the username exists, the backend deletes the user's record from the database.
5. Confirmation message is sent to the frontend.

### Get Users:

1. Admin or authorized user requests to retrieve all users.
2. Frontend sends the request to the backend.
3. Backend retrieves all user records from the database.
4. User data is sent back to the frontend for display.

### Authenticate:

1. User provides their username and password.
2. Frontend sends the credentials to the backend.
3. Backend validates the credentials against the stored data in the database.
4. If authentication succeeds, a token is generated and sent back to the frontend for future authorized access.


## Deployment Options

### AWS Lambda Microservices Architecture

### Explanation

We deploy each microservice as a separate AWS Lambda function, which provides a serverless environment. This architecture offers several benefits, including automatic scaling and cost optimization, as you only pay for the compute time you consume.

1. **User Registration Service:**
    - We create a Lambda function to handle user registration requests.
    - An API Gateway endpoint is set up to trigger this Lambda function when registration requests are received.
    - The API Gateway integrates with the Lambda function to pass incoming requests.
    - The Lambda function interacts with (MongoDB or DynamoDB) Atlas, where user registration data is stored.

2. **User Update Service:**
    - Another Lambda function is created to handle user profile update requests.
    - A separate API Gateway endpoint is set up to trigger this Lambda function.
    - Similarly, the API Gateway integrates with the Lambda function to pass update requests.
    - The Lambda function interacts with (MongoDB or DynamoDB) Atlas to update user details.

3. **User Deletion Service:**
    - A Lambda function is deployed to facilitate user deletion requests.
    - Again, an API Gateway endpoint is configured to trigger this Lambda function.
    - The API Gateway integrates with the Lambda function to pass deletion requests.
    - The Lambda function interacts with (MongoDB or DynamoDB) Atlas to delete user records.

4. **User Retrieval Service:**
    - We deploy another Lambda function to retrieve all users from the database.
    - An API Gateway endpoint is set up to trigger this Lambda function.
    - The API Gateway integrates with the Lambda function to pass retrieval requests.
    - The Lambda function interacts with (MongoDB or DynamoDB) Atlas to fetch user data.

5. **Authentication Service:**
    - A Lambda function is set up to handle user authentication requests.
    - An API Gateway endpoint triggers this Lambda function for authentication.
    - The API Gateway integrates with the Lambda function to pass authentication requests.
    - The Lambda function interacts with (MongoDB or DynamoDB) Atlas to validate user credentials.

In summary, each microservice is deployed as a separate Lambda function, triggered by an API Gateway endpoint. These Lambda functions interact with (MongoDB or DynamoDB) Atlas for data storage and retrieval. This architecture offers scalability, cost-effectiveness, and seamless integration with AWS services.

### Kubernetes
- **Why Kubernetes?** It provides a system for automating deployment, scaling, and operations of application containers across clusters of hosts.
- **Architecture**: Each microservice runs in its own container within a pod. Kubernetes services provide stable endpoints for these pods.


#### User Registration Service

#### Deployment YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-registration-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-registration-service
  template:
    metadata:
      labels:
        app: user-registration-service
    spec:
      containers:
      - name: user-registration-service
        image: your-registry/user-registration-service:latest
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: user-registration-service
spec:
  selector:
    app: user-registration-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```
#### Explanation

##### Deployment:
- **Definition:** Defines the configuration for the User Registration Service deployment.
- **Replicas:** Specifies the number of pod replicas (instances) for high availability. Adjust as needed.
- **Selector:** Defines the labels used to select pods for this deployment.
- **Template:** Defines the pod template for the deployment.
    - **Container:** Specifies the container details including the Docker image and port.

##### Service:
- **Definition:** Exposes the User Registration Service within the Kubernetes cluster.
- **Selector:** Specifies the labels used to select pods for the service.
- **Ports:** Defines the port mapping between the service and the pods.


##### Deployment of Other Microservices

Each microservice will have its own Deployment and Service defined in YAML files. The Deployment will specify the configuration for the microservice, including the number of replicas, labels, and pod template. The Service will expose the microservice within the Kubernetes cluster, allowing other services to communicate with it.

The communication between microservices can be achieved through Kubernetes services. Each microservice will have its own service defined, which other microservices can use to access it. Kubernetes provides built-in DNS for service discovery, allowing microservices to communicate with each other using their service names.

![Detailed Flow Diagrams for User Management System](https://github.com/narmidm/system-design/blob/master/user%20management%20system/image1.png?raw=true)
![Flow Charts for AWS Lambda ](https://github.com/narmidm/system-design/blob/master/user%20management%20system/image2.png?raw=true)

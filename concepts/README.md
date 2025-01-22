<h1>
  <span class="headline">JWT Authentication in Flask APIs</span>
  <span class="subhead">Concepts</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to describe using JWTs as an authentication strategy.

## Authentication and Authorization

APIs often require some sort of **authentication** for you to use them - that is, some way of knowing who you are. They'll also often manage resource access, which we call authorization. We'll frequently use these two words together, but they handle two distinct parts of an application.

| Authentication                           | Authorization                              |
| ---------------------------------------- | ------------------------------------------ |
| Confirms users are who they say they are | Manages user access to different resources |

## Token Authentication with JWT

There are many ways to manage **authentication**, but for this lesson we'll focus on Token Authentication. This style of authentication in HTTP uses _tokens_, or unique identifier strings, to tell who a user is when they're making an **authenticated request**.

JSON Web Tokens (JWT) are a type of token authentication. JWT utilizes a JSON-formatted token that is signed with a secret key. This token is then sent to the client, who sends it back to the server with every request. The server can then verify the token and use the information in it to **authenticate** the user.

## Authentication Flow with JWT

Let's go through the flow of how JWT works in an application.

### Token Generation

1. **User Login**: The user provides their credentials (username/password) to the server for authentication.
2. **Authentication**: The server verifies the provided credentials. If they are correct, the server generates a JWT.
3. **JWT Creation**: The server creates a JWT by encoding user information (such as user ID, username, and possibly roles or permissions) into a JSON payload. This payload is digitally signed using a secret key known only to the server.
4. **Encrypted Token**: The server encrypts the JWT with its secret key, producing the final encrypted token.

### Token Inclusion in HTTP Requests

1. **Token Issuance**: After successful login, the server sends the encrypted JWT back to the client.
2. **Token Storage**: The client typically stores the JWT in local storage or a cookie for later use.
3. **HTTP Requests**: When the client needs to access protected resources on the server, the JWT is included in the HTTP headers of the request.

### Token Verification and Authentication

1. **HTTP Request with Token**: The client sends an HTTP request to the server, including the JWT in the Authorization header.
2. **Token Extraction**: Upon receiving the request, the server extracts the JWT from the Authorization header.
3. **Token Decryption**: The server decrypts the JWT using its secret key, verifying its authenticity. If the decryption is successful, the server obtains the JSON payload.
4. **User Authentication**: The server verifies the claims within the JSON payload to authenticate the user. This typically involves checking if the user exists, if they have the required permissions, and if the token has not expired.
5. **Access Granted**: If the token is valid and the user is authenticated, the server grants access to the requested resource or performs the requested action.

In conclusion, JWTs are a powerful tool for managing authentication in web applications. They allow for secure, stateless authentication and can be used to manage user access to resources. By understanding how JWTs work you can build secure, scalable web applications.

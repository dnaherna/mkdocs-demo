---
title: Introduction to API authentication
---

In this module, we will explore the various authentication concerns in APIs and how they can be exploited. By the end, you will know how to secure APIs effectively and prevent similar attacks when you implement an API.

## Authentication

To integrate with different software or services, APIs will often define methods to share data and perform actions. For example, you might be familiar with delivery services. These services use APIs to share location data and estimated arrival times so you can stay updated. Because APIs often handle sensitive data, we expect only relevant parties can see our data.

It would be wrong if anyone can access each other's data. That is why at the core of any API is authentication. This is a process that verifies the identity of a user to ensure only authorized people can access the API. We need authentication to secure sensitive data and ensure privacy and data integrity.

## Authentication Methods

When it comes to authentication, there are several methods available, each with its own use cases. These methods provide the necessary security and verification to ensure only authorized users can interact with the API.

**Basic Authentication:** This is an older HTTP-based authentication scheme that lets users send their username and password as plain text in the HTTP request. This means it is easier to implement, but it is now considered outdated and insecure with most services requiring more secure methods.

**API Keys:** This is a simple and commonly used method, where a unique key is assigned to each user and must be included in each API request. This ensures that the user is identified and authorized to access the API, but these keys can be hard to manage and distribute in large environments.

**JSON Web Tokens:** This is a stateless and compact method to authenticate users. JWTs encode user credentials into a token, which is then sent with each API request. This means the server does not need to store session information, which makes JWTs a scalable solution to authenticate users.

**OAuth:** This is a more advanced and secure protocol that allows users to grant third-party services access to their resources without sharing their actual credentials. It provides a safe and convenient way for users to log in to services using their existing social media accounts or other platforms.

Each method has its own uses cases. To decide which method is best, we have to think about the security requirements, user experience, and scalability. For example, API keys are secure but it could be awkward for everyday use on a social media platform. But with proper authentication, we can secure sensitive data and maintain the integrity of the API.

## JWTs

JSON Web Tokens (JWTs) were created to safely send data between systems. It uses a token, or compact string, to contain any kind of data, but is most commonly used to send information about users as part of authentication.

What makes JWTs different from other authentication methods is that it stores all the data a server needs on the client side. This makes JWTs a popular choice when it comes to highly distributed services that scale.

To better understand this, let's look at a JSON Web Token:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.
KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30
```

To make it easier to read, we split this token to show each part on its own line. While this token might look random, it is actually JSON data that has been encoded. This makes it very compact and easy to send in an API.

The encoding used for JWTs is Base64Url, a variation of Base64 encoding that is URL-safe. This encoding ensures that the token can easily be sent in HTTP headers and URLs without issues. Base64Url encoding makes the token compact and easy to handle in API calls and can be decoded to see the data.

```javascript
// URL-safe variant of Base64
function b64(str) {
    return new Buffer(str).toString('base64')
                          .replace(/=/g, '')
                          .replace(/\+/g, '-')
                          .replace(/\//g, '_');
}
```

If you decode this token, you will find information about the user that received this token. When a user sends a request to the API, they can use their token to tell the API their identity and what permissions they have.

For example, this user is an admin called John Doe.

```text
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.
```

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "iat": 1516239022
}
```

By using JWTs, you can create secure and scalable systems where user information is reliably verified with each request. This makes JWTs an excellent choice for modern web applications and distributed services.

## JWT Format

A JWT is made of three parts: the header, the payload, and the signature. Each part is encoded separately before being combined into the final token. These are all separated by a dot, as shown in the previous example.

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.
KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30
```

In most cases, the header typically has two fields: the signing algorithm and the type of token. The signing algorithm tells the server what algorithm to use to verify the signature. And the type depends but is usually always JWT.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

The payload contains all the information about the user. These are called "claims", and some are defined by the JWT spec. But, you will most likely define your own "claims" that aligns with what you need to authenticate.

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "iat": 1516239022
}
```

By now, you might have noticed that the data inside the token can easily be read or modified by anyone with access to the token. Because of this, the security of any JWT-based authentication is heavily reliant on the signature.

## JWT Signature

The signature can tell us if the token was tampered with. To create a signature we use the encoded header, encoded payload, a secret, and the algorithm specified in the header. This is signed to create a secure token.

For example, let's say we want to use the HMAC SHA256 algorithm:

```text
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

When we combine these parts together we can create a signature:

```text
KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30
```

Anytime the API gets a JWT, we can use the signature to verify the content it has. This is because it is derived from the rest of the token, so changing a single byte in the header or payload will result in a mismatched signature.

## JWT Security

As you work with JWTs, you will notice how flexible they are. With JWTs, you can decide many implementation details to fit your use case. This is by design, making JWTs very adaptable, however, this flexibility can lead to vulnerabilities if not handled properly, even when using well-tested libraries.

For example, one common mistake happens when the signature is not verified properly. Because the token tells the server what signing algorithm to use, an attacker can confuse the server with an algorithm it does not use, or trick the server by telling it there was no signing algorithm used at all.

This is the `none` algorithm specified in the JWT spec.

```json
{
  "alg": "none",
  "typ": "JWT"
}
```

When the signature is not verified properly, this is known as an unsecure token. As a result, an attacker can modify the token to gain complete access.

To avoid this, we need to implement proper checks and validations on the server side. For example, always ensure that the server only accepts specified algorithms and rejects tokens with none or any unsupported algorithms.

```javascript
const allowedAlgorithms = [
  'HS256', // HMAC using SHA-256
  'RS256', // RSASSA PKCS1 v1.5 using SHA-256
  'ES256'  // ECDSA using P-256 and SHA-256
];
```

But, even if the signature is robustly verified, the security of JWTs relies heavily on the server's secret key staying a secret. If this key is leaked, guessed, or brute-forced, an attacker can generate a valid signature for any token. This would compromise the entire authentication mechanism.

## Broken Authentication

In APIs, we sometimes need to authenticate users to control what data they can access and the actions they can perform. This is because APIs often handle sensitive data, so we need robust authentication to verify the users.

Broken Authentication is a vulnerability that happens when the API fails to properly authenticate users, which means unauthorized users could gain access to the API and its resources. This vulnerability usually happens because of weak credentials, misconfigurations, or wrong implementation.

To better understand this, let's say we have an API that uses JWTs to authenticate users and we have an endpoint that returns sensitive data.

```python
@app.route('/sensitive-data', methods=['GET'])
def sensitive_data():
    token = request.headers.get('Authorization').split()[1]
    try:
        decoded = jwt.decode(token, options={'verify_signature': VERIFY_SIGNATURE})
        return jsonify({'data': 'This is secured data'}), 200
    except jwt.InvalidTokenError:
        return jsonify({'message': 'Unauthorized'}), 401
```

When a user wants to access their data, a request is made to the `sensitive-data` endpoint. This will return any data about the user if their token is valid. However, an attacker might try to tamper and modify their token to steal another user's data, so we try to verify the token signature.

But during development and testing phases, it might be easier to disable signature verification, so we include a `VERIFY_SIGNATURE` option to determine whether the token's signature should be verified. Developers can set this option locally using environment variables which is loaded after.

This option, along with other data is managed in a `Settings` config.

```python
class Settings:
    JWT_SECRET_KEY = os.getenv("JWT_SECRET_KEY", generate_random_secret())

    # someone needs to remember to set this variable to True in env variables
    JWT_VERIFY_SIGNATURE = os.getenv("JWT_VERIFY_SIGNATURE")
```

However, when the application was deployed to production, the signature verification was mistakenly left disabled, leading to broken authentication. This means an attacker, could skip the signature and access the endpoint.

```json
{
  "data": "This is secured data"
}
```

## Handle Broken Authentication

To address this issue, we need to ensure that the signature will always be verified. This is a simple misconfiguration mistake that can be fixed easily by setting `True` as the default value for `JWT_VERIFY_SIGNATURE`.

```python
class Settings:
    JWT_SECRET_KEY = os.getenv("JWT_SECRET_KEY", generate_random_secret())

    JWT_VERIFY_SIGNATURE = os.getenv("JWT_VERIFY_SIGNATURE", True)
```

By using default values, we can ensure that important options are securely set to reduce the risk of broken authentication due to misconfiguration.

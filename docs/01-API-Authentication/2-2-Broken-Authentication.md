---
title: Broken authentication
---

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

---
title: Practice
---

## Multiple Choice

**Question 1:** Which of the following methods provides the most secure way to authenticate users in an API?

- A. Basic authentication with plaintext passwords.
- B. API keys passed in query parameters.
- C. Token-based authentication using signed JWTs.
- D. IP-based authentication without additional credentials.

Correct Answer: C

**Question 2:** Which of the following is the most effective strategy to mitigate brute-force attacks in API authentication?

- A. Increase the length of passwords required during registration.
- B. Implement account lockout after consecutive failed login attempts.
- C. Store hashed passwords in the database.
- D. Use symmetric encryption for transmitted credentials.

Correct Answer: B

**Question 3:** A company decides to use facial recognition as the primary method for API authentication in its apps. Which of the following poses the greatest ethical and security risk?

- A. Reduced convenience for users due to technical failures.
- B. Inadequate protection for biometric data against theft or misuse.
- C. Difficulty in scaling facial recognition technology across devices.
- D. Increased reliance on machine learning algorithms to process biometric data.

Correct Answer: B

**Question 4:** Consider the following login function:

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.json.get('username')
    password = request.json.get('password')

    # Hardcoded credentials
    if username == "admin" and password == "password123":
        return jsonify({"message": "Login successful"}), 200
    return jsonify({"error": "Invalid credentials"}), 401
```

What is the primary vulnerability in this implementation?

- A. Hardcoded credentials can be easily guessed or leaked.
- B. The `password123` is stored securely but can be brute-forced.
- C. The username is hardcoded, but the password is dynamically validated.
- D. Weak passwords are encrypted improperly during transmission.

Correct Answer: A

**Question 5:** Consider the following token validation implementation:

```python
@app.route('/validate', methods=['POST'])
def validate_token():
    token = request.headers.get('Authorization')
    if not token:
        return jsonify({"error": "Unauthorized"}), 401

    # Misconfiguration: signature verification disabled during development
    decoded = jwt.decode(token, options={"verify_signature": False})
    return jsonify({"message": "Token is valid"}), 200
```

What is the result of the misconfiguration in this code?

- A. Token expiration claims are ignored, resulting in prolonged validity.
- B. Signature verification causes the API to reject all tokens.
- C. The decoding process fails due to missing headers in the token.
- D. Any token can bypass signature validation and be accepted.

Correct Answer: D

**Question 6:** Consider this API authentication flow:

```python
@app.route('/login', methods=['POST'])
def login():
    email = request.json.get('email')
    code = request.json.get('access_code')
    if authenticate(email, code):
        return jsonify({"message": "Access granted"}), 200
    return jsonify({"error": "Access denied"}), 403
```

The developer is asked to add IP rate limiting to prevent abuse. Which approach BEST integrates this feature?

- A. Create a blacklist of IPs that exceed a predefined request limit.
- B. Validate the client’s access code and IP address before processing login requests.
- C. Use middleware to track and enforce limits on requests per IP address.
- D. Limit the number of authentication attempts allowed globally.

Correct Answer: C

**Question 7:** How did attackers bypass Kaiten's rate limiting mechanism?

- A. They used a dictionary attack to guess PINs.
- B. They spoofed multiple IP addresses using the `X-Forwarded-For` header.
- C. They exploited an endpoint that didn't require authentication.
- D. They injected malicious SQL queries into the PIN request payloads.

Correct Answer: B

**Question 8:** In the provided proof of concept, why was the delay between retries important for the brute force attack?

- A. It synchronized with the expiration time of the generated PIN.
- B. It allowed the attackers to generate new PIN codes dynamically.
- C. It ensured PIN codes were guessed in sequential order.
- D. It prevented the server from detecting abnormal behavior.

Correct Answer: A

**Question 9:** In AWS IAM, what optional feature introduced timing differences that could be exploited for username enumeration?

- A. Session-based authentication
- B. Single sign-on (SSO)
- C. Role-based access control
- D. Multi-factor authentication (MFA)

Correct Answer: D

**Question 10:** Why were timing differences in AWS IAM's authentication flow measurable during login attempts?

- A. They revealed valid usernames by checking passwords before usernames.
- B. They occurred only for usernames associated with administrator roles.
- C. Valid usernames triggered additional checks, resulting in slight delays.
- D. Invalid usernames generated server errors, causing longer delays.

Correct Answer: C

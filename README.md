# Assignment

The objective of the assignment was to programmatically log into the secure internal portal (http://51.195.24.179:5005) without using browser automation tools. The login system uses JavaScript on the client side to generate a Base64‐encoded authentication token and an MD5 signature, which must be recreated exactly in Python using raw HTTP requests.

Key Steps Performed

Portal Analysis

Used browser Developer Tools to inspect the login request.

Identified the endpoint:
POST /api/login

Captured the request payload structure and headers.

JavaScript Reverse Engineering

Located the login logic inside the page’s inline script.

Extracted the algorithm:

auth_token = Base64(username + "::" + password)

timestamp = current UNIX time (seconds)

signature = MD5(auth_token + timestamp + APP_SECRET)

Determined the static secret used for signing:
APP_SECRET = "X9_Pc3_Salt_v2"

Python Implementation

Recreated the same logic using:

base64 for token generation

hashlib.md5() for signature creation

requests for sending the POST request

Ensured a non-browser User-Agent so the server reveals the flag.

Successful Login

Sent the correct JSON payload:

{
  "auth_token": "<base64>",
  "timestamp": "<unix_ts>",
  "signature": "<md5_hash>"
}


Received a successful response including the flag.

Key Outcome

A working Python script (solve.py) was created that:

Correctly replicates the portal's login crypto logic,

Authenticates as intern / 1234,

Retrieves and prints the server’s success message and flag.

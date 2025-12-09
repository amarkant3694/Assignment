# 🔐 Secure Portal Login Automation (Reverse Engineering Project)

This repository contains a Python-based solution to authenticate into a secure internal portal by reverse engineering the front-end JavaScript responsible for generating the authentication token and signature. The script successfully logs in as the `intern` user and retrieves the hidden flag that is only available when the request is made programmatically.

---

## 📌 Project Overview

The portal is located at:

```
http://51.195.24.179:5005
```

The challenge required:

- Reverse engineering the client-side authentication mechanism  
- Understanding how the portal signs login requests using JavaScript  
- Reproducing the exact logic in Python using `requests`  
- Logging in as `intern / 1234`  
- Extracting the flag returned only to non-browser clients  

No browser automation (Selenium/Playwright) was allowed — only raw HTTP requests.

---

## 🕵️‍♂️ Reverse Engineering Summary

Using Developer Tools, the login request was captured:

```json
{
  "auth_token": "aW50ZXJuOjoxMjM0",
  "timestamp": "1765288470",
  "signature": "7cdf55a72b4cd223407124602dc43f76"
}
```

The JavaScript generating this payload:

```js
const APP_SECRET = "X9_Pc3_Salt_v2";

const rawAuth = u + "::" + p;
const token = btoa(rawAuth);

const ts = Math.floor(Date.now() / 1000).toString();

const rawSign = token + ts + APP_SECRET;
const sign = CryptoJS.MD5(rawSign).toString();
```

### ✔ Final Authentication Algorithm

1. `auth_token = Base64(username + "::" + password)`
2. `timestamp = UNIX time (seconds)`
3. `signature = MD5(auth_token + timestamp + APP_SECRET)`
4. Send request to `POST /api/login` with appropriate headers

---

## 🧠 Insight

Although the portal labels this as "secure authentication," the mechanism relies only on:

- Base64 encoding (not encryption)  
- MD5 hashing (cryptographically broken)  
- A static secret stored in frontend JS  

These can be reproduced easily outside the browser.

---

## 🚀 Python Solution (`solve.py`)

```python
import base64
import hashlib
import time
import requests

BASE_URL = "http://51.195.24.179:5005"
LOGIN_PATH = "/api/login"

USERNAME = "intern"
PASSWORD = "1234"
APP_SECRET = "X9_Pc3_Salt_v2"


def make_auth_token():
    raw = f"{USERNAME}::{PASSWORD}"
    return base64.b64encode(raw.encode()).decode()


def make_timestamp():
    return str(int(time.time()))


def make_signature(token, ts):
    raw = token + ts + APP_SECRET
    return hashlib.md5(raw.encode()).hexdigest()


def solve():
    session = requests.Session()

    token = make_auth_token()
    ts = make_timestamp()
    sign = make_signature(token, ts)

    payload = {
        "auth_token": token,
        "timestamp": ts,
        "signature": sign,
    }

    headers = {
        "Content-Type": "application/json",
        "X-Client-Version": "2.1.0",
        "User-Agent": "python-requests/intern-task",
    }

    url = BASE_URL + LOGIN_PATH

    resp = session.post(url, json=payload, headers=headers)

    try:
        data = resp.json()
    except:
        print("Non-JSON response:", resp.text)
        return

    if data.get("success"):
        print("[+] Login success!")
        print("Message:", data.get("message"))
        print("Flag:", data.get("flag"))
    else:
        print("[-] Login failed:", data)


if __name__ == "__main__":
    solve()
```

---

## ▶️ How to Run

### 1. Install dependencies
```bash
pip install requests
```

### 2. Run the script
```bash
python solve.py
```

### 3. Expected Output

```
[+] Login success!
Message: Welcome, intern!
Flag: FLAG{...}
```

---

## 📂 Project Structure

```
├── solve.py
├── README.md
└── screenshots/   ← (optional, add proof screenshots here)
```

---

## 🛡️ Security Notes

This challenge demonstrates common web security issues:

- Secrets stored in frontend JavaScript  
- MD5 hashing instead of secure algorithms  
- Base64 encoding used as “protection”  
- No backend-side encryption or integrity validation  

---

## ✅ Conclusion

This project successfully:

- Reverse engineered the login mechanism  
- Reproduced the signing logic in Python  
- Authenticated programmatically  
- Retrieved the hidden flag  

It demonstrates strong skills in:

- JavaScript reverse engineering  
- HTTP request automation  
- Understanding weak authentication mechanisms  
- Python scripting  

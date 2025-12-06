# Nitro Automation - Writeup

**Category:** Scripting / Automation  
**Language:** Python  

---

## 1. Challenge Overview
The challenge provided a brief requiring us to interact with a hidden API. The goal was to visit a specific endpoint, retrieve a random string, process it according to specific rules, and submit it back to the server.

**Constraints:**
1.  **Strict Time Limit:** The server enforces a tight timeout. Manual copy-pasting is impossible; the server returns a "Too Slow" message.
2.  **Session Persistence:** The challenge mentioned a "strict per-session timer." This means we cannot use one tool (like curl) to fetch the task and another (like a browser) to submit it. We must maintain the session cookies (Session ID) across requests.

<img width="777" height="438" alt="Screenshot 2025-12-06 100149" src="https://github.com/user-attachments/assets/e883a13a-4a83-4982-a828-5f701d8f2206" />


---

## 2. The Task Workflow
Upon analyzing the endpoint `http://15.206.47.5:9090/task`, the workflow was identified as follows:

1.  **GET Request:** Connect to `/task` to generate a random string.
2.  **Extraction:** The string is embedded in the HTML response: `Here is the input string: [RANDOM_STRING]`.
3.  **Transformation:**
    *   **Reverse** the string.
    *   **Base64 Encode** the reversed value.
    *   **Wrap** it in the format: `CSK__{payload}__2025`.
4.  **Submission:** POST the result to `/submit`.

**Crucial Discovery:**
Initial attempts to send the data as JSON failed (`400 Bad Request`). Error analysis implied the server expected **Raw Text** in the POST body, not a JSON object or Form Data.

---

## 3. Automation Logic
To solve this, I wrote a Python script.

*   **Library:** `requests` was used for HTTP communication. specifically `requests.Session()` to handle cookie persistence automatically.
*   **Parsing:** The `re` (Regular Expression) module was used to robustly extract the target string from the HTML.
*   **Processing:** Python's string slicing `[::-1]` and `base64` library handled the transformation.

### The Algorithm
1.  Initialize `requests.Session()`.
2.  Fetch the HTML from `/task`.
3.  Use Regex `r"Here is the input string:\s*([A-Za-z0-9]+)"` to grab the string.
4.  Reverse and Encode.
5.  Construct the flag string.
6.  POST the raw string back to `/submit` using the *same* session object.

---

## 4. The Exploit Script
Here is the final python script (`exploit.py`) used to capture the flag:

```python
import requests
import re
import base64
import sys

# Target Configuration
BASE_URL = "http://15.206.47.5:9090"
TASK_URL = f"{BASE_URL}/task"
SUBMIT_URL = f"{BASE_URL}/submit"

def solve_challenge():
    # 1. Initialize Session to maintain cookies
    session = requests.Session()
    
    try:
        # 2. Fetch Task
        print(f"[*] Fetching task from {TASK_URL}...")
        response = session.get(TASK_URL)
        
        if response.status_code != 200:
            print(f"[!] Failed to connect. Status: {response.status_code}")
            return

        # 3. Extract String using Regex
        # Looks for "Here is the input string: " followed by alphanumeric characters
        match = re.search(r"Here is the input string:\s*([A-Za-z0-9]+)", response.text)
        
        if match:
            random_string = match.group(1).strip()
            print(f"[*] Extracted: {random_string}")
        else:
            print(f"[!] Extraction failed. Response content:\n{response.text}")
            return

        # 4. Process (Reverse -> Base64)
        reversed_str = random_string[::-1]
        b64_encoded = base64.b64encode(reversed_str.encode('utf-8')).decode('utf-8')
        
        # 5. Wrap Payload
        final_payload = f"CSK__{b64_encoded}__2025"
        print(f"[*] Payload: {final_payload}")

        # 6. Submit Raw Payload
        print(f"[*] Submitting RAW payload to {SUBMIT_URL}...")
        post_response = session.post(SUBMIT_URL, data=final_payload)
        
        print("-" * 30)
        print(f"[*] Status: {post_response.status_code}")
        print(f"[*] Response: {post_response.text}")
        print("-" * 30)

    except Exception as e:
        print(f"[!] Error: {e}")

if __name__ == "__main__":
    solve_challenge()
```

---

## 5. Result
I ran the script from the terminal. It successfully parsed the string, performed the calculations, and submitted the answer within the time window.
<img width="1000" height="192" alt="Screenshot 2025-12-06 101023" src="https://github.com/user-attachments/assets/43b15731-a043-4b0e-9dce-c4cf9ae39da4" />


**Flag:** `CL0UdSEk_ReSeArCH_tEaM_CTF_2025{ab03730caf95ef90a440629bf12228d4}`

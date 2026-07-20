# NovaEnergy Challenge

**Platform:** Hack The Box

**Author:**  0xL0m4

**Category:** Web (Easy)

<img width="1572" height="187" alt="image" src="https://github.com/user-attachments/assets/684b8a22-10dd-48e0-9c06-d1d43bb481b1" />


## Tools Used

* Browser Developer Tools
* Burp Suite
* ffuf


## Steps


### 1. Analyzing the JavaScript Files

I started by opening the website and analyzing the JavaScript files.

I found three js files.

After diving into the source code, I understood the authentication flow:
```
Register → Verify Email → Generate Flask Token
```

The application required users to register before accessing the dashboard.

### 2. Registering a New Account

I tried to register with an email, but the registration failed.

The application required an internal email address ending with @gonuclear.com

While exploring the website, I noticed an email address in the footer:

```
it-support@gonuclear.com
```

So I decided to register using it.

The registration was successful, but I still needed to verify the email address.

### 3. Finding Hidden API Endpoints

Since I needed the verification link, I started looking for hidden endpoints.

I used ffuf to enumerate API routes:

```
ffuf -u "http://htb.htb:32316/FUZZ" -w /usr/share/wordlists/seclists/Discovery/Web-Content/api/api-endpoints.txt
```
Result:
```
/api/docs
```

### 4. Exploring API Documentation

After opening /api/docs, I found the available API endpoints.

One interesting endpoint was:
```
/api/userDetails
```

The endpoint only required the email address.

I tried requesting it with:
```
it-support@gonuclear.com
```

The response was:
```
{
  "id": 2,
  "email":"it-support@gonuclear.com",
  "is_verified":false,
  "created_at":"2026-07-20T02:04:15.310527",
  "verifyToken":"a57413c3-efff-44b4-ad59-b769a832a354",
  "last_login":null
}
```

### 5. Bypassing Email Verification

My first instinct was to try changing:
```
"is_verified": false
```

to:
```
"is_verified": true
```

but it did not work.

The value remained false.

However, while checking the response again, I noticed something interesting:
```
"verifyToken":"a57413c3-efff-44b4-ad59-b769a832a354"
```

The verification token was exposed.

<img width="1261" height="780" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_20_07_2026_05_29_49" src="https://github.com/user-attachments/assets/bb811d39-edee-4c8b-87fb-ca464845fc56" />


The email verification endpoint required email and token

So I used the leaked token.

The response was:

{
  "message":"Email verified successfully"
}

The account was successfully verified.

<img width="1255" height="759" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_20_07_2026_05_36_05" src="https://github.com/user-attachments/assets/5310eaf7-3b10-4148-a07f-77af1531ae00" />


### 6. Logging In

After verifying the email, I logged in successfully.

The dashboard became accessible.

While exploring the dashboard, I found an upload feature.

There was an uploaded file:
```
flag.txt
```
### 7. Retrieving the Flag

I downloaded the file and checked its content:
```
cat flag.txt
```

Voilaaaa!!!

The flag was successfully retrieved.

### The Flag
```
HTB{g00d_j0b_r3g1str4ti0n_byp4s5}
```

[The challenge link](https://app.hackthebox.com/challenges/NovaEnergy?tab=play_challenge)

See you in the next writeup, in sha Allah!

سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.


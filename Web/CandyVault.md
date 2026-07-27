# CandyVault Challenge

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web ( Very Easy)

<img width="1585" height="172" alt="image" src="https://github.com/user-attachments/assets/d4aa42a2-ff34-4232-b66f-d3a605a29504" />


## Tools Used

* Browser Developer Tools
* Burp Suite

## Steps

### 1. Analyzing the Website

First, I opened the website and downloaded the source code.

The application contained a login page.

My first thought was that this could be a SQL Injection vulnerability, so I tried using sqlmap to test the login functionality.

However, it was not useful and I did not find any SQL injection.

### 2. Analyzing the Source Code

I went back to the source code and started analyzing the application logic.

I noticed that the application was using MongoDB as the database.

So instead of SQL Injection, I redirected my attention to NoSQL Injection.

The vulnerable code was:

```
user = users_collection.find_one({"email": email, "password": password})
```

<img width="1920" height="708" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_27_07_2026_22_56_28" src="https://github.com/user-attachments/assets/6700de8e-8853-4c94-a251-3210f56a3875" />

The query was directly using user-controlled input.

This means we can manipulate the MongoDB query.

### 3. Exploiting NoSQL Injection

The idea is to use MongoDB operators such as $ne which means "not equal".

By sending a condition that is always true, we can bypass the login check.

The payload:
```json
{
  "email": {
    "$ne": null
  },
  "password": {
    "$ne": null
  }
}
```
Don't forget to change the request content type to JSON

### 4. Retrieving the flag

After sending the payload, the authentication was bypassed successfully.

I was redirected to the admin page.

Voilaaaaaa!

The flag was successfully retrieved.

<img width="1259" height="717" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_27_07_2026_09_55_33" src="https://github.com/user-attachments/assets/6c70ae77-00c2-41d7-81a3-e8206c1c834d" />


### The Flag
```
HTB{s4y_h1_t0_th3_c4andy_v4u1t!}
```

#

[The challenge link](https://app.hackthebox.com/challenges/CandyVault?tab=play_challenge)

See you in the next write-up, In sha Allah!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

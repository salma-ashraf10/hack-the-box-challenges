# Trapped Source Challenge

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web (Very Easy)

<img width="1592" height="187" alt="image" src="https://github.com/user-attachments/assets/07241cfb-a95e-48b7-8417-cb5dd439fc9e" />


## Tools Used

* Browser Developer Tools (Inspect)

## Steps

### 1. Inspecting the Website

First, I opened the website and found a PIN code lock.

The application only allows me to enter a four digit PIN.

My first instinct was to try brute forcing the PIN.

But it turned out to be much easier than that.


### 2. Finding the PIN Code

I opened the browser's Developer Tools and inspected the page source.

While looking through the JavaScript, I found the correct PIN code inside a script tag: 4451

<img width="1920" height="795" alt="image" src="https://github.com/user-attachments/assets/25452e16-cfa5-40c1-9a05-87f1c2760a4f" />


### 3. Retrieving the Flag

The PIN was accepted successfully, and the application retrieved the flag.

Voilaaaaa!!

The flag was successfully retrieved.

<img width="1920" height="780" alt="image" src="https://github.com/user-attachments/assets/b1a916f2-0032-49a9-b39d-28124ab60b59" />


### The Flag

```
HTB{vi3w_cli13nt_s0urc3_S3cr3ts!}
```

#

[The challenge link](https://app.hackthebox.com/challenges/Trapped%2520Source?tab=play_challenge)

See you in the next writeup, In sha Allah!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.


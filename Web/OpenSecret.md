<h1 style="color:blue;"> OpenSecret challenge</h1>

  Platform: Hack The Box
  
  Author: 0xL0m4
  
  category: web (very easy)
  
  <img width="1546" height="197" alt="image" src="https://github.com/user-attachments/assets/4549735d-0942-4168-862c-4faf97c6949f" />


## Challenge Description: 
    
A simple help desk portal where users can submit support tickets. The application uses JWT tokens for session management, but something seems off about how they're implemented. Can you find the security flaw?


## Steps:

### 1. open the website and open dev tools (inspect).

### 2. scroll until finding script tag.

### 3. Voila , the flag appeared!!!

<img width="1431" height="787" alt="image" src="https://github.com/user-attachments/assets/2c6bbcf6-3f61-45c0-9c17-ac31b7099a7f" />


## Note

It is so easy . But, it learned us that may be developers forgot very important information in client-side code such as api keys.
Sensitive data should never be stored or exposed in frontend code.

### The flag: 

    HTB{0p3n_s3cr3ts_ar3_not_s3cr3ts}
     
[The challenge link](https://app.hackthebox.com/challenges/OpenSecret?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك لا إله إلا أنت أستغفرك وأتوب إليك

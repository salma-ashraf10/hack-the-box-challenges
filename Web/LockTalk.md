<h1 style="color:blue;">LockTalk Challenge</h1>

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web (Medium)

<img width="1586" height="187" alt="image" src="https://github.com/user-attachments/assets/b5da101c-7b69-4413-8b7d-09cef7216b13" />


## Challenge Description

In "The Ransomware Dystopia," LockTalk emerges as a beacon of resistance against the rampant chaos inflicted by ransomware groups. In a world plunged into turmoil by malicious cyber threats, LockTalk stands as a formidable force, dedicated to protecting society from the insidious grip of ransomware. Chosen participants, tasked with representing their districts, navigate a perilous landscape fraught with ethical quandaries and treacherous challenges orchestrated by LockTalk. Their journey intertwines with the organization's mission to neutralize ransomware threats and restore order to a fractured world. As players confront internal struggles and external adversaries, their decisions shape the fate of not only themselves but also their fellow citizens, driving them to unravel the mysteries surrounding LockTalk and choose between succumbing to despair or standing resilient against the encroaching darkness.

## Tools Used

- Burp Suite
- Vs code

# Steps

## 1. Exploring the application

After opening the website, I downloaded the provided source code and started reviewing it.

While reading the source code, I found that the flag endpoint permits the user who has the administrator role only.

The application determines the user's role from a JWT.

<img width="1852" height="533" alt="image" src="https://github.com/user-attachments/assets/9a63d11c-fe49-4b8a-9cc3-a2f84729a6f1" />

## 2. Finding the vulnerable JWT library

Inside requirements.txt I noticed:

```
python_jwt==3.3.3
```

Searching for this version revealed a known vulnerability:

**CVE-2022-39227**

This vulnerability allows an attacker to modify the JWT payload without invalidating the signature because the library fails to properly verify it under specific conditions.

A script is available for this  vulnerability here:

[PoC](https://github.com/user0x1337/CVE-2022-39227)

So:

- Obtain any valid JWT.
- Modify its payload to change the role to administrator.
- Use the forged token to access the flag endpoint.

## 3. Finding a valid JWT

While reviewing the API routes, I found this endpoint:

```
@api_blueprint.route('/get_ticket', methods=['GET'])
def get_ticket():

    claims = {
        "role": "guest",
        "user": "guest_user"
    }

    token = jwt.generate_jwt(
        claims,
        current_app.config.get('JWT_SECRET_KEY'),
        'PS256',
        datetime.timedelta(minutes=60)
    )

    return jsonify({'ticket': token})
```

This endpoint generates a valid JWT for a guest user.

Requesting it returned:

```
Unauthorized
```

## 4. Why was it forbidden?

I searched through the JavaScript files and found the request being made normally:

```javascript
$('#get_ticket_btn').on('click', function() {
    $.ajax({
        url: '/api/v1/get_ticket',
        type: 'GET',
        ...
    });
});
```

This suggested that the endpoint itself probably wasn't protected by the backend.

So I continued reviewing the infrastructure configuration.

## 5. Discovering the HAProxy restriction

Inside the HAProxy configuration I found:

```
frontend haproxy
    bind 0.0.0.0:1337
    default_backend backend

    http-request deny if { path_beg,url_dec -i /api/v1/get_ticket }
```

The request was being blocked before reaching the backend.

The Dockerfile also revealed the HAProxy version:

```
haproxy-2.8.1
```

Searching for vulnerabilities affecting this version led me to:

**CVE-2023-45539**

[Reference](https://www.sentinelone.com/vulnerability-database/cve-2023-45539/#cq=i1)


## 6. Bypassing the HAProxy rule

The vulnerability allows bypassing certain ACL rules.

Appending a # to the URL caused the backend to receive the request while bypassing HAProxy's restriction.

```
GET /api/v1/get_ticket#
```

Yes, the server returned a valid JWT.

<img width="1256" height="728" alt="image" src="https://github.com/user-attachments/assets/7d2e6b03-6c98-4961-bbfd-4ac33981bb17" />

## 7. Forging an administrator token

Using the public exploit for CVE-2022-39227, I modified the JWT payload.

Original payload:

```
{
     .....
    "role": "guest",
    "user": "guest_user"
     .....
}
```

Modified payload:

```
{
    "role": "administrator",
    "user": "guest_user"
}
```

The exploit generated a forged authorization token.

<img width="1927" height="394" alt="image" src="https://github.com/user-attachments/assets/77e92d84-637c-4828-887a-e0b6e058e4e6" />


## 8. Retrieving the flag

Finally, I added the forged token in the Authorization header while requesting the flag endpoint:

```
GET /api/v1/flag HTTP/1.1
Host: htb
Authorization: <JWT Token>
```

Voilaaaa!!!

The flag was successfully retrieved.

<img width="1269" height="685" alt="image" src="https://github.com/user-attachments/assets/68ea6001-9f88-4d98-8a7c-61511f8ab35e" />


## The Flag

```
HTB{h4Pr0Xy_n3v3r_D1s@pp01n4s_4t_bugg5_4nd_h4ck5}
```
#

[The challenge link](https://app.hackthebox.com/challenges/LockTalk?tab=play_challenge)

See you in the next writeup, In sha Allah!

سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

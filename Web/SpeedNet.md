<h1 style="color:blue;">SpeedNet Challenge</h1>

**Platform**: Hack The Box

**Author**: 0xL0m4

**Category**: Web (Easy)

<img width="1617" height="159" alt="image" src="https://github.com/user-attachments/assets/919b2a03-75bd-4ba9-a79f-e70e1b527b37" />



## Challenge Description

Speednet is an ISP platform. Join our bug bounty to find vulnerabilities and retrieve the hidden flag. 

Test using the email service at http://IP:PORT/emails/ with address test@email.htb.

## Tools Used

* Burp Suite
* Python
* Browser Developer Tools

## Steps

### 1. Registering a user

The challenge provided a test account email:

```
test@email.htb
```

I registered as a normal user and started exploring the application.

After inspecting the requests in Burp Suite, I noticed that the backend was using GraphQL instead of traditional REST endpoints.

Whenever I encounter GraphQL during a challenge, my first objective is to understand the schema and available operations because GraphQL often exposes functionality that is not visible through the web interface.

### 2. Checking whether introspection is enabled

The first thing I tested was whether GraphQL introspection was enabled.

I started with a simple schema query using Graphql extension in Burp Suite :

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

The request succeeded, confirming that introspection was enabled.

<img width="1668" height="816" alt="image" src="https://github.com/user-attachments/assets/2cda51a7-12f3-4af8-80d6-a418c110ba4a" />


Since introspection was available, I used the standard GraphQL introspection query to enumerate all available types, queries, mutations, and fields.

```graphql
query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    types {
      ...FullType
    }
  }
}
```

The response revealed a large portion of the application's GraphQL schema.

<img width="1672" height="815" alt="image" src="https://github.com/user-attachments/assets/56adf1db-e844-4e20-9fd1-be0b52dc7655" />


### 3. Discovering an IDOR vulnerability

While reviewing the schema, I found a query named:

```
GetUserProfile
```

The query accepted a user ID as input and returned profile information such as:

* Email
* Address
* User details

I first queried my own profile to understand the response structure.

Then I changed the user ID parameter from my own user ID to:

```
1
```

The request returned another user's information instead of denying access.

This confirmed the presence of an IDOR (Insecure Direct Object Reference) vulnerability.

The response exposed the administrator's details, including the email address:

```
admin@speednet.htb
```

This email became very important during the next stage.

<img width="1672" height="821" alt="image" src="https://github.com/user-attachments/assets/9c2b39a4-2540-45d2-a668-cc0c1014e2f0" />


### 4. Enumerating password reset functionality

Continuing to review the schema, I noticed two interesting mutations:

```
devForgetPassword
resetPassword
```

The names immediately suggested a password reset workflow.

The first mutation:

```
devForgetPassword
```

accepted an email address and returned a password reset token.

<img width="1672" height="771" alt="image" src="https://github.com/user-attachments/assets/3bea6997-8b5c-4481-98e1-af2a7bcf8b56" />


The second mutation:

```
resetPassword
```

accepted:

* Token
* New password

Using the administrator email obtained through the IDOR vulnerability:

```
admin@speednet.htb
```

I requested a reset token through the development password reset functionality.

The token was returned directly in the response.

I then used that token with the `resetPassword` mutation and successfully changed the administrator password.

<img width="1670" height="812" alt="image" src="https://github.com/user-attachments/assets/79ab8093-5d9b-40f9-91d5-60a9717e0ebe" />


### 5. Logging in as administrator

With the new credentials, I attempted to log in as the administrator.

The login was successful, but the application immediately requested a two-factor authentication code.

At this point, I needed to determine whether brute-forcing the OTP would be feasible.

### 6. Analyzing the OTP system

The challenge provided access to a testing mailbox:

```
/emails
```

I logged in using the test account and requested a verification code.

The mailbox displayed the received OTP message.


* The OTP consisted of 4 digits.
* The OTP remained valid for 5 minutes.

<img width="1920" height="854" alt="image" src="https://github.com/user-attachments/assets/bd14cb13-ce7c-44b0-86b4-49f42fa038c6" />


This meant the search space was:

```
0000 - 9999
```

Only 10,000 possible combinations.

### 7. Discovering the OTP verification mutation

Using Burp Suite, I intercepted the authentication requests.

I found a mutation named:

```
verifyTwoFactor
```

The mutation accepted:

* User token
* OTP code

and returned a valid session token when the OTP was correct.

I logged in again as the administrator and captured the request.

The temporary authentication token required by the mutation was present in the request body.

<img width="1658" height="821" alt="image" src="https://github.com/user-attachments/assets/dee76967-6760-4d9e-be39-441556451a42" />


### 8. Brute-forcing the OTP

I created a Python script that:

* Accepts the target URL.
* Accepts the temporary authentication token.
* Iterates from 0000 to 9999.
* Sends requests to the `verifyTwoFactor` mutation.

  **The script**
  ```python
  import requests
  import sys
  
  url = sys.argv[1]
  token = sys.argv[2]
  
  def make_request(start, end):
      query = "mutation{"
      for i in range(start, end):
          query += f'v{i}: verifyTwoFactorAuthentication(token: "{token}" , otp:    "{i:04}"){{token}}'
      query += "}"
      data = {'query': query}
      return requests.post(url, headers={'Content-Type': 'application/json'}, json=data)
  
  step =1000
  for i in range(0, 10000 , step):
      print(f'{i}-{i+step}')
      respon = make_request(i , i+step)
      print ({k:v for k , v in respon.json().get('data').items() if v})
    
  ```

After a short time, the correct OTP was found.

The server responded with a valid administrator session token.

Most importantly, this token granted administrator access without requiring the email or password again.

<img width="1008" height="610" alt="image" src="https://github.com/user-attachments/assets/647f12af-e498-4fa9-9a5a-2bed323e5f9b" />


### 9. Taking over the administrator session

I returned to my normal tester account and logged in legitimately.

Then I opened the browser developer tools and navigated to:

```
Local Storage
```

The application stored the current session token there.

I replaced my tester token with the administrator token obtained from the OTP brute force.

After refreshing the page, the application treated me as the administrator.

Voilaaa!

I now had full administrator access.

### 10. Getting the flag

After browsing the administrator dashboard, I navigated to:

```
Billing
```

The flag was displayed there.

<img width="1920" height="782" alt="image" src="https://github.com/user-attachments/assets/9b661021-df93-44b1-a3c2-dc1e0d50a9f3" />

## The Flag

```
HTB{gr4phql_3xpl01t_1n_a_nutsh3ll}
```
[The challenge link](https://app.hackthebox.com/challenges/SpeedNet?tab=play_challenge)

See you in the next write-up!

سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

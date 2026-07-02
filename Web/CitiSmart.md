# CitiSmart Challenge

**Platform:** Hack The Box

**Author:** 0xL0m4

**Category:** Web (Easy)

<img width="1541" height="180" alt="image" src="https://github.com/user-attachments/assets/fe5ad5dd-32f7-4d80-86cc-578f20b263ef" />


## Challenge Description

Citismart is an innovative Smart City monitoring platform aimed at detecting anomalies in public sector operations. 

We invite you to explore the application for any potential vulnerabilities and uncover the hidden flag within its depths.


## Tools Used

* Burp Suite
* Browser Developer Tools


## Steps

### 1. Bypassing the login page

I started by opening the website. I found a login page.

I entered random credentials and intercepted the request using Burp Suite.

The response contained an interesting parameter:

```json
{
  "isLogin": false
}
```

<img width="1255" height="770" alt="image" src="https://github.com/user-attachments/assets/4b336f92-e1c3-4b1a-a805-a5ccef0bc4d3" />


The first thing that came to mind was: what happens if I change it to true?

To test this, I send the request again to intercept tab and right click + do intercept then close the intercept and open again --> the request appear and you can edit on the response now:

```json
{
  "isLogin": false
}
```

to:

```json
{
  "isLogin": true
}
```

<img width="1360" height="829" alt="image" src="https://github.com/user-attachments/assets/c709a880-eb06-4f30-a27c-6558b3b0b9c5" />

# 

<img width="1603" height="762" alt="image" src="https://github.com/user-attachments/assets/1835965b-e4b0-44e6-ae45-05bb07267446" />


and forwarded the response.

I was successfully redirected to the dashboard without valid credentials.

###### Why this happened?

This behavior suggests that the frontend relied on the isLogin value returned by the server to determine whether the user should be redirected to the dashboard.

<img width="1920" height="708" alt="image" src="https://github.com/user-attachments/assets/a28ad4d9-28e8-4679-9788-064f47eba871" />


### 2. Exploring the dashboard

After bypassing authentication, I found two main sections:

* Dashboard
* Manage

The Manage section allows users to:

* Add endpoints
* Delete endpoints
  
<img width="1920" height="786" alt="image" src="https://github.com/user-attachments/assets/f742dc63-2dd1-4e59-89cb-a06d9201e20c" />

When creating a new endpoint, I noticed the application was communicating with:

```
/api/dashboard/endpoints
```

So I decided to inspect the JavaScript files for additional API routes.

I found the following endpoints:

```
/api/dashboard/endpoints
/api/auth/login
/api/auth/me
/api/auth/logout
/api/dashboard/metrics
```

<img width="1920" height="780" alt="image" src="https://github.com/user-attachments/assets/06fdbcf9-25b3-446a-8ac0-801d1ac6012e" />


### 3. Inspecting metrics

The most interesting endpoint was:

```
GET /api/dashboard/metrics
```

The response contained monitoring information for all previously created endpoints.

At this point, it looked like the application was collecting data from user-defined endpoints and displaying the results through the metrics API.

<img width="1258" height="598" alt="image" src="https://github.com/user-attachments/assets/7fa39422-686e-42a7-aae6-529950578305" />


### 4. Looking for SSRF

During reconnaissance, I noticed that another service was running internally:

```
CouchDB (Port 5984)
```

Before testing, I spent a few minutes learning how CouchDB works.

Apache CouchDB is a document-oriented NoSQL database that:

* Stores data in JSON format
* Usually runs on port 5984

One useful endpoint is:

```
/_all_dbs
```

which returns all available database names.

Since the application allowed me to create arbitrary endpoints, I wondered if it was fetching data from them server-side.



### 5. Confirming SSRF

I added a new endpoint pointing to:

```
http://127.0.0.1:5984
```

The response returned:

```
500 Internal Server Error
Request failed with status code 404
```

Then I tested another internal IP:

```
http://10.0.0.1
```

This time the response was:

```
500 Internal Server Error
timeout of 5000ms exceeded
```

Interesting!

The error messages changed depending on the target.

This confirmed a blind SSRF vulnerability.

The blind SSRF:

Initially, the SSRF appeared blind because the application only returned error messages.

However, the /metrics endpoint later exposed the data collected from the configured endpoints, effectively turning the SSRF into a useful data extraction primitive.

[Useful Article](https://wpsites.ucalgary.ca/jacobson-cpsc/2024/10/29/exploring-blind-ssrf-server-side-request-forgery-and-mitigations/)



### 6. Enumerating internal databases

Now that SSRF was confirmed, I targeted CouchDB's database enumeration endpoint:

```
http://127.0.0.1:5984/_all_dbs
```

The endpoint was accepted successfully.

After that, I checked the metrics endpoint again:

```
GET /api/dashboard/metrics
```

The response now contained:

```
{
  "flag": "[\"citismart\"]"
}
```

<img width="1267" height="765" alt="image" src="https://github.com/user-attachments/assets/f874d368-d48b-4ac5-b117-e402999d0533" />


This revealed the existence of an internal database named:

```
citismart
```


### 7. Accessing the database

The next objective was retrieving data from the database.

I first tried:

```
http://127.0.0.1:5984/citismart/FLAG
```

but received:

```
Request failed with status code 404
```

After some testing, I discovered that appending a special character bypassed the filtering:

```
http://127.0.0.1:5984/citismart/FLAG?
```

Alternatively:

```
http://127.0.0.1:5984/citismart/FLAG#
```

also worked.

The endpoint was accepted and stored by the application.

<img width="1252" height="744" alt="image" src="https://github.com/user-attachments/assets/af5beff9-1359-4aae-8ac6-246d0f3b0ddd" />



### 8. Retrieving the flag

After adding the endpoint, I queried the metrics endpoint one final time:

```http
GET /api/dashboard/metrics
```

This time the response revealed:

```
{
  "flag":"FLAG\",\"_rev\":\"1-ee80946409fe94e517fdfd32b4d96d3c\",\"value\":\"FLAG=HTB{sm4rt_cit1_but_n0t_s3cur3}\"}"
}
```

Voilaaa!

The flag was successfully retrieved.

<img width="1254" height="799" alt="image" src="https://github.com/user-attachments/assets/43867f59-2f61-4663-8a27-4932c267fcf5" />


### The Flag

```
HTB{sm4rt_cit1_but_n0t_s3cur3}
```

#

[The challenge link](https://app.hackthebox.com/challenges/CitiSmart?tab=play_challenge)

See you in the next writeup!
 
.سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك

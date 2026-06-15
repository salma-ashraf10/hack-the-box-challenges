<h1 style="color:blue;">Secure notes challenge</h1>

  Platform: Hack The Box
  
  Author: 0xL0m4
  
  category: web (easy)





## Challenge Description: 
    
We built this note-taking app to be so simple, there can't possibly be any bugs. We even added a door to claim the flag. However, only those who knock from inside may enter!

## Tools Used

  - Vs code
  - Burp Suite
  

## Steps:

### 1. Download the source code and open the website

After downloading and reviewing the source code, I noticed that the ip address must be internal "127.0.0.1" to get the flag.


### 2. The website use express and mongoose

While walking through dave into the code and intercept all of request , I realized that there are create , update endpoint for creating/updating notes.
The schema for creating notes is consist of title , content field.

### 3. Try checking prototype pollution
Mongoose set strict to true by default. So, it ingores the extra fields from client.
since there is update endpoint , there is possibility that the $rename field is setted. so , you can exploit it to rename the title and content field name.

### 3. Start to inject.
let's deal with the following
__proto__._pearname.address --> responsible for remote ip address.
__proto__._pearname.family --> we use it to identify that is ipv4.
So, how about swapping the current two fields name to these name. and before changing the name , we can set the ip and its type value usnig create endpoint.


### 4. Creat new note and update its field names
Create new note normally . The title value is "127.0.0.1" and the content value is "IPv4" .
Then update the title to __proto__._pearname.address and the content to __proto__._pearname.family using update endpoint.
```
"$rename":{
"title": "Object.prototype._peername.address",
"content":"Object.prototype._peername.family"
}
```
### 5. Get the flag
Send Get request to request the flag endpoint.
The flag successfully appeared in the response.

### The flag: 

    HTB{m0ng00s3_pr0t0typ3_p0llus10n_c0mb1n3d_w1th_1nt3rn4l_n0d3_g4dg3ts!}
     
[The challenge link](https://app.hackthebox.com/challenges/Secure%2520Notes?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك أشهد أن لا إله إلا أنت أستغفرك وأتوب إليك

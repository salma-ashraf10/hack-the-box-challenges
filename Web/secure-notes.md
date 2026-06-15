<h1 style="color:blue;">Secure notes challenge</h1>

  Platform: Hack The Box
  
  Author: 0xL0m4
  
  Category: Web (Easy)

<img width="1499" height="139" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_15_06_2026_20_41_16" src="https://github.com/user-attachments/assets/162d5dce-3abc-4f9f-ba42-744f9b859da2" />


## Challenge Description: 
    
We built this note-taking app to be so simple, there can't possibly be any bugs. We even added a door to claim the flag. However, only those who knock from inside may enter!

## Tools Used

  - Vs code
  - Burp Suite
  

## Steps:

### 1. Download the source code and open the website
After downloading and reviewing the source code, I noticed that access to the /flag endpoint is restricted based on the client's IP address.

The condition checks whether the request is coming from localhost.

<img width="1278" height="213" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_15_06_2026_20_47_12" src="https://github.com/user-attachments/assets/c839d64b-426d-4e33-b07c-8afd76fa621b" />



### 2. The website uses Express and Mongoose.

While walking through dave into the code and intercept all requests , I realized that there are create , update endpoint for creating/updating notes.

The schema for creating notes consists of title and content fields.

<img width="607" height="355" alt="image" src="https://github.com/user-attachments/assets/6960af28-8b21-4cf2-8c05-67c4ea00a904" />


### 3. Try checking prototype pollution
Mongoose set strict to true by default. So, it ignores the extra fields from client.

Since there is an update endpoint, there is a possibility that the $rename field is set. so , you can exploit it to rename the title and content field name.

### 3. Start to inject.
Let’s consider the following:

\_\_proto__._pearname.address --> for ip address.

\_\_proto__._pearname.family -->we use it to identify that it is IPv4.

So, how about changing the current two fields name to these name. And before changing the name , we can set the ip and its type value using create endpoint.


### 4. Create a new note and update its field names
Create a new note normally. The title value is "127.0.0.1" and the content value is "IPv4" .

<img width="1254" height="588" alt="image" src="https://github.com/user-attachments/assets/0f096f91-6dd7-4327-9fba-a28f775844ba" />

Then update the title to \_\_proto__._pearname.address and the content to \_\_proto\_\_._pearname.family using update endpoint for the same note.

```
"$rename":{
"title": "__proto__._peername.address",
"content":"__proto__._peername.family"
}
```
<img width="1257" height="545" alt="image" src="https://github.com/user-attachments/assets/828432df-eea3-4fe0-9dd3-82b5130e8c04" />

### 5. Get the flag
Send Get request to request the flag endpoint.
The flag successfully appeared in the response.

<img width="1267" height="755" alt="image" src="https://github.com/user-attachments/assets/359c76a2-40e0-4659-b443-bda58d3ea96f" />


### The flag: 

    HTB{m0ng00s3_pr0t0typ3_p0llus10n_c0mb1n3d_w1th_1nt3rn4l_n0d3_g4dg3ts!}
     
[The challenge link](https://app.hackthebox.com/challenges/Secure%2520Notes?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك.

<h1 style="color:blue;">Flag Command challenge</h1>

  Platform: Hack The Box
  
  Author: 0xL0m4
  
  category: web (very easy)
  
<img width="1577" height="180" alt="image" src="https://github.com/user-attachments/assets/91e54d16-d814-4aa0-8734-7efc1722f596" />

## Challenge Description: 
    
Embark on the "Dimensional Escape Quest" where you wake up in a mysterious forest maze that's not quite of this world. 
Navigate singing squirrels, mischievous nymphs, and grumpy wizards in a whimsical labyrinth that may lead to otherworldly surprises. 
Will you conquer the enchanted maze or find yourself lost in a different dimension of magical challenges? The journey unfolds in this mystical escape!

## Tools Used

  Burp Suite

## Steps:

### 1. open the website and open dev tools (inspect).

### 2. scroll until finding js URLs.

<img width="1422" height="782" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_12_05_2026_16_28_25" src="https://github.com/user-attachments/assets/bc5528b7-0f0c-44b1-8655-6c91526c9f70" />


### 3. After opening the JavaScript files, nothing looked particularly interesting at first. But, focus a little bit. there is an api was called,
 which suggested that there might be additional hidden API endpoints running in the background.let's see it.

  <img width="1891" height="762" alt="image" src="https://github.com/user-attachments/assets/b4a75850-2a71-4b70-a954-5381a3e62ff7" />

      
### 4. You can inspect the requests using the Network tab in the browser Dev Tools, but I preferred using Burp Suite for easier analysis.


### 5. while browsing Proxy HTTP history in Burp Suite , I found an API called "/api/options".
when opening it , I found the secret message.

<img width="1651" height="856" alt="image" src="https://github.com/user-attachments/assets/e784c388-43f9-4ce3-86f8-de95ea01870b" />


### 6. Try to enter it in the input . voila , the hidden flag was appeared. 

<img width="828" height="80" alt="image" src="https://github.com/user-attachments/assets/cea18c52-f55a-445b-9089-dfe81eb40ace" />


### The flag: 

     HTB{D3v3l0p3r_t0015_4r3_b35t__t0015_wh4t_d0_you_Think??}
     
[The challenge link](https://app.hackthebox.com/challenges/Flag%2520Command?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك لا إله إلا أنت أستغفرك وأتوب إليك

<h1 style="color:blue;">Spookifier challenge</h1>

  Platform: Hack The Box
  
  Author: 0xL0m4
  
  category: web (very easy)
  
<img width="1566" height="197" alt="image" src="https://github.com/user-attachments/assets/09852711-cc59-466d-9a88-d31ab224b3c6" />


## Challenge Description: 
    
There's a new trend of an application that generates a spooky name for you. Users of that application later discovered that their real names were also magically changed, causing havoc in their life. Could you help bring down this application?

## Tools Used

  Vs code

## Steps:

### 1. Open the website and inspect the source code

After downloading and reviewing the source code, I noticed that the application takes user input and converts it into four different font styles.

### 2. I found user input that take any word and convert it to 4 type formatting.

### 3. While walking through the source code, I found a function called change_font() that take the input and convert it char by char.

  <img width="1499" height="720" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_12_05_2026_23_54_22" src="https://github.com/user-attachments/assets/2031ecf9-188d-4c53-866e-3dd54107e016" />

### 3. After change_font function called , there is a another function was called named generate_render that its functionality is rendering the text that user entered after change its font.
At this point, the SSTI idea became clear because the user input was rendered inside the template without proper sanitization. so , let's exploit it.

<img width="1662" height="685" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_12_05_2026_23_37_28" src="https://github.com/user-attachments/assets/fcf4d4fc-93fd-4a30-9592-a6281b2ee5fb" />


### 4. Before attempting exploitation, I needed to identify the template engine being used. 
It is so easy, you can scroll down. The Mako template engine was used.

<img width="1429" height="533" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_12_05_2026_23_52_51" src="https://github.com/user-attachments/assets/069ac1fb-f727-4042-8ecc-b52dcd866a43" />


### 5. So, let's come back to exploit it. first , I try to run id comman. The command executed successfully.

<img width="1696" height="854" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_12_05_2026_23_56_05" src="https://github.com/user-attachments/assets/42ff7c10-a791-40ce-bdb6-82c637c82a1d" />


### 6. Then I try to print flag file using cat commmand. Voila , The hidden flag was appeared!!!!

<img width="1884" height="861" alt="VirtualBox_kali-linux-2025 3-virtualbox-amd64_12_05_2026_23_36_59" src="https://github.com/user-attachments/assets/905d4e38-f780-48af-b1f4-a5cfb09cd65e" />
### Note

This article was very useful and contains payloads for multiple template engines:

[Hacktrick Article](https://hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html)

### The flag: 

    HTB{t3mp14+3_1nj3ct10n_C4n_ 3x1st5_4nywh343!!}
     
[The challenge link](https://app.hackthebox.com/challenges/Spookifier?tab=play_challenge)

See you in the next writeup!
 
سبحانك اللهم وبحمدك لا إله إلا أنت أستغفرك وأتوب إليك

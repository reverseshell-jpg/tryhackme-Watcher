TryHackMe — Watcher Writeup
Overview
Watcher is a medium Linux box built on a chained exploit path: an LFI leaks FTP creds, an uploaded reverse shell lands on www-data, and four separate misconfigurations (a sudo rule, a writable cron script, a hijackable Python import, and a world-readable backup key) pivot that foothold all the way to root.
Flag
User gained
Technique
flag_1
pre-auth
Initial web recon
flag_2
ftpuser
FTP login with LFI-leaked credentials
flag_3
www-data
LFI-triggered reverse shell uploaded via FTP (or LFI2RCE via PHP filter chain)
flag_4
toby
sudo -u toby /bin/bash
flag_5
mat
Cron job (cow.sh) overwritten by toby
flag_6
will
Python library hijack (cmd.py imported by will_script.py)
flag_7
root
SSH private key leaked via adm group read access
Each pivot abuses a different misconfiguration: a sudo rule, a writable cron script, a hijacked Python import, and a readable backup key.
1. Recon
The server has 3 open ports: 21 (FTP), 22 (SSH), and 80 (HTTP).
<img width="684" height="188" alt="image" src="https://github.com/user-attachments/assets/63e7af1a-736f-424c-a59d-cc37d1c73684" />
Used dirsearch to fuzz directories:
<img width="562" height="678" alt="image" src="https://github.com/user-attachments/assets/542bdf08-4e1a-42e5-87d5-70d147997ed3" />
Contents of robots.txt:
<img width="336" height="88" alt="image" src="https://github.com/user-attachments/assets/22fce656-924c-4c96-83c8-c90778de548c" />
I tried accessing the /secret_file_do_not_read.txt endpoint, but got a 403 Forbidden response.
Going back to the website's homepage, I noticed the site is vulnerable to LFI (reference).
<img width="355" height="38" alt="image" src="https://github.com/user-attachments/assets/d3b60aec-46a7-4e0e-badf-f1edb80266c5" />
<img width="1267" height="384" alt="image" src="https://github.com/user-attachments/assets/82915d1d-aee6-4267-83e3-6b65d69769f9" />
I then tried reading secret_file_do_not_read.txt via the LFI:
<img width="857" height="48" alt="image" src="https://github.com/user-attachments/assets/7c8e1760-24ee-4447-bf5b-90001310ce6f" />
From the file's content, we obtain FTP credentials, and this user has write access and can store files in /home/ftpuser/ftp/files. We can abuse this to upload a reverse shell and trigger it via the LFI.
2. Exploit
2.1 Read flag_1
<img width="2169" height="725" alt="image" src="https://github.com/user-attachments/assets/88051fc4-bca8-4d64-8027-cd3597e09c05" />
2.2 Read flag_2
Log into FTP with the credentials obtained above and read flag_2.txt:
<img width="2167" height="726" alt="image" src="https://github.com/user-attachments/assets/20d54e3d-fb22-4a1d-b33f-aff91fb3352d" />
2.3 Read flag_3
Navigate to the files directory and use the put command to upload a reverse shell:
<img width="387" height="112" alt="image" src="https://github.com/user-attachments/assets/afa2c2f8-2294-4335-bd65-72923c6c05a6" />
Use the LFI to trigger the payload:
<img width="1106" height="210" alt="image" src="https://github.com/user-attachments/assets/2438d987-11bf-48a1-9d51-5254906a5550" />
Use find to locate flag_3.txt and read it:
<img width="2170" height="725" alt="image" src="https://github.com/user-attachments/assets/db9db0dc-6a13-4bc1-a763-b6ddd41e51f7" />
Alternatively, we can use the LFI2RCE via PHP Filters technique to get a shell back (reference).
You can use synacktiv's tool to generate the payload: https://github.com/synacktiv/php_filter_chain_generator
<img width="1240" height="331" alt="image" src="https://github.com/user-attachments/assets/28aafd7d-9430-4ec9-905c-aa4e2ca3f859" />
<img width="527" height="51" alt="image" src="https://github.com/user-attachments/assets/7a44cfdd-a70b-47b7-8a03-1ee40fac6245" />
2.4 Read flag_4
Ran sudo -l to check for anything interesting:
<img width="776" height="116" alt="image" src="https://github.com/user-attachments/assets/0bee9174-fc66-438b-b0ac-74401ea27dca" />
From here we can see that we can operate on the system as the toby user.
Reading the flag:
<img width="531" height="36" alt="image" src="https://github.com/user-attachments/assets/13bda706-dc3d-4540-a700-d0e4d4ec9fe4" />
2.5 Read flag_5
Checked /etc/crontab:
<img width="379" height="33" alt="image" src="https://github.com/user-attachments/assets/c27c73d0-e746-4a89-9237-0bf830885270" />
<img width="578" height="33" alt="image" src="https://github.com/user-attachments/assets/f7ed4a4a-b8ff-4bda-a0a3-ab1db9e17d99" />
We can see that mat is scheduled to execute the cow.sh file.
So we can abuse the toby account to modify cow.sh and obtain a new shell as mat:
<img width="645" height="120" alt="image" src="https://github.com/user-attachments/assets/e5006615-3592-4756-84b9-480e18d8bde2" />
Reading the flag:
<img width="316" height="34" alt="image" src="https://github.com/user-attachments/assets/91b57f24-6e40-4373-a2d6-3f3de46aa4bd" />
2.6 Read flag_6
<img width="438" height="395" alt="image" src="https://github.com/user-attachments/assets/6a11bd04-8cdd-4bd8-8d4e-1ebef6ff32ad" />
<img width="777" height="115" alt="image" src="https://github.com/user-attachments/assets/51f6b65d-8ef9-4536-8aaa-7a2959630486" />
As before, we can execute /home/mat/scripts/will_script.py as the will user.
We can see that will_script.py imports cmd.py, and after checking, we find that the mat user has write access to cmd.py.
We can abuse this to inject a payload into cmd.py; when will_script.py imports and executes it, we get a shell as will:
<img width="781" height="27" alt="image" src="https://github.com/user-attachments/assets/9c8b1d41-3fd7-44fe-8be2-d5d041e6458d" />
<img width="558" height="126" alt="image" src="https://github.com/user-attachments/assets/6b09bdec-44e1-4a93-9e0d-9b982fe0d311" />
Reading the flag:
<img width="345" height="33" alt="image" src="https://github.com/user-attachments/assets/fa2b642c-cf97-4ae1-9749-889346e89247" />
2.7 Read flag_7
Checked id for will:
<img width="445" height="16" alt="image" src="https://github.com/user-attachments/assets/7080a6ee-26cf-4e17-86bb-b09d371c5c7f" />
We can see that will belongs to the adm group, one of the special groups that can read certain sensitive directories.
Used find to search for files:
<img width="535" height="152" alt="image" src="https://github.com/user-attachments/assets/62711068-b80f-4359-92f6-f6639ee57156" />
Here it looks like we found a file containing root's key:
<img width="331" height="18" alt="image" src="https://github.com/user-attachments/assets/bc2a5616-2e99-4961-a269-1384ee8863f3" />
Reading the file:
<img width="536" height="176" alt="image" src="https://github.com/user-attachments/assets/1abb9b25-8eac-4309-be7e-67eaeb6ebdb2" />
Excellent — it looks like we found the private key used for root's SSH login:
<img width="594" height="436" alt="image" src="https://github.com/user-attachments/assets/a646a2ff-df8a-431f-9634-73e79ae1ef12" />
Reading flag_7.txt:
<img width="264" height="21" alt="image" src="https://github.com/user-attachments/assets/415afb7f-8627-4272-846b-604f02b27035" />
Done!

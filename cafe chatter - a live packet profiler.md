---
tags:
  - note
date: 2025-06-08
---
```summary
a live packet capturing system using wireshark and a custom python script to consolidate ip and mac addresses with their unencrypted traffic. captured data is then analyzed using gemini to generate behavioural profiles, answering the question: "who is this?".

created as an investigation into real-world netowrk acitivity in public environments, exploring how much can be inferred from passive observation.
```

**Tools used:** 
- Wireshark / `tshark` / `airmon` suite for packet capture
- Kali Linux VM with VirtualBox to use monitor mode
- ALFA Wifi adapter to capture others' packets
- Custom Python scripts for analysis and clean up
- Gemini API for profiling
---
# technical details

>[!star] ethical note
>The exact details of my process (commands and more volatile instructions) are not published publicly to avoid misuse. I am happy to discuss details privately or in interview contexts. 

---
### wifi types observed
There are 3 types of WIFI that I looked at for this project.
1. **Open** - unencrypted; any unencrypted protocols can be seen (DNS, TCP, HTTP, etc) in full.
2. **WPA-Personal** - password protected
3. **WPA-Enterprise** - user/password protected

WPA-Personal is password protected. When connect, your device and the router do the handshake and establish a secret key that your traffic will be encrypted with. By intercepting the handshake and having the WIFI password, it is able to be decrypted as if there were no password.

WPA-Enterprise is user and password protected. Even with the 4-way handshake, you need the unique credentials for each account to be able to do any decryption, which is basically impossible to acquire passively.

>for this project the cafe I frequented used WPA-Personal with a publicly available password for patrons.

---
### capture and filtering
When you access a website and the IP address is not in your machine's cache, it will go to DNS server and ask for the IP. This request is typically unencrypted and can be quite frequent, so it was my primary profiling vector.

I used the aircrack-ng suite to locate active channels and collect information about the exact configuration of the WIFI and then started capture with `tshark`. Due to the high volume of traffic, I had a separate capture for handshakes, as not to miss them, and then stitched them onto the main pcaps after capture for decryption.

![[Pasted image 20250608232816.png|center]]
*Sample Wireshark decrypted capture with non-encrypted filters*

Working on-site, there were some complications with capture. Below are some issues that I ran into and my solutions.

| Problem                                  | Solution                                                                   |
| ---------------------------------------- | -------------------------------------------------------------------------- |
| ALFA adapter had low packet capture rate | Filtered with `eapol`, change capture times, or (future) upgraded hardware |
| `.pcap` files too large                  | Split files, used live filters, and rotated capture files                  |
| Missed handshakes                        | Forced handshakes to happen again and captured before peak hours           |
| Poor frequency capture                   | Used aircrack-ng suite to lock to target channel                           |

![[Pasted image 20250608225935.jpg|center]]
*Diagram of general data flow (wlan0 was the previous project name)*

---
### data processing and analysis
After all the packets were captured, I decrypted and converted them to CSV files and sent them to my local machine to do further analysis. I wrote a Python script that consolidated all the non-encrypted packets to be sorted by IP (essentially by device). This also allowed me to reveal what IPs were given to what domains that they were searching for, just by matching request to response. 

Sample output of grouping:
```text
IP: 192.168.200.59
=================================
ENTRY 242134 -----------
dest: www.google.com @ 
time: 1936.409025507
ref : 0x5830

ENTRY 244970 -----------
dest: instantmessaging-pa-jms-us.googleapis.com @ 
time: 1937.962279076
ref : 0x0a7d

ENTRY 247821 -----------
dest: voilatile-pa.googleapis.com @ 
time: 1939.601636880
ref : 0x3b32

ENTRY 382168 -----------
dest: pixelweatherhub-pa.googleapis.com @ 
time: 2040.176508006
ref : 0x0e39

ENTRY 248796 -----------
dest: connectivitycheck.gstatic.com @ 
time: 2462.386269794
ref : 0x7609

...
```

For analysis, the above data was sent to Gemini with a guiding prompt to come up with a creative way to present a highly summarized manner. While this is a more playful approach, just by changing the prompt, more useful and accurate reports can be generated. 

I can see this process being used to quickly condense data, and could pair well as an initial summary before further details can be gathered on key areas, anomalies, and points of interest.

>This project left me surprised at how much passive metadata can reveal about behavior. The profiling step felt simultaneously clever and unsettling — a small reminder of how fragile digital privacy can be in open environments.

---
# links
- Python Source Code
- Python Sample Output
- Gemini Sample Output

>[!star]- thoughts regarding the use of AI
There were two ways I used AI in this project. One was in profiling the users with Gemini. This I think is a practical use for AI as it automates a lot of time-consuming parsing, as long as its used in moderation.
>
The other way, using DeepSeek to aid my learning while I built this entire system,
>
I was able to learn more about how networks worked by just applying myself and learning. There are small details that I picked up while putting everything together, and I think that was extremely productive.
>
However, it's scary how easily I was able to put this together. It is able to allow malicious actors to act faster and above their skill level. It brings into question how we can prevent the "hacking" of AI to build something malicious while still allowing it to be useful to the right people. I don't know if that will ever be possible and will always be something concerning.
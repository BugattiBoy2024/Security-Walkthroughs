
# BuildingMagic Walkthrough

**Challenge Source:** HackSmarter  
**Created By:** Noah Heroldt, Haik Isikbay  
**Writeup By:** BugBoy  
**Difficulty:** Easy  
**Primary Skill:** AD Takeover  

---

## Objective

The objective is to achieve a full compromise of the Active Directory environment.

---

## Challenge Overview

BuildingMagic is a targeted enumeration and privilege‑escalation challenge focused on identifying weak credentials, analyzing exposed SMB shares, and leveraging misconfigurations to gain progressive access across the environment. The room emphasizes disciplined scanning, methodical validation of findings, and chaining small oversights into full compromise. Your objective is to enumerate thoroughly, pivot using recovered credentials, and extract the final artifact once full access is achieved.

---

## Methodology

---

## Step 1: Setup

Launch the machine and wait a few minutes for it to fully boot up. While waiting, download the VPN configuration file and run this command…

<img width="493" height="51" alt="image" src="https://github.com/user-attachments/assets/5fc1798a-8da0-4450-b761-a6eafa40a89b" />

To validate that we are connected to the vpn run this command,

<img width="700" height="145" alt="image" src="https://github.com/user-attachments/assets/dd7eb1a3-3790-4493-b4c1-a972f02d00c2" />

Next, the machine tells us to add the name of our DC and localhost to `/etc/hosts`.

<img width="725" height="390" alt="image" src="https://github.com/user-attachments/assets/23937be3-f023-4ca3-93ee-8d3ff4069ec9" />

---

## Step 2: Initial Compromise

The lab begins with a leaked database of credentials, a common scenario in real-world pentesting. The leaked passwords are non-salted hashes, making them vulnerable to cracking.

### **Leaked Database**

<img width="959" height="331" alt="image" src="https://github.com/user-attachments/assets/ff915df8-7839-4328-887d-3e096036b0e3" />


The first step is to use AI to make our non-salted hashes into a copiable format. (For this I used Copilot AI.)

<img width="1107" height="502" alt="image" src="https://github.com/user-attachments/assets/b7bb4d04-2907-454a-a652-9ef0e6eac289" />

Now with that copied let’s use a very convenient website that will crack these hashes for us! Browse to CrackStation.

<img width="2246" height="648" alt="image" src="https://github.com/user-attachments/assets/b3627e96-c818-4ebb-ba41-98717c62b721" />

You can see that CrackStation was able to crack 2 of the 10 password hashes! But how do we know that those passwords still work? For that we use this netexec command.

<img width="1935" height="310" alt="image" src="https://github.com/user-attachments/assets/f2e63c4f-57a3-48b8-a9f6-1db450d97d13" />

As we can see that pair of credentials was valid because it showed those database shares. Now let's test the other pair of credentials.

<img width="1944" height="108" alt="image" src="https://github.com/user-attachments/assets/cfb00640-be6d-4ed2-b046-bac120820166" />

It looks like this pair of credentials were not valid. So we now have one valid pair of credentials.

---

## Step 3: Enumerating with Bloodhound

The next step is launching BloodHound — but before we fire it up, we need to collect the necessary “loot” so BloodHound has the data it needs to operate properly. We do that by using netexec again.

<img width="1348" height="66" alt="image" src="https://github.com/user-attachments/assets/0d8945ec-f737-445f-a612-2fb6f44d9ba6" />

After running that command we need to copy that `.zip` file to our current directory.

<img width="961" height="51" alt="image" src="https://github.com/user-attachments/assets/09cd22e4-8420-452c-b03e-4525ef636a30" />

Now finally we have the proper data to launch BloodHound with this command.

<img width="390" height="60" alt="image" src="https://github.com/user-attachments/assets/620cce12-db73-4222-a821-1b51da9080f0" />
<img width="1095" height="111" alt="image" src="https://github.com/user-attachments/assets/ca3644a1-ec07-431f-9321-b686e28ed9d0" />

Now with these credentials we will log into BloodHound.  
(Note: the default username for logging into BloodHound is **admin**.)

<img width="529" height="724" alt="image" src="https://github.com/user-attachments/assets/1c236ee7-a17f-4121-ba61-a88e6f150a59" />

Before we start enumerating we have to ingest our `.zip` file into BloodHound.  
Go to: **Administration → File Ingest → Upload your .zip file (note: it may take some time to upload the file!)**.

<img width="1018" height="469" alt="image" src="https://github.com/user-attachments/assets/66bcc54a-e2cb-465b-93bf-db3848ec4e93" />

Now that everything is set up we use the search bar to search for our user **r.widdleton**.

<img width="2559" height="1362" alt="image" src="https://github.com/user-attachments/assets/92547bf9-f247-4cf5-8736-ae36548806ad" />

Initially we don’t seem to find anything interesting… r.widdleton is not a part of any special groups and he doesn’t have any outbound object control.

Instead of searching let's maybe try and find a kerberoastable user.

Scroll down in Cypher until you see **All Kerberoastable Users**.

<img width="2262" height="1349" alt="image" src="https://github.com/user-attachments/assets/260e51e7-de10-4d2c-b422-9a709b7e1c5e" />

And it looks like we may have a target **r.haggard**.

Let's real quick verify that this target is juicy by looking at his 1 outbound object control of force changing the user **h.potch**'s password. This is almost undoubtedly the path forward.

Now you may be wondering how we would pull off this Kerberoasting attack — well we use netexec again!

<img width="2546" height="460" alt="image" src="https://github.com/user-attachments/assets/d34de3a9-418d-4901-906f-088e685a6cb7" />

With that we have r.haggard's password hash! Let's use hashcat to crack it offline.

<img width="676" height="83" alt="image" src="https://github.com/user-attachments/assets/d1aa1287-7e66-4114-b8e9-1c6ebfad9d01" />

We cracked it!!  
**Credentials:** `r.haggard : rubeushagrid`

<img width="2559" height="366" alt="image" src="https://github.com/user-attachments/assets/83216207-28e0-4c24-bd8f-9b41eec8fbb0" />

Now we need to verify that those credentials are valid.

<img width="1944" height="316" alt="image" src="https://github.com/user-attachments/assets/1e2109c6-eb52-41e7-9d65-145e5afd9c90" />

It worked!! Now we need to force change the password of the **h.potch** user.

BloodHound shows different ways to change that user's password.  

<img width="328" height="815" alt="image" src="https://github.com/user-attachments/assets/96de6ca2-7148-491e-ab2f-4a4942c9ecef" />


Let's copy this command and use it to change h.potch's password!

<img width="1312" height="66" alt="image" src="https://github.com/user-attachments/assets/1dcb6823-8068-4dcc-81d7-1914338d42b4" />

If it didn’t throw any errors it most likely means that it worked. We will still double check.

<img width="1951" height="329" alt="image" src="https://github.com/user-attachments/assets/1ab62766-c4cd-449d-b1bd-b54b146ee501" />

If you notice in the command above we see that there is a share that we can read and write to.  
We will leverage this writable share to perform an **LLMNR/NBNS poisoning attack**.

To perform this attack we first need to set up **Responder** to listen for an incoming broadcast name-resolution request.

<img width="735" height="215" alt="image" src="https://github.com/user-attachments/assets/d3bfdbc8-3c73-45ba-8e7e-7a5f359d6d5f" />

After starting responder we will use netexec again with a very powerful module called **slinky** to automate the process of creating and placing a malicious shortcut file on a writable share.

<img width="1948" height="415" alt="image" src="https://github.com/user-attachments/assets/46d5f862-5140-40be-92b0-1b1bd25f9307" />

---

## Step 4: Capturing h.grangon’s Hash

After the command successfully runs we will check responder and see what it captures.

<img width="2227" height="379" alt="image" src="https://github.com/user-attachments/assets/ab981d07-39e4-4e50-8ca6-4d36511c017e" />

Using nano paste h.grangon's password hash into a `.txt` file that we will use when we attempt to crack his password hash with hashcat.

<img width="708" height="65" alt="image" src="https://github.com/user-attachments/assets/0120d26e-24bf-47e3-863a-86e3d1188768" />

After a little while we see that hashcat successfully cracks the password and we get the password **magic4ever**.  

As always we will test this password using netexec… see if you can do it on your own.

With a successfully validated credentials let’s use BloodHound to see if we can find anything special about this user.

After we look at what groups this user is a part of we can see one that sticks out to us:  
**"Remote Management Users"**

<img width="1008" height="376" alt="image" src="https://github.com/user-attachments/assets/15122cbf-5581-4f7c-b4fd-59b024e74a9e" />

Looks like we can use **evil-winrm** to get a shell as the h.grangon user.

<img width="1544" height="272" alt="image" src="https://github.com/user-attachments/assets/edb33e9b-b16f-4807-89de-879f5e2e89cf" />

When we run this command we find something interesting… see if you can spot it.

<img width="830" height="266" alt="image" src="https://github.com/user-attachments/assets/d241862b-fe81-4b28-969d-78bad80eb4fb" />

If you noticed from the previous command we have the privilege to **backup files and directories**.

So we must ask ourselves:  
**"What files would be most valuable to grab from this user?"**

How about the **SAM** and **SYSTEM** files? 
Because SAM holds all local NTLM password hashes and SYSTEM holds the bootkey needed to decrypt them, attackers who steal both can recover or reuse local admin credentials, enabling offline cracking, pass‑the‑hash, and rapid lateral movement across an Active Directory environment. 
Using these commands we can save both of these files to our local user account.

<img width="1140" height="50" alt="image" src="https://github.com/user-attachments/assets/1177f41c-6f89-4bd7-b731-869f593ee6bb" />
<img width="1202" height="53" alt="image" src="https://github.com/user-attachments/assets/4b214341-55e0-484b-8a7e-26069e0beca7" />

Then download both files to your local Kali account.

<img width="742" height="237" alt="image" src="https://github.com/user-attachments/assets/80a9c1be-0fe7-462d-9450-823573ad0b7f" />

On our local Kali account we will use **impacket-secretsdump** to extract password hashes from the files we downloaded.

<img width="1258" height="276" alt="image" src="https://github.com/user-attachments/assets/b26aabaf-fda7-475c-9ef5-1fc901e48d39" />

With that we have the **Administrator Hash!!**

When we try to get a shell as the Administrator it doesn’t work.

<img width="1551" height="344" alt="image" src="https://github.com/user-attachments/assets/93431b65-2dac-4e16-b7a6-3669d8a95b88" />

---

## Step 5: Final Compromise

So let's check what other users we have yet to compromise.

<img width="965" height="221" alt="image" src="https://github.com/user-attachments/assets/e75229e0-c6cc-47d5-be65-0b5d93e6e453" />

Hmmm… let's do the same attack but against the **a.flatch** user.

<img width="1550" height="266" alt="image" src="https://github.com/user-attachments/assets/6d053493-03c7-4d5f-b5ed-7a753dcc0c02" />

Woo-hoo let's grab our flag.

<img width="909" height="363" alt="image" src="https://github.com/user-attachments/assets/389271c8-db7a-429c-a7eb-875445d7642a" />

**Nice job we did it!**

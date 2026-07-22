# Ascension Walkthrough

**Challenge Source:** HackSmarter  
**Created By:** Ryan Yager  
**Writeup By:** BugBoy  
**Difficulty:** Easy  
**Primary Skill:** Enumeration

---

# Objective

Gain Root privileges (hopefully!)

---

# Challenge Overview

Ascension is an entry-level HackSmarter lab designed to reinforce the importance of thorough enumeration as the foundation of every successful penetration test. Although categorized as Easy, this challenge intentionally punishes assumptions and rewards methodical recon.

The machine emphasizes:

- Checking default services instead of jumping to advanced exploitation
- Leveraging simple misconfigurations for real access
- Maintaining a clean, structured workflow
- Understanding how a single missed enumeration step can halt progress

Ascension is ideal for beginners learning disciplined recon, or for experienced testers wanting a reminder that fundamentals win engagements.

---

# Methodology

# Step 1: Setup

1. Connect to the VPN by Downloading the VPN configuration file to your machine. Test that your connected by pinging the target IP.
<img width="690" height="169" alt="image" src="https://github.com/user-attachments/assets/0e21b674-ec37-4abe-bedf-07ce861cb7e3" />

2. Next we will put our target IP into `/etc/hosts` so will easily remember it.

<img width="346" height="45" alt="image" src="https://github.com/user-attachments/assets/72bb7b6b-64a0-48c1-92ff-0be63b19901d" /><br><br>
<img width="336" height="196" alt="image" src="https://github.com/user-attachments/assets/152aec0c-03b7-422c-abce-8cf8cb48bce1" />





After saving our target IP and hostname to `/etc/hosts` we will ping `ascension.hsm` to make sure everything is working properly!
<img width="877" height="168" alt="image" src="https://github.com/user-attachments/assets/038c4e9b-01b5-47ab-b04f-4ee7577f8576" />

Looks like we are all set up! Let's start hacking!

---

# Step 2: Initial Enumeration

1. We will run an initial NMAP scan to see what ports are open on the target machine.

<img width="383" height="311" alt="image" src="https://github.com/user-attachments/assets/d744fa03-e939-44aa-9fda-3cf6283eb179" />

A few interesting things to note, it appears there is a webserver running on port 80 as well as SSH and FTP respectively.

<img width="1063" height="1321" alt="image" src="https://github.com/user-attachments/assets/31d1bdcd-1f5e-4f1a-95ab-99cb21c3045b" />


The second part of this scan shows that anonymous ftp login is allowed. Anonymous FTP login means anyone can access the FTP server, often leading to information disclosure, file manipulation, and sometimes remote code execution.

---

# Step 3: Exploiting FTP

Before proceeding with further enumeration, we attempt to authenticate to the FTP service using anonymous credentials. The presence of anonymous FTP access is itself a valid security finding, as it allows unauthenticated users to connect to the server and is widely considered an insecure and outdated configuration.

<img width="447" height="247" alt="image" src="https://github.com/user-attachments/assets/6c41d832-fb64-4b99-9d2e-7fb0507f73be" />

As you can see we were able to successfully get access to the ftp server by using the anonymous username and a password of our choosing. Let's see what we can find on this FTP server!

<img width="798" height="163" alt="image" src="https://github.com/user-attachments/assets/fc7030f0-acc8-42fa-979f-c3f530165dba" />

Looks like we found something, before we can read it we have to copy it to our machine. Another important step is checking whether the FTP server allows write access. While anonymous read access is already insecure, anonymous write access significantly increases the impact of the vulnerability. If an attacker can upload files to the FTP server, they could place malware, malicious scripts, or other harmful content on the system, potentially leading to service compromise or further exploitation.

<img width="2536" height="165" alt="image" src="https://github.com/user-attachments/assets/f6a94db6-5886-4f87-9ee6-e10fd792db6d" />
<img width="352" height="481" alt="image" src="https://github.com/user-attachments/assets/2df402bb-7c44-4538-9bcd-a0ce74b7a4c5" />


After reviewing the contents of the file, we find a list of passwords that appear to be extremely common and easily guessable. This strongly suggests the presence of weak credential practices and increases the likelihood of successful brute-force or credential-stuffing attacks. Although not super useful right now this might be of use in the future.

---

# Step 4: Enumerating SSH (22)

There is one main thing that we want to check for when enumerating SSH, that is whether key based authentication or password based authentication is enabled. On less secure systems password based authentication will be enabled. This is because passwords can be brute forced and keys cannot. Checking for this is very simple.

<img width="972" height="193" alt="image" src="https://github.com/user-attachments/assets/dc086070-5a12-41d5-ba53-22bc7e501188" />

As you can see this ssh server is configured securely and it requires key based authentication.

---

# Step 5: Enumerating Web Server (80)

If you recall back to our initial enumeration phase you might remember that we had port 80 open which tells us that there is a website running.

<img width="438" height="118" alt="image" src="https://github.com/user-attachments/assets/21ea778d-aae4-4329-9d71-dd08dd7df65d" />

Ater looking up if this version of apache is outdated or has known vulnerabilities we find that this version of apache is in fact up to date with no known vulnerabilities!

Let's browse to the target website.

<img width="804" height="1170" alt="image" src="https://github.com/user-attachments/assets/52d1c9f0-c469-4b16-9a61-405d8e77922d" />

We can see that we are brought to the default apache page!

The first thing I did was run a subdomain enumeration tool called dirsearch, (sidenote I also ran a vhost fuzzer but got nothing)

<img width="451" height="48" alt="image" src="https://github.com/user-attachments/assets/ec3261f4-184f-4346-8b13-1df49c33a186" />
<img width="939" height="508" alt="image" src="https://github.com/user-attachments/assets/4733baa5-cdc3-4b7a-b0f2-09a917955a2b" />

As you can see this appears to be an unfinished WordPress website.

I also ran WPScan but WP isn’t fully installed on the target webserver so it didn't recognize the hostname. Safe to say that this webserver could be a rabbit hole!

---

# Step 5: Enumerating NFS

If you recall our Nmap scan you might remember we saw that there was a port open for NFS. NFS (Network File System) is a protocol that lets computers share and access files over a network as if they were on a local disk. As you can see we found a mountable share!

<img width="372" height="97" alt="image" src="https://github.com/user-attachments/assets/b73b13f0-3f59-4aaa-80b2-f915033c06e9" />

Run this command to mount the share!

<img width="1030" height="57" alt="image" src="https://github.com/user-attachments/assets/39c16997-2cbb-40e3-aceb-b510cab9aee3" />

It appears the directory contains an SSH private key and public key: `id_rsa` and `id_rsa.pub`. To find out which user owns this set of keys we run this command!

<img width="2545" height="133" alt="image" src="https://github.com/user-attachments/assets/181786be-1f80-4858-879a-73983f5fa235" />

Now let’s see if we can authenticate as user 1 using their keys.

<img width="465" height="90" alt="image" src="https://github.com/user-attachments/assets/5d1feef1-92d5-4648-983f-218e6ba7472c" />

As you can see we ran into a problem. We need a passphrase for this privatekey. How about we use `ssh2john` to try and give us a hash that we can than try to crack.

<img width="2547" height="510" alt="image" src="https://github.com/user-attachments/assets/aeb10bca-16fd-45d7-aabd-57c5fdc1a33c" />

We than copy this output and put it in a file called `hash2.txt`.

<img width="748" height="52" alt="image" src="https://github.com/user-attachments/assets/85453391-a2e2-4016-a2d8-24db91dea655" />

After running this command (and waiting a little while) you should see that john successfully cracks it and gets a passphrase of `"sammie1"`. Let's now login as user 1 and grab our first flag!

<img width="366" height="49" alt="image" src="https://github.com/user-attachments/assets/9c5a9bcc-d804-4a94-9bc8-db48b0f3fd8f" />

We have successfully logged in as user1!

<img width="540" height="118" alt="image" src="https://github.com/user-attachments/assets/0b7908ba-2efc-4a6e-ac0d-3072c68f7a7f" />

BOOM our first flag!!

---

# Lateral Movement to User2

# Step 1: Enumerating User1

The first thing we are going to do is cat out `/etc/passwd` to see what users are on this machine.

<img width="661" height="144" alt="image" src="https://github.com/user-attachments/assets/cef04792-e1e8-449f-ab7f-f75a92603a23" />

It looks like we have 3 other users of interest.

After digging around the users file system I found a few interesting things. One being in `/var/www/html` where I found quite a few configuration files for the target website.

<img width="771" height="604" alt="image" src="https://github.com/user-attachments/assets/38518818-146d-4cb5-b596-ede8446b6d39" />
<img width="634" height="457" alt="image" src="https://github.com/user-attachments/assets/482ad529-728f-4af7-841c-3d247677e212" />

We found WordPress credentials that might not be useful but at least their worth putting in our notes.

---

# Step 2: Exploitation

After continual snooping around I didn’t come across anything that really stood out to me. So I decided to use a tool called `pspy`, which stands for process spy. This tool allows us to see if there are any cronjobs running on the target users account. Before we can us this we have to download it to the target machine. Let's first spin up a python web server!

<img width="726" height="76" alt="image" src="https://github.com/user-attachments/assets/185488c8-29fb-4246-8559-11f4abb1eab9" />

And than use wget to grab that file.

<img width="2542" height="244" alt="image" src="https://github.com/user-attachments/assets/1f1f6be0-ed74-41f1-85eb-c6dd2008c1ba" />

Add executable rights to it!

<img width="507" height="28" alt="image" src="https://github.com/user-attachments/assets/4768c8f5-f2ad-4c0e-afe2-64b0cf661ac6" />

Run it.

At the bottom we see something interesting…

<img width="889" height="52" alt="image" src="https://github.com/user-attachments/assets/b900deff-c014-46dc-83af-e0f0ccb79527" />

Now let's find out who user 1002 is. After looking back at the users on this machine we can see that UID 1002 is user 2! This is definitely the path forward. Maybe we could write to the `backup.sh` file and put a reverse shell in there and when the cronjob executes we have a shell! To create a custom reverse shell I will be using a browser extension called Hack-Tools.

<img width="748" height="594" alt="image" src="https://github.com/user-attachments/assets/b575cc24-1478-4ca5-be65-8bf51546ee8b" />

In the `/tmp` directory let's make a file called `backup.sh` that when run will execute with user2 permissions.

<img width="505" height="28" alt="image" src="https://github.com/user-attachments/assets/25d4d7ec-98bb-4d77-b7bc-5b1de1f5c432" />
<img width="679" height="81" alt="image" src="https://github.com/user-attachments/assets/63bf5a59-30ee-4898-8140-b97465f3e176" />

Let's now set up a listener on port 1337.

<img width="957" height="166" alt="image" src="https://github.com/user-attachments/assets/a941dada-b330-4f9a-911b-4324dfd111a7" />

After waiting a little while we can see we got a shell as user2!! Let's get our hard earned flag!

<img width="540" height="72" alt="image" src="https://github.com/user-attachments/assets/effd9d72-ff0a-4878-82d9-55aa167847f2" />

BOOM!!! Give yourself a quick break and let’s get back to work.

---

# Lateral Movement to ftpuser

# Step 1: Stabilizing Our Shell

First things first let's get our shell working properly! Using the Hack-Tricks extension we can see there is a see is a section on stabilizing a shell.

<img width="583" height="322" alt="image" src="https://github.com/user-attachments/assets/5e160ac1-f4d0-4d70-9536-de84fb5e1472" />

Using these two commands we can get a shell as user 2 that is fully functional!

---

# Step 2: Enumerating/Exploiting User2

We know that we want to compromise the ftpuser to get flag 3! With that in mind I enumerated the entire user2 and found nothing… but I had an idea! What if I could switch my user to the ftpuser!

<img width="412" height="70" alt="image" src="https://github.com/user-attachments/assets/14898717-7278-4b31-a6aa-7328e6724cd5" />

Remember that password list that we found at the beginning of the room called `pwlist.txt` that was found by anonymously logging into ftp? What if we could use that to bruteforce ftpuser's password!

<img width="2497" height="244" alt="image" src="https://github.com/user-attachments/assets/51c70e5b-41f7-45ef-add6-58a59495ab08" />

BOOOOOOM!!! We found the ftp user password!

<img width="442" height="75" alt="image" src="https://github.com/user-attachments/assets/279216bd-92f3-4e3d-af78-a4623c70c5ad" />

After successfully logging in let’s grab that flag!

<img width="552" height="121" alt="image" src="https://github.com/user-attachments/assets/d00b6919-81d2-4d1a-86d5-1304d9838c4f" />

And we got the 3rd Flag!!

---

# Compromising User 3

We get a significant hint from the creator of the room that says "Flag 4 is in the MySQL database under a table named flags" If you remember from enumerating user 1 we found MYSQL credentials for a default WordPress database.

<img width="634" height="457" alt="image" src="https://github.com/user-attachments/assets/64f49762-d8f7-43c4-a00e-a1977727d681" />

Let’s try and use those credentials to login into MYSQL.

<img width="952" height="363" alt="image" src="https://github.com/user-attachments/assets/79c02edb-246c-48ec-a2a5-a61dd7b1e696" />
<img width="315" height="222" alt="image" src="https://github.com/user-attachments/assets/585f19e0-ed3c-4ce5-a9d0-9812b35fa6d0" />
<img width="805" height="120" alt="image" src="https://github.com/user-attachments/assets/cb92670a-b7af-49c9-bdb4-aeac1525c79e" />
<img width="306" height="202" alt="image" src="https://github.com/user-attachments/assets/825496c1-8011-4a2c-be4d-c50952722496" />
<img width="613" height="175" alt="image" src="https://github.com/user-attachments/assets/14cb438d-49c8-4621-a50b-1ee86b1d3660" />

And there we have it!! Another flag! Let's find out what was in the other table!

<img width="412" height="168" alt="image" src="https://github.com/user-attachments/assets/42855a24-b868-4fca-98f2-5e0880cc553c" />

Looks like we have the user3 password!

<img width="466" height="93" alt="image" src="https://github.com/user-attachments/assets/7b18a00a-f57b-42c3-8f94-889c7056c156" />

In the `/opt/user3` directory we get flag 5!!

<img width="493" height="45" alt="image" src="https://github.com/user-attachments/assets/e5a7d171-5f68-4715-afa7-5ab27a65ac03" />

---

# Privilege Escalation to Root

Let’s first check if user 3 can run sudo.

<img width="637" height="72" alt="image" src="https://github.com/user-attachments/assets/fdb6f8b6-0c43-42d1-827e-3d61a2fcdfa7" />

Hmmm… so we can't run sudo as user 3.

Let's try running this one liner command that recursively lists every file on the system that has Linux capabilities assigned, while hiding error messages.

<img width="916" height="107" alt="image" src="https://github.com/user-attachments/assets/e8792d0d-05b7-4cc0-bfcd-311dad6301d7" />

Everything looks normal except for one thing. Python should never have capabilities. It’s an interpreter — meaning you can run arbitrary code inside it. Giving an interpreter the ability to change UID is basically giving any user who can run Python full root access! On GTFO Bins I found a command that could potentially give us root access.

<img width="1108" height="70" alt="image" src="https://github.com/user-attachments/assets/cf641614-a026-4626-915d-9f1f2365ae96" />

WE HAVE ROOT!! Congratulations we did it! Before we finish up let's break down this command to understand how it works!

## Line-by-line breakdown

`/home/user3/python3`

You’re executing the Python interpreter that has the dangerous capability attached.

`-c`

Run the following Python code as a one-liner instead of a script file.

`import os`

Loads Python’s OS interface module.

`os.setuid(0)`

Changes the current process’s UID to 0, which is root.

Normally this would fail unless:

- The binary is setuid-root, or
- The binary has the capability `cap_setuid=ep`

Your Python binary has the second one.

`os.system("/bin/sh")`

Spawns a shell. Because the process is already UID 0, the shell is root.

Now let’s grab that root flag!

Pat yourself on the back, mission accomplished!

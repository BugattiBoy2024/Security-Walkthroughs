# Verbose Walkthrough

**Challenge Source:** HackSmarter  
**Created By:** Tyler Ramsbey  
**Writeup By:** BugBoy  
**Difficulty:** Easy  
**Primary Skill:** Web App Pentesting 

---

## Objective

Gain Root privileges (hopefully!)

---

## Challenge Overview 

The Verbose lab is an easy‑difficulty web application penetration test designed to walk you through a full kill chain: enumeration → vulnerability discovery → exploitation → privilege escalation. Everything below is grounded in the retrieved sources

Verbose is ideal for beginners learning the fundamentals of web app pentesting as well as more seasoned professionals building skills like editing metadata.

---

## Methodology

### Step 1: Setup

Launch the machine and wait a few minutes for it to fully boot up. While waiting, download the VPN configuration file and run this command…

<img width="514" height="51" alt="image" src="https://github.com/user-attachments/assets/d31b9b60-da89-4caf-aac6-020b8ea63bb0" />

Than add verbose.hsm to /etc/hosts so we don’t have to memorize the IP address.

<img width="336" height="203" alt="image" src="https://github.com/user-attachments/assets/fdafe45e-9530-40f8-a6d9-86066f555736" />

The last and final step of the setup process is to ping verbose.hsm.

<img width="871" height="174" alt="image" src="https://github.com/user-attachments/assets/33660252-b7aa-488d-a727-2504b6a6e134" />

Looks like were fully connected!

---

### Step 2: Enumeration

Let's do a basic NMAP scan 

<img width="765" height="436" alt="image" src="https://github.com/user-attachments/assets/8ca1525a-f65a-4ed7-98f6-53bbad93486b" />

Now we are going to open up a browser in caido and take a look around the website. (We also ran a dirsearch scan but found nothing.)

<img width="1248" height="511" alt="image" src="https://github.com/user-attachments/assets/58d8d535-3167-4ba7-87ad-e88eebb1f45b" />

We can see we are met with a login page! Let's list a few things we can do to discover users and their passwords…

- Bruteforce  
- Username Enumeration  
- Registering a user and doing  recon on the website

---

### Step 3: Exploitation

The first thing we will do is register a fake user to take a look around the website.

<img width="904" height="363" alt="image" src="https://github.com/user-attachments/assets/74db4f67-fa51-4f3f-8234-f98af3355015" />

Let's capture a messages request with Caido.

<img width="2356" height="919" alt="image" src="https://github.com/user-attachments/assets/1b1bd5df-62a5-489e-b658-529802614e4d" />

Nothing there let's forward the request again!

<img width="635" height="98" alt="image" src="https://github.com/user-attachments/assets/56614b8c-f27f-41c1-a613-559656b5678b" />

On the second request we can see that there is an API request to /api/users/all! Let's browse to that endpoint and see what we find.

<img width="1969" height="127" alt="image" src="https://github.com/user-attachments/assets/9709680a-9686-4df4-bd73-2b0d062d112a" />

BOOM! We found every users username, email, and password!!!

Let's login as the administrator and grab our first flag!

When we enter the admin's username and password an hit login we are faced with a problem… the admin seems to have 2fa enabled.

<img width="900" height="274" alt="image" src="https://github.com/user-attachments/assets/b0863d4a-2f22-49f6-ad3d-64c5d947d6ba" />

Let's capture a verify request with Caido send it to automate and bruteforce it.

I highlight our target selection and then use a custom list I made with every possible 4 digit combination.

<img width="1765" height="375" alt="image" src="https://github.com/user-attachments/assets/8b791c33-f712-4899-9b4b-3ee48dc91e13" />

Hit run and see what we get!

When we filter the status portion of our many different payloads. We see that one sticks out with a 302 status code. As you might know a 302 status code signifies that there was a redirect!

<img width="272" height="274" alt="image" src="https://github.com/user-attachments/assets/2976dfc5-6825-48b5-a6e4-3c45d6564863" />

Let's use that 4 digit key to loin is the administrator and bypass the 2fa prompt. (Note: your 4 digit key may be different than the one highlighted in the screenshot)

There we have our first flag!!

<img width="895" height="885" alt="image" src="https://github.com/user-attachments/assets/0178342e-7ef1-4567-bee8-f5726a791b20" />

---

### Step 5: Enumerating as Admin

The main thing that sticks out to me is the file upload for changing the website logo on the admin panel.

<img width="400" height="250" alt="image" src="https://github.com/user-attachments/assets/64677748-4b67-468b-b69e-bc298f8f14f5" />

I tried uploading malicious files and used Caido to tamper their contents but nothing was working… but than I noticed something. When I previewed the current logo I found that it reflected the metadata in the image onto the website. Than I had the brilliant idea to try and inject metadata into the logo picture!

<img width="829" height="112" alt="image" src="https://github.com/user-attachments/assets/6173bd06-1dcd-4a75-a6de-1bbb9403efc0" />

Let's try to inject some metadata to our logo and see if it gets reflected. For this we are using exiftool.

<img width="496" height="85" alt="image" src="https://github.com/user-attachments/assets/317af3f1-1458-4c06-a2d9-0dd014f6589e" />

After we run this command and reupload the file we can see that our name was reflected on the website.

<img width="892" height="777" alt="image" src="https://github.com/user-attachments/assets/5bfab14e-b964-4c30-b67e-7d5806eedf23" />

Let's try a simple SSTI payload. 

<img width="510" height="78" alt="image" src="https://github.com/user-attachments/assets/960fb74d-fefe-4a82-a9b1-327c7f141019" />

Reupload the file and…

<img width="892" height="771" alt="image" src="https://github.com/user-attachments/assets/7e0f6e95-0540-4790-8d08-227d797e5c77" />

It worked!! Now let's prepare a simple reverse shell payload

<img width="1857" height="82" alt="image" src="https://github.com/user-attachments/assets/0578067d-31a2-4137-bfd3-bf17f593f2d2" />

Run this command and set up a netcat listener.

<img width="360" height="81" alt="image" src="https://github.com/user-attachments/assets/37688cb3-4d6e-42c7-8099-731d9477fdaf" />

Now lets reupload the file and see what we get!!

<img width="744" height="144" alt="image" src="https://github.com/user-attachments/assets/692ec375-21d6-4412-b44e-90e738b43521" />

WE ARE ROOT!!!!!

Let's now grab the root flag. In the /root directory we find the flag!

<img width="208" height="40" alt="image" src="https://github.com/user-attachments/assets/c573632d-48c1-4218-9716-0ba3f6839045" />

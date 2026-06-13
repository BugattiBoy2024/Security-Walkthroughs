# Hunter (Easy) – Username Enumeration Challenge Walkthrough
**Challenge Source:** HackSmarter  
**Created By:** Tyler Ramsby  
**Writeup By:** BugBoy  
**Difficulty:** Easy  
**Target IP:** 10.1.34.70  
**Primary Skill:** Timing-Based Username Enumeration  

---

## Objective
Identify a valid username from a provided list by analyzing the behavior of a web application's login portal.

---

## Challenge Overview
This challenge demonstrates a common authentication weakness known as username enumeration. By observing subtle differences in the application's response time, it is possible to determine whether a username exists in the system.

### Key Concept
When a login request is submitted:  
• Valid usernames trigger additional backend processing (such as password verification).  
• Invalid usernames are rejected immediately.  
• This difference creates a measurable timing discrepancy that can reveal valid accounts.  

---

# Methodology

---

## Step 1: Setup
1. Connect to the VPN Download the VPN configuration file to your machine. Test that your connected by pinging the target IP  
<img width="762" height="322" alt="image" src="https://github.com/user-attachments/assets/b870a6b3-3ee1-45b4-b962-bee6d3647c8f" />

2. Download "usernames.txt" to your machine  
<img width="284" height="50" alt="image" src="https://github.com/user-attachments/assets/46930d82-695f-48ad-9eeb-626058c35d34" />



3. Upload "usernames.txt" to Caido as we will be using it later
<img width="899" height="247" alt="image" src="https://github.com/user-attachments/assets/9aafd38a-5478-4826-8992-5f04649c9c1e" />


---

## Step 2: Enumeration
1. Browse to the target IP  
<img width="191" height="215" alt="image" src="https://github.com/user-attachments/assets/1ef6193a-dd51-43be-8a18-3b493ee92586" />

2. Let's try first with the "Forgot password?" page, when we enter a random username we see a seemingly secure alert box appear  
<img width="198" height="219" alt="image" src="https://github.com/user-attachments/assets/b1f9feff-3097-4c68-b991-9487457edea1" />

---

## Step 3: Exploitation
1. Intercept a "Send Reset Link" request using Caido, than right click > Send to Automate  
<img width="901" height="373" alt="image" src="https://github.com/user-attachments/assets/428849ed-6e9d-442d-9a25-4f557d6b80f9" />

2.  
<img width="876" height="206" alt="image" src="https://github.com/user-attachments/assets/e2fcc319-673f-4bbd-ad2a-3954c9026a9c" />

   • Highlight the username field  
   • Click "+" to add your selection as a payload  
   • Select "usernames.txt" which you should have previously downloaded!  


3.  
<img width="882" height="395" alt="image" src="https://github.com/user-attachments/assets/a7b371f5-5354-4993-873c-406158812f72" />

   • Click "Settings"  
   • Change "# of workers" to 5 so that we won't overload the target server  
   • Click "Run" to start our attack  

---

## Step 5: Results/Post Exploitation
1. After waiting for a little we can filter the results and see that on result stands out because it took exponentially longer for the server to respond, this means that we have a valid user named "joey"!  
<img width="1756" height="1237" alt="image" src="https://github.com/user-attachments/assets/01e3604f-797f-455c-ade0-f4425252e358" />

<img width="622" height="216" alt="image" src="https://github.com/user-attachments/assets/879cd571-f742-4b52-9261-5bf6d8544e85" />

**Nice work!**

---

## Why This Works?
Timing‑based username enumeration works because the server accidentally leaks information through how long it takes to respond. Valid usernames trigger different internal code paths than invalid ones, and those paths take measurably different amounts of time.

# TryHackMe - Frosteau Busy with Vim

**Room Link:** [Frosteau Busy with Vim](https://tryhackme.com/room/busyvimfrosteau)  
**Difficulty:** Insane / Hard (Advent of Cyber 2023 Side Quest Challenge)  
**Objective:** Breach the outer defenses, navigate the restricted environments, and compromise Detective Frosteau's machine to claim the Yeti Key.

---

## 🛠️ Challenge Tasks & Solutions

### Task 1: Start the Machine & Investigate
The Bandit Yeti is seeking revenge and aiming to compromise the network defenses maintained by Elf McSkidy and Detective Frosteau.

#### **Question 1: What is the value of the first flag?**
* **Answer:** `THM{Let.the.game.begin}`

#### **Question 2: What is the value of the second flag?**
* **Answer:** `THM{Seems.like.we.are.getting.busy}`

#### **Question 3: What is the value of the third flag?**
* **Answer:** `THM{Not.all.roots.and.routes.are.equal}`

#### **Question 4: What is the value of the fourth flag?**
* **Answer:** `THM{Frosteau.would.be.both.proud.and.disappointed}`

#### **Question 5: What is the value of the third Yetikey that has been placed in the root directory to verify the compromise?**
* **Answer:** `3-d2dc6a02db03401177f0511a6c99007e945d9cb9b96b8c6294f8c5a2c8e01f60`

---

## 🚀 Brief Walkthrough Summary

1. **Initial Reconnaissance:** Scan ports to uncover open services. Main entry points point toward standard networking protocols and internal web services hosted across unique ports (e.g., FTP, Port 80, 8085, 8095).
2. **Foothold via Vim (Port 8085):** Interact with the text editor session exposed over the network. Utilizing Vim's interactive escape mechanisms and command execution triggers a local shell environment or internal environment variables exploration.
3. **Escaping the Sandbox/Container:** Identify system restrictions and navigate processes (`/proc` directory mapping analysis) to transition or hijack context outside the underlying container or restricted runtime environment.
4. **Privilege Escalation:** Exploit write/execution permissions on internal system files or paths (such as `/usr/frosty/sh` or altering raw system files like `/etc/passwd` via high-privilege write mechanics) to pivot completely to `root` and grab the `Yetikey`.

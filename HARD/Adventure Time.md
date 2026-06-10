# Adventure Time - TryHackMe Walkthrough
## Author: Nayanjyoti Kumar

**Room:** Adventure Time
**Difficulty:** Hard
**Platform:** TryHackMe

A fun puzzle-oriented CTF where players help Finn and Jake recover BMO's reset code. The challenge combines enumeration, steganography, cryptography, web exploitation, and privilege escalation techniques.

---

## Skills Covered

* Network Enumeration
* FTP Enumeration
* Web Enumeration
* SSL Certificate Analysis
* Steganography
* Cryptography

  * Binary
  * Base32
  * ROT11
  * Morse Code
  * AES Decryption
  * Vigenère Cipher
  * Spoon Language
* Bash Scripting
* SSH Brute Force
* Linux Privilege Escalation

---

## Initial Enumeration

A full TCP scan revealed several interesting services:

| Port  | Service                       |
| ----- | ----------------------------- |
| 21    | FTP (Anonymous Login Enabled) |
| 22    | SSH                           |
| 80    | HTTP                          |
| 443   | HTTPS                         |
| 31337 | Custom Service                |

The FTP service allowed anonymous access and contained six image files.

---

## FTP Enumeration

After downloading all images, metadata analysis was performed using ExifTool.

```bash
for i in {1..6}; do
    exiftool $i.jpg | grep "XP Comment" | cut -d ":" -f2
done
```

The extracted binary text decoded to:

```text
you really like to puzzle don't ya
```

This indicated that hidden clues would be embedded throughout the challenge.

---

## Web Enumeration

Visiting the web server displayed an image of Finn.

Directory brute forcing with Gobuster revealed:

```text
/ candybar /
```

Further SSL certificate inspection exposed additional domain names:

```text
land-of-ooo.com
adventure-time.com
```

These were added to the local hosts file:

```text
10.10.x.x land-of-ooo.com adventure-time.com
```

---

## Hidden Directories

Recursive Gobuster scans uncovered several paths:

```text
/ yellowdog /
/ bananastock /
/ princess /
```

Each page contained clues hidden either in images or HTML comments.

---

## Morse Code Challenge

An HTML comment contained Morse code:

```text
_/..../.\_.../._/_./._/_./._/...\._/._./.\_/..../.\_..././.../_/_._.__/_._.__/_._.__
```

Decoding revealed:

```text
THE BANANAS ARE THE BEST!!!
```

This phrase later served as an important password.

---

## AES Encrypted Message

The Princess Bubblegum page contained encrypted data along with:

* Ciphertext
* AES Key
* IV
* CBC Mode

After decrypting the message, the following clue was recovered:

```text
The magic safe is accessible at port 31337.
The magic word is: ricardio
```

---

## Port 31337

Connecting to the custom service:

```bash
nc TARGET_IP 31337
```

The service requested a magic word.

Providing:

```text
ricardio
```

returned:

```text
The new username is: apple-guards
```

---

## SSH Access

Using:

```text
Username: apple-guards
Password: THE BANANAS ARE THE BEST!!!
```

SSH access was obtained.

### Flag 1

```text
tryhackme{Th1s1sJustTh3St4rt}
```

---

## Marceline User

A mailbox message hinted that Marceline had hidden a file.

Searching the filesystem:

```bash
find / -user marceline -type f 2>/dev/null
```

revealed:

```text
/etc/fonts/helper
```

The challenge involved solving:

```text
Gpnhkse
```

Using ROT11 decoding produced:

```text
Abadeer
```

Entering the answer revealed Marceline's password:

```text
My friend Finn
```

Switching users:

```bash
su marceline
```

### Flag 2

```text
tryhackme{N1c30n3Sp0rt}
```

---

## Spoon Language Puzzle

Marceline's secret note contained encoded Spoon language.

Decoding it produced:

```text
The magic word you are looking for is ApplePie
```

Submitting the word to the service on port 31337 revealed:

```text
The password of peppermint-butler is:
That Black Magic
```

---

## Peppermint Butler

Switching users:

```bash
su peppermint-butler
```

### Flag 3

```text
tryhackme{N0Bl4ckM4g1cH3r3}
```

Two files provided steganography clues:

```text
/usr/share/xml/steg.txt
/etc/php/zip.txt
```

A hidden archive was extracted from an image using:

```bash
steghide extract -sf butler-1.jpg
```

The extracted notes referenced:

```text
The Ice King s????
```

---

## SSH Brute Force

Using the clue, a custom wordlist was generated and Hydra was used:

```bash
hydra -l gunter -P passwords.txt ssh://TARGET_IP
```

Valid credentials:

```text
gunter : The Ice King sucks
```

---

## Gunter User

Switching users:

```bash
su gunter
```

### Flag 4

```text
tryhackme{P1ngu1nsRul3!}
```

---

### Flag 5

```text
tryhackme{Th1s1s4c0d3F0rBM0}
```

---

## Flags Obtained

| Flag   | Value                         |
| ------ | ----------------------------- |
| Flag 1 | tryhackme{Th1s1sJustTh3St4rt} |
| Flag 2 | tryhackme{N1c30n3Sp0rt}       |
| Flag 3 | tryhackme{N0Bl4ckM4g1cH3r3}   |
| Flag 4 | tryhackme{P1ngu1nsRul3!}      |
| Flag 5 | tryhackme{Th1s1s4c0d3F0rBM0}  |

---

## Conclusion

This room provides a highly engaging journey through multiple cybersecurity concepts including FTP enumeration, web discovery, cryptography, steganography, SSH brute forcing, and Linux privilege escalation. The challenge emphasizes careful observation and puzzle-solving rather than exploiting real-world vulnerabilities, making it an excellent learning experience for beginners and intermediate CTF players.

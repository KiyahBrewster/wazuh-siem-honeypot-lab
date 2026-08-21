# Incident Analysis: Cowrie Honeypot Capture Report

**Sensor:** Cowrie SSH honeypot (DigitalOcean droplet, Ubuntu 24.04)
**Public IP:** 157.230.225.198
**Capture window:** August 21, 2026, 06:04 UTC – 06:48 UTC (~44 minutes)
**Analyst:** Kiyah Brewster

## Summary

Within hours of deployment, the honeypot recorded over 25 distinct connection
attempts from more than 20 unique source IPs, confirming that internet-facing
SSH services are targeted by automated scanning almost immediately upon
exposure. The majority of activity consisted of low-effort credential-stuffing
attempts. One session, however, escalated beyond simple login testing into an
attempted **SSH persistence attack** — a botnet actor attempting to plant its
own SSH key to maintain long-term access to the system.

## Environment

The honeypot was deployed on a public cloud VM specifically so it would be
reachable by real internet traffic, rather than only simulated activity. Real
administrative access to the host was moved to a non-standard port prior to
exposure, and `iptables` was used to transparently redirect inbound traffic on
port 22 into Cowrie's listener on port 2222 — allowing the honeypot to
impersonate a standard SSH service without exposing the real management
interface.

## Key Finding: Attempted SSH Key Persistence (Backdoor Installation)

**Source IPs:** 134.112.56.47, 14.141.157.218 (consistent with the same
campaign or shared botnet infrastructure)

After authenticating with a weak, commonly-used credential pair
(`it`/`itpass` and `sumit`/`sumit123` respectively), the actor immediately
executed the following command sequence:

```
cd ~; chattr -ia .ssh; lockr -ia .ssh
cd ~ && rm -rf .ssh && mkdir .ssh && echo "ssh-rsa AAAAB3NzaC1yc2E..." >> .ssh/authorized_keys
```

**What this does:**
1. `chattr -ia .ssh` / `lockr -ia .ssh` — removes the immutable file attribute
   from the `.ssh` directory, clearing any protection that would block
   modification.
2. `rm -rf .ssh && mkdir .ssh` — deletes the existing SSH configuration
   directory entirely and recreates it empty.
3. The `echo` command writes an attacker-controlled public key directly into
   `authorized_keys`.

**Why this matters:** this is not credential testing — it is an attempt to
establish **persistent, passwordless access** to the system. If successful,
the attacker would retain access even if the compromised account's password
were later changed, since authentication would now succeed via their planted
key. This is a well-documented technique used by SSH-targeting botnets (such
as variants associated with the "Diicot"/similar credential-stuffing
campaigns) to convert a single successful login into durable, long-term
access.

The identical command sequence appearing from two different source IPs within
a five-minute window suggests shared tooling or shared botnet infrastructure,
rather than two independent actors coincidentally using the same technique.

## Secondary Finding: Host Reconnaissance Script

**Source IP:** 92.118.39.14

A separate session, after authenticating with `root`/`1`, executed a
multi-stage shell script collecting:
- OS type, kernel version, and architecture
- CPU core count and model
- GPU information (explicitly checking for NVIDIA hardware)
- System uptime and login history (`last`)

The script also included a distinct behavioral probing section, testing how
the shell responded to intentionally invalid commands (`./xxxxxx`) across
multiple shell interpreters (`bash`, `sh`, `busybox sh`). This pattern is
consistent with **anti-honeypot / sandbox-evasion tooling** — automated
frameworks increasingly attempt to fingerprint whether they have landed in a
monitored research environment before proceeding to a payload stage,
suggesting the actor's tooling was designed to verify a genuine target before
continuing further action.

## General Activity Patterns

- **~25 connection attempts** across the capture window from **20+ unique
  source IPs**, confirming the target was found and probed almost immediately
  after going public — consistent with continuous, internet-wide SSH scanning
  rather than targeted attention.
- **Username `root` dominated** login attempts, alongside a small number of
  generic/default-sounding usernames (`it`, `sumit`), reflecting standard
  credential lists used by scanning tools.
- **Most sessions were extremely short** (under 1–2 seconds) and involved at
  most a single `whoami` command before disconnecting — consistent with
  automated "verify access, log result, move on" behavior rather than manual
  human interaction.
- **Password patterns** ranged from default/common values (`123456`-style,
  `admin`) to what appear to be personally meaningful but weak strings
  (`blood666`, `driven`), consistent with credentials sourced from prior
  breach/password-dump lists rather than randomly generated.

## Indicators of Compromise (IOCs)

| IP Address | Behavior |
|---|---|
| 134.112.56.47 | SSH key persistence attempt |
| 14.141.157.218 | SSH key persistence attempt (identical TTPs) |
| 92.118.39.14 | Host reconnaissance / anti-honeypot probing |

## Recommendations

1. **Disable password-based SSH authentication** in favor of key-only access,
   eliminating the credential-stuffing attack surface entirely.
2. **Monitor `.ssh/authorized_keys` for unexpected modification** — file
   integrity monitoring (e.g. Wazuh's FIM module) would have flagged the
   `rm -rf .ssh` / key-plant sequence in real time on a real production host.
3. **Rate-limit or fail2ban repeated authentication failures** to slow
   automated scanning against real-world exposed services.
4. **Never expose SSH management interfaces on default ports** for
   internet-facing systems where avoidable, reducing exposure to
   opportunistic, non-targeted scanning.

## Conclusion

Within under an hour of exposure, this honeypot captured not just generic
credential-stuffing noise, but a specific, describable persistence technique
(SSH key backdoor installation) and evidence of automated sandbox-evasion
behavior. This confirms that internet-facing SSH services are attacked
essentially immediately upon exposure, and that even unsophisticated,
automated actors employ real persistence techniques upon gaining access —
underscoring the importance of layered defenses (key-only auth, file
integrity monitoring, and rate limiting) beyond password strength alone.

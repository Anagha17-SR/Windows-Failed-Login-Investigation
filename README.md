# Windows Failed Login Investigation | SOC Home Lab

**A beginner-friendly Windows Security Event Log investigation using a controlled authentication failure.**

![Lab type](https://img.shields.io/badge/Lab-Simulated-blue)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

## Project overview

This project demonstrates how I generated a **controlled failed sign-in attempt** in my Windows Server 2022 virtual machine, located the corresponding Windows Security event in Event Viewer, analyzed the error codes, and documented an investigation verdict.

**Key result:** Windows logged **Event ID 4625** for an intentionally nonexistent account. The `0xC0000064` substatus indicated that the username did not exist.

> **Scope:** This is a training exercise, not a confirmed cyberattack or production security incident. I used Windows Event Viewer, not a SIEM, in this project.

## Environment and tools

| Component | Details |
| --- | --- |
| Lab platform | Oracle VirtualBox |
| Investigated system | Windows Server 2022 evaluation VM |
| Log source | Windows Security log |
| Analysis tool | Windows Event Viewer |
| Test tool | Windows Command Prompt (`runas`) |
| Detection/event | Windows Security Event ID `4625` |

## Lab workflow

1. Opened **Event Viewer > Windows Logs > Security** and reviewed relevant Windows login events.
2. Used Command Prompt to attempt a sign-in with a deliberately nonexistent local username.
3. Confirmed that Windows rejected the attempt with **RUNAS error 1326**.
4. Filtered the Security log for **Event ID 4625**.
5. Found the event dated **2026-10-28 at 1:44:20 PM** (VM local time).
6. Reviewed the failed account name and authentication status/substatus.
7. Recorded findings and classified the activity as an **expected, benign test**.

### Command used in the lab

```cmd
runas /user:%COMPUTERNAME%\SOC_LAB_NO_SUCH_USER cmd
```

The entered password was intentionally invalid. The command did **not** launch a session under that test account.

## Evidence

**Account named in the failed login:**

![Windows Event 4625 target account](evidence/01-failed-account.png)

**Authentication failure and status codes:**

![Windows Event 4625 status and substatus](evidence/02-failure-codes.png)

| Observed field | Value | Interpretation |
| --- | --- | --- |
| Event ID | `4625` | Failed authentication attempt |
| Computer | `ANAGHA` | Lab VM hostname |
| Target account | `SOC_LAB_NO_SUCH_USER` | Nonexistent test username |
| Target account domain | `ANAGHA` | Local machine domain |
| Timestamp | `2026-10-28 13:44:20` | VM local time |
| Status | `0xC000006D` | Incorrect username or authentication information |
| Substatus | `0xC0000064` | Account name was not found |
| Command output | `RUNAS ERROR 1326` | Windows rejected the credentials |

## Findings and conclusion

**Verdict: Benign / expected activity.** The user account did not exist, Windows correctly denied authentication, and the recorded failure matched my deliberately generated test.

A single failed authentication event is **not evidence of a brute-force attack**. Identifying an actual password attack would require investigating repeated events, accounts, timing, sources, and surrounding activity.

For the full case analysis, see **[Investigation-Report.md](Investigation-Report.md)**.

## Skills demonstrated

- Windows Security event analysis
- Event ID 4624 versus 4625 interpretation
- Authentication failure status-code interpretation
- Evidence capture and investigation documentation
- Separating simulated activity from real security incidents

## Next improvements

- Collect and compare several harmless failed-login attempts in the lab.
- Install Sysmon and investigate process creation events.
- Forward VM logs to a SIEM such as Wazuh or Splunk and practice alert correlation.

## References

- [Microsoft Learn: Windows Event ID 4625](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [MITRE ATT&CK T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/) (background context only; **not demonstrated** in this lab)

*All activity was generated on a personal training VM. No production systems or external accounts were involved.*

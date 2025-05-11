## Threat-Hunting-Scenario-Internet-Facing-VMs
### Misconfigured Devices are setup to be Internet Facing

## 10:15 AM – Unexpected Exposure
It’s a routine maintenance window at a busy financial services company. The infrastructure team is focused on health checks and patch verification, while the security team is assigned a proactive audit task. The mission: to review the shared services VM cluster—which includes critical systems like DNS, Domain Services, and DHCP—for any virtual machines that may have been unintentionally exposed to the public internet.

The security team begins by pulling configuration and firewall rules to identify any misconfigured external-facing interfaces.

## 11:00 AM – The Discovery
During the sweep, the team flags a handful of VMs with open ports that should be restricted to internal traffic. Initial indicators suggest at least one VM may be accepting remote login attempts from external IPs. Suspicion grows as login logs begin to show failed authentication attempts—some from potentially malicious IP ranges.

With the stakes high and core infrastructure potentially vulnerable, the team pivots to identify whether any brute-force login attempts have succeeded and how deep the exposure goes.

### - Preparation
> [!NOTE]
> <strong>Set up the hunt by defining what you're looking for:</strong><BR><BR>
During routine maintenance, the security team is tasked with investigating any VMs in the shared services cluster (handling DNS, Domain Services, DHCP, etc.) that have mistakenly been exposed to the public internet. The goal is to identify any misconfigured VMs and check for potential brute-force login attempts/successes from external sources.
Activity: Develop a hypothesis based on threat intelligence and security gaps (e.g., “Could there be lateral movement in the network?”).
During the time the devices were unknowingly exposed to the internet, it’s possible that someone could have actually brute-force logged into some of them since some of the older devices do not have account lockout configured for excessive failed login attempts.
### - Data Collection
> [!TIP]
> <strong>Gather relevant data from logs, network traffic, and endpoints.</strong><BR><BR>
Consider inspecting the logs to see which devices have been exposed to the internet and have received excessive failed login attempts. Take note of the source IP addresses and number of failures, etc.
Activity: Ensure data is available from all key sources for analysis.
Ensure the relevant tables contain recent logs:<br><br>
DeviceInfo<br>
DeviceLogonEvents
### - Data Analysis
> [!TIP]
> <strong>Investigate any suspicious findings and analyze data to test your hypothesis.</strong><BR><BR>
Activity: Look for anomalies, patterns, or indicators of compromise (IOCs) using various tools and techniques.
Is there any evidence of brute force success (many failed logins followed by a success?) on your VM or ANY VMs in the environment?
If so, what else happened on that machine around the same time? Were any bad actors able to log in?
### - Investigation
> [!WARNING]
> <strong>Investigate any suspicious findings.</strong><BR><BR>
Activity: Dig deeper into detected threats, determine their scope, and escalate if necessary. See if anything you find matches TTPs within the MITRE ATT&CK Framework.
You can use ChatGPT to figure this out by pasting/uploading the logs: Scenario 1: TTPs
### - Response
> [!WARNING]
> <strong>Mitigate any confirmed threats.</strong><BR><BR>
Activity: Work with security teams to contain, remove, and recover from the threat.
Can anything be done?
### - Documentation
> [!NOTE]
> <strong>Record your findings and learn from them.</strong><BR><BR>
Activity: Document what you found and use it to improve future hunts and defenses.
Document what you did
### - Improvement
> [!IMPORTANT]
> <strong>Improve your security posture or refine your methods for the next hunt.</strong><BR><BR>
Activity: Adjust strategies and tools based on what worked or didn’t.
Anything we could have done to prevent the thing we hunted for? Any way we could have improved our hunting process?

## Notes / Findings:

### Timeline Summary and Findings:

After scanning the network, I was able to discover 5 devices that were Internet Facing on the date of 2025-05-05T21:32:01.8689757Z.

![Inititial Internet Facing Scan Code](https://github.com/user-attachments/assets/a6ed9b3b-75f2-412f-9f47-3c91cd4150ae)

The KQL code above let to the following results:

![Initial Scan Results](https://github.com/user-attachments/assets/8bc069f5-46e0-4000-8e76-c79f2fad7ebb)

Understanding what was found: The PublicIPs are not the LocalP addresses, what Microsoft Defender is saying is that these are the What Defender Means by PublicIP (example: 20.75.88.220):

- 20.75.88.220 is a public-facing IP address seen as the source of a network connection (e.g., RDP, SMB, VPN).

- This is not a local/internal IP like 192.168.x.x.

- Defender calls it PublicIP because it’s the IP visible from outside your network — i.e., the external client connecting into your systems.

![Initial Scan Internet Facing Log](https://github.com/user-attachments/assets/7139c3cc-a3c1-45b8-b30e-9f7c3067ff8f)
<BR><BR>
Running this KQL code specifically to check for Brute Force attacks:
<BR><BR>
![Device Logon Event](https://github.com/user-attachments/assets/4f91c0c7-cb5c-43c0-b58b-0c9ab26d3620)

Looking at the record above, this IP (20.75.88.220) is “InternetFacing”
<BR><BR>
![No Brute Force Results](https://github.com/user-attachments/assets/ff929f7f-d2eb-4b80-99c4-094f7d08790a)


The DeviceName is “wnx” and after further investigation, I found that while there are 2 FailedLogonAttempts, there is no evidence of it being the target of Brute Force as that would have yielded a higher failure rate.<BR>

After further investigation, I found that even though the devices were misconfigured and InternetFacing, nothing malicious had happened.

In the real world we would check each of the 5 InternetFacing devices, but for this scenario we will assume I did and they all came back uncompromised.

In a scenario where we did find Brute Force attacks, we see this information from the MITRE ATT&CK Framework:

## Brute Force: Password Guessing: 
Adversaries with no prior knowledge of legitimate credentials within the system or environment may guess passwords to attempt access to accounts. Without knowledge of the password for an account, an adversary may opt to systematically guess the password using a repetitive or iterative mechanism. An adversary may guess login credentials without prior knowledge of system or environment passwords during an operation by using a list of common passwords. Password guessing may or may not take into account the target's policies on password complexity or use policies that may lock accounts out after a number of failed attempts.<BR>

![image](https://github.com/user-attachments/assets/7a9c0a3e-c73f-433d-aa39-539fbb04c78e)

In this case you would take the following steps once you identified Brute Force attacks:

- Containment:
  - Remove the device from the network. For example in Defender you can Isolate the device.
- Eradication:
  - Remove the Public IP or close unnecessary ports. Reconfigure the firewall and NSGs (Network Security Groups)
- Recovery:
  - Restore from a clean backup or secure baseline image. Reapply hardened settings, endpoint protection, and monitoring agents
- Post-Incident Lessons Learned:
  - Document Everything, Audit the environment and update policies & automation

<p align="center">
  <strong>That wraps up Threat Hunting Scenario: Investigating Internet Facing VM's</strong>
</p>

<p align="center">
____________________________________________________________________________________________________________________________
</p><BR>
<p align="center">
  <img src="https://github.com/user-attachments/assets/1d130215-59d9-4e38-978c-87f024ce3604" alt="Description" width="400"/>
</p>
<p align="center">
____________________________________________________________________________________________________________________________
</p>


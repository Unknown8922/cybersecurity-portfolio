# Metasploitable2 Vulnerability Assessment and Exploitation

## 1. Project Overview

In this project, Metasploitable2, an intentionally vulnerable Linux virtual machine (VM), was used as the target for a vulnerability assessment and penetration-testing exercise. The purpose was to identify vulnerable network services, investigate known security weaknesses and perform controlled exploitation to understand how these vulnerabilities could be identified and evaluated.

The assessment focused on three findings: an FTP backdoor, an insecure SUID configuration and an IRC backdoor.

## 2. Lab Environment

Oracle VirtualBox was used to set up an isolated virtual lab environment, with Kali Linux acting as the attacking machine and Metasploitable2 as the target.

| **Component** | **Details** |
|---|---|
| Attacker Machine | Kali Linux 2024.1 |
| Target Machine | Metasploitable2 |
| Virtualisation Software | Oracle VirtualBox |
| Attacker IP | 192.168.56.103 |
| Target IP | 192.168.56.104 |
| Target Operating System | Metasploitable2 Linux |

A host-only network was configured to allow communication between the attacker and target virtual machines without exposing the vulnerable target to external networks.

## 3. Tools Used

- **Nmap:** Used for host discovery, port scanning and service enumeration.
- **Metasploit Framework:** Used to validate the identified vulnerabilities through controlled exploitation.
- **Netcat:** Used to manually connect to and enumerate the IRC service.
- **Linux Command Line:** Used to verify privileges and identify system information.
- **Oracle VirtualBox:** Used to establish and manage the virtual lab environment.

## 4. Methodology

The assessment followed these stages:

1. **Lab Setup:** Started the Kali Linux and Metasploitable2 virtual machines using an isolated host-only network.
2. **Host Discovery:** Used Nmap to identify active hosts within the lab network.
3. **Port Scanning:** Scanned the target to identify open ports and exposed services.
4. **Service Enumeration:** Identified service versions and investigated potentially vulnerable services.
5. **Exploitation:** Used the appropriate Metasploit modules to validate the identified vulnerabilities within the isolated lab.
6. **Verification:** Used system commands to verify the access obtained and assess the impact of each vulnerability.

## 5. Findings

### 5.1 VSFTPD 2.3.4 Backdoor

**Service:** FTP (TCP port 21)

**Identification:**

Nmap identified an FTP service running VSFTPD version 2.3.4 on the target machine.

**Exploitation:**

The Metasploit module `exploit/unix/ftp/vsftpd_234_backdoor` was used to validate the known backdoor vulnerability.

**Result:**

A command shell was successfully established. The `whoami` and `id` commands confirmed that the session had root privileges (`uid=0(root)`).

**Evidence:**

1. [Network host discovery](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/vsftpd/1_network-host-discovery.png)
2. [Nmap scan results](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/vsftpd/2_nmap_scan.png)
3. [Metasploit console setup](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/vsftpd/3_using-msfconsole.png)
4. [Exploitation and root shell verification](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/vsftpd/4_exploit_and_root_shell_verification.png)
5. [Target system identification](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/vsftpd/5_Target_System_Identification.png)
6. [Service identification](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/vsftpd/6_service_identification.png)

### 5.2 SUID Nmap Privilege Escalation

**Service:** Nmap (installed on the target machine)

**Identification:**

After accessing the target through SSH using the `msfadmin` account, I checked the permissions of the Nmap executable. The command `ls -l /usr/bin/nmap` revealed that the executable had the SUID permission enabled and was owned by root.

The SUID permission allows an executable to run with the privileges of its file owner. Since Nmap was owned by root and had SUID enabled, its interactive mode could be abused to obtain elevated privileges.

**Exploitation:**

The installed Nmap version was identified as 4.53. Its legacy interactive mode was used to obtain a shell through the SUID-enabled executable.

**Result:**

Privilege escalation was successful. The `whoami` command returned `root`, while the `id` command showed an effective UID of 0 (`euid=0(root)`).

**Evidence:**

1. [SUID permissions and ownership](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/suid-nmap/1-suid-nmap-permissions.png)
2. [Nmap version](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/suid-nmap/2-nmap-version.png)
3. [SUID Nmap privilege escalation](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/suid-nmap/3-suid-nmap-root-escalation.png)

### 5.3 UnrealIRCd 3.2.8.1 Backdoor (CVE-2010-2075)

**Service:** IRC (TCP port 6667)

**Identification:**

Nmap identified an IRC service running on port 6667. Manual service enumeration using Netcat and the IRC `VERSION` command confirmed that the server was running UnrealIRCd 3.2.8.1.

**Exploitation:**

The Metasploit module `exploit/unix/irc/unreal_ircd_3281_backdoor` was used to exploit the known backdoor vulnerability. A bind TCP shell was successfully established on port 4444.

**Result:**

Root access was confirmed using the `whoami` and `id` commands. The target hostname was also verified as `metasploitable`.

**Evidence:**

1. [UnrealIRCd version identification](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/unrealircd/1-unrealircd-version-identification.png)
2. [UnrealIRCd successful exploitation](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/unrealircd/2-unrealircd-exploitation.png)
3. [UnrealIRCd root access verification](https://github.com/Unknown8922/cybersecurity-portfolio/blob/main/projects/virtual-lab-and-network-enumeration/evidence/unrealircd/3-unrealircd-root-verification.png)

### 5.4 Results Summary

| **Finding** | **Security Impact** | **Result** |
|---|---|---|
| VSFTPD 2.3.4 backdoor | Remote root access | Successful |
| SUID Nmap misconfiguration | Local privilege escalation | Successful |
| UnrealIRCd 3.2.8.1 backdoor | Remote root access | Successful |

## 6. Security Recommendations

Based on the findings, the following security measures are recommended:

- **FTP:** Remove vulnerable or backdoored FTP software and replace it with a maintained, secure file-transfer solution such as SFTP.
- **SUID Permissions:** Review SUID-enabled executables and remove unnecessary SUID permissions. Restrict privileged execution to trusted applications.
- **IRC:** Replace vulnerable UnrealIRCd versions with trusted, patched releases. Regularly update services and verify software integrity.
- **General System Security:** Perform regular vulnerability assessments, maintain software updates and monitor exposed network services.

## 7. What I Would Do Differently

If I were to repeat this assessment, I would improve how I approach service enumeration and evidence collection. I would keep a more structured record of the commands used, scan results and exploitation steps so that each finding is easier to review and reproduce.

I would also spend more time researching the identified service versions and checking their associated vulnerabilities before selecting an exploitation method. This would help me understand the weaknesses more thoroughly rather than relying only on successful exploitation.

Another improvement would be to test the recommended security fixes in the lab. For example, I would remove unnecessary SUID permissions from Nmap, replace vulnerable service versions and perform another scan to check whether the security weaknesses had been addressed.

## 8. Learning Outcomes

This practical exercise improved my understanding of network scanning, service enumeration and vulnerability exploitation in a controlled environment.

I gained experience using Nmap to identify active hosts and exposed services, Metasploit to validate known vulnerabilities, and Netcat to manually enumerate an IRC service.

The three findings demonstrated different security weaknesses, including a backdoor in a network service, privilege escalation through an incorrectly configured SUID executable and remote command execution through a vulnerable IRC service.

I also learned the importance of verifying exploitation results rather than relying only on successful module execution. Commands such as `whoami` and `id` helped confirm the privileges obtained on the target system.

## 9. Scope and Disclaimer

All testing was conducted against an intentionally vulnerable Metasploitable2 virtual machine in an isolated Oracle VirtualBox host-only network. The assessment was limited to the lab environment, and no external systems were targeted.

This project was performed for educational purposes to develop practical vulnerability assessment and penetration-testing skills.

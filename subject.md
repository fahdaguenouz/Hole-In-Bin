## Hole-In-Bin

### Introduction

**Hole-In-Bin** is a comprehensive learning platform designed to teach participants the fundamentals of reverse engineering and binary exploitation. By analyzing and exploiting vulnerabilities in binaries, you will strengthen your understanding of low-level system mechanics and learn essential techniques for identifying and mitigating security risks.

### Why This Project Matters

Binary exploitation and reverse engineering are foundational skills in cybersecurity. Understanding how software works at the assembly level allows you to identify vulnerabilities that higher-level analysis would miss. This project teaches you these critical skills so you can:

- **Build Secure Software**: By understanding how buffer overflows and memory corruption work, you can write code that avoids these vulnerabilities.
- **Conduct Vulnerability Research**: Security researchers analyze binaries to discover zero-day vulnerabilities and help vendors patch them before attackers exploit them.
- **Perform Malware Analysis**: Incident responders and threat analysts reverse-engineer malware to understand its behavior and develop defenses.
- **Strengthen Security Assessments**: Penetration testers who understand binary exploitation can identify deeper vulnerabilities in applications and systems.

The knowledge gained here should be used to **protect systems, improve software security, and conduct authorized security research** — never for malicious purposes.

### Objective

The primary objective of this project is to develop your skills in binary exploitation and reverse engineering. By analyzing and exploiting the provided binaries, you will gain practical experience with concepts such as buffer overflows and memory vulnerabilities.

By completing this project, you will:

- Analyze and exploit binaries to uncover security vulnerabilities.
- Understand assembly code and memory structures.
- Gain hands-on experience with debugging and disassembly tools.
- Enhance your ability to communicate complex technical findings effectively.
- Learn how to identify and prevent these vulnerabilities in real-world applications.

### Role Play

As part of the project, you will participate in a role-play session where you act as a **Security Analyst**. Be prepared to:

- Explain your approach to analyzing and exploiting binaries.
- Discuss the real-world implications of the vulnerabilities you identified.
- Propose practical solutions and prevention methods.
- Demonstrate your understanding of assembly code by explaining specific instructions.
- Explain how organizations can protect against binary exploitation attacks.

The role-play session will test your ability to communicate complex technical concepts effectively and ethically.

### Project Requirements

#### System Requirements

**For Intel/AMD Systems (VirtualBox):**

- VirtualBox 6.0 or later
- Minimum 4GB RAM available for the VM (8GB host RAM recommended)
- 20GB free disk space
- 64-bit processor with virtualization support enabled

**For Apple Silicon (M1/M2/M3/M4) - UTM:**

- UTM (available free from [Mac App Store](https://apps.apple.com/app/utm-virtual-machines/id1538878817) or [getutm.app](https://getutm.app))
- Minimum 4GB RAM available for the VM (8GB host RAM recommended)
- 20GB free disk space

#### Setup and Installation

Download the provided VM image and set it up:

**For Intel/AMD Systems:**

1. Download the [VM Image - OVA Format](https://assets.01-edu.org/cybersecurity/hole-in-bin/hole-in-bin.ova)
2. Verify the SHA1 checksum: `00fda7d71361240d4d32499eb7fc5b156bbd53fc`
3. Open VirtualBox and import the OVA file (File → Import Appliance)
4. Configure network settings to Bridged Adapter or NAT as needed
5. Start the VM

**For Apple Silicon (M1/M2/M3/M4):**

1. Download the [VM Image - UTM Format](https://assets.01-edu.org/cybersecurity/hole-in-bin/hole-in-bin.utm.zip)
2. Verify the SHA1 checksum: `fc93533b2054d10d03b09d53c223e57bf7ac7b62`
3. Extract the ZIP file
4. Open UTM and import the VM (click + → Open Existing)
5. Configure network settings as needed
6. Start the VM

**Verifying Checksums:**

On Linux/macOS:

```bash
sha1sum hole-in-bin.ova
# or
shasum hole-in-bin.utm.zip
```

On Windows (PowerShell):

```powershell
Get-FileHash -Algorithm SHA1 hole-in-bin.ova
```

> Ensure the VM is installed and properly configured for the audit.

#### Access

Log in using the following credentials:

- **Username**: `user`
- **Password**: `user`

#### The Challenges

Navigate to `/opt/hole-in-bin` and review the binaries. Each folder contains:

- A binary file for exploitation.
- A `README.txt` file explaining the exercise requirements and providing hints.

Your task is to exploit these binaries, following ethical hacking guidelines.

### Guidelines

- **Allowed Tools**:
  - Debuggers/Disassemblers: Ghidra (Listing View only), GDB, PEDA, Radare2
  - Scripting: Python, Bash

- **Prohibited**:
  - Automated external scripts or tools for exploitation.
  - Decompilers or decompiler views (including Ghidra's Decompiler Window).

> **Important:** Using a Decompiler is forbidden. You must use Debuggers/Disassemblers instead!

**Clarification on Ghidra Usage:**

Ghidra is allowed, but **only the Listing View (Assembly)**. The **Decompiler Window (which shows C code)** is strictly prohibited. During the audit, you must demonstrate your exploitation using only the assembly view. If you used Ghidra, be prepared to close the Decompiler Window and work exclusively with the Listing View.

**Why Assembly Only?**

- The disassembler translates machine language into assembly language — this is allowed.
- The decompiler attempts to reconstruct high-level C code from assembly — this is NOT allowed.
- Understanding assembly directly is a critical skill for binary exploitation and reverse engineering.

### Documentation

Create a `README.md` file that includes:

1. **Challenge Walkthroughs**:
   - Step-by-step explanation of how you exploited each binary.
   - Tools and commands used.
   - Key takeaways for each challenge.

2. **Remediation Suggestions**:
   - Practical steps to fix or mitigate the identified vulnerabilities.

3. **Ethical Hacking Report**:
   - Importance of proper authorization.
   - Legal and ethical boundaries.
   - Responsible disclosure practices.

### Bonus

If you complete the mandatory part successfully, and you still have free time, you can implement anything that you feel deserves to be a bonus, for example:

- **Exploring Alternative Exploitation Paths**: Document different approaches to solving the challenges.
- **Writing Custom Shellcode**: Develop your own shellcode for the exploits.
- **Return-Oriented Programming (ROP)**: Demonstrate ROP techniques if applicable.

Challenge yourself!

### Ethical and Legal Considerations

This project is for educational purposes only. The skills you learn here are the same skills used by:

- **Vulnerability Researchers** who discover and responsibly disclose security flaws.
- **Malware Analysts** who reverse-engineer threats to protect organizations.
- **Exploit Developers** who work for security companies to test and improve defenses.
- **Security Engineers** who understand exploitation to build more secure systems.

All testing must be conducted in the provided VM environment. Unauthorized attempts to exploit vulnerabilities on live systems or networks are illegal and unethical.

> **Disclaimer**: This project is for learning purposes only. Adhere to ethical hacking practices and legal standards. Misuse of these techniques is prohibited.

### Submission and Audit

Submit the following:

- `README.md` with your challenge walkthroughs and ethical hacking report.
- Any scripts or files used during the project.

Ensure VirtualBox (for Intel/AMD systems) or UTM (for Apple Silicon) is installed and properly configured for the audit.

> **Important:** It's forbidden to use external scripts or decompilers. During the audit, you will be asked to explain specific assembly instructions and demonstrate your understanding of the binaries. Prepare yourself!

### Resources

Some useful resources:

- [Binary Exploitation Techniques](https://www.secquest.co.uk/white-papers/binary-exploitation-techniques): A detailed guide to binary exploitation techniques.
- [Radare2](https://radare.org/n/radare2.html): An open-source framework for reverse engineering and analyzing binaries.
- [Ghidra](https://ghidra-sre.org/): A software reverse engineering suite developed by the NSA.
- [GDB Documentation](https://www.gnu.org/software/gdb/documentation/): Official GNU Debugger documentation.
- [x86 Assembly Guide](https://www.cs.virginia.edu/~evans/cs216/guides/x86.html): Reference for x86 assembly instructions.
- [Smashing the Stack for Fun and Profit](http://phrack.org/issues/49/14.html): Classic paper on buffer overflow exploitation.

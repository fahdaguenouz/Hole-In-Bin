#### General

##### Check the Repo Content

Files that must be present in the repository:

- A `README.md` file that explains all the steps taken to bypass the challenges in the project.
- All tools and scripts used, including their purpose and implementation.

###### Are all the required files present in the repository?

##### Verify VM Setup

> **Important**: All exploitation must be performed inside the provided VM environment. This ensures a controlled, isolated environment for security testing.

**For Intel/AMD Systems (Linux/Windows using VirtualBox):**

1. Download the VM image: [hole-in-bin.ova](https://assets.01-edu.org/cybersecurity/hole-in-bin/hole-in-bin.ova)
2. Verify the SHA1 checksum:

   **On Linux:**

   ```bash
   sha1sum hole-in-bin.ova
   ```

   **On Windows (PowerShell):**

   ```powershell
   Get-FileHash -Algorithm SHA1 hole-in-bin.ova
   ```

   **Expected checksum:** `00fda7d71361240d4d32499eb7fc5b156bbd53fc`

3. Import the OVA file into VirtualBox (File → Import Appliance).
4. Configure network settings as needed (Bridged Adapter or NAT).
5. Start the VM.

**For Apple Silicon (M1/M2/M3/M4 using UTM):**

1. Download the VM image: [hole-in-bin.utm.zip](https://assets.01-edu.org/cybersecurity/hole-in-bin/hole-in-bin.utm.zip)
2. Verify the SHA1 checksum:

   ```bash
   shasum hole-in-bin.utm.zip
   ```

   **Expected checksum:** `fc93533b2054d10d03b09d53c223e57bf7ac7b62`

3. Extract the ZIP file.
4. Open UTM and import the VM (click + → Open Existing).
5. Configure network settings as needed.
6. Start the VM.

**Login Credentials:**

- **Username:** `user`
- **Password:** `user`

###### Is the appropriate virtualization software (VirtualBox or UTM) installed and configured?

###### Does the VM launch successfully?

###### Can the student log in with the provided credentials?

##### Play the Role of a Stakeholder

Conduct a simulated scenario where the student plays the role of a **Security Analyst** presenting their findings to a team of stakeholders (auditors). Evaluate their understanding, communication skills, and depth of knowledge.

Suggested questions include:

- How did you analyze and exploit the binaries?
- What vulnerabilities did you identify, and what is their impact?
- What tools and techniques did you use for exploitation?
- How would you recommend mitigating these vulnerabilities?
- How did you ensure adherence to ethical guidelines?
- How can organizations protect against binary exploitation attacks?

###### Did the student demonstrate a thorough understanding of the project and concepts?

###### Was the student able to communicate effectively and explain their findings?

###### Did the student discuss the real-world implications and propose practical solutions?

##### Review the Student's `README.md` File

Verify that the documentation contains:

1. **Challenge Walkthroughs**:
   - Step-by-step explanation of how each binary was exploited.
   - Tools and commands used.
   - Key takeaways for each challenge.

2. **Remediation Suggestions**:
   - Practical steps to fix or mitigate the identified vulnerabilities.

3. **Ethical Hacking Report**:
   - Importance of proper authorization.
   - Legal and ethical boundaries.
   - Responsible disclosure practices.

###### Is the documentation clear, well-structured, and complete?

###### Does the documentation reflect the student's thought process and understanding?

##### Disassemble and Explain the Binaries

> **Important:** Decompilers are **NOT allowed**. Students must use disassemblers only.

**Ghidra Usage Clarification:**

If the student used Ghidra, verify they are using only the **Listing View (Assembly)**. The **Decompiler Window (which shows C code) must be closed** during the audit demonstration.

- **Allowed:** Ghidra Listing View, GDB, PEDA, Radare2
- **NOT Allowed:** Ghidra Decompiler Window, any tool that shows C/pseudocode

Ask the student to demonstrate their analysis using only assembly view.

###### If using Ghidra, is the Decompiler Window closed and only the Listing View (Assembly) being used?

###### Has the student successfully disassembled all the binaries?

###### Can the student explain the purpose and functionality of the binaries?

###### Did the student demonstrate an understanding of reverse engineering principles and binary mechanics?

##### Assembly Code Comprehension Test

> **Purpose:** This test ensures the student genuinely understands the assembly code and did not simply find a solution online.

Pick a 5-10 line block of assembly from one of the binaries and ask the student to explain exactly what the CPU is doing. The student should be able to answer questions such as:

- "What value is being moved into EAX/RAX here?"
- "What does this CMP instruction compare, and what happens based on the result?"
- "What is the purpose of this PUSH/POP sequence?"
- "What does this CALL instruction do, and where does execution go?"
- "What is this LEA instruction calculating?"
- "How does this JMP/JNE/JE instruction affect the program flow?"

The student should explain the instructions without referring to decompiled C code.

###### Can the student correctly explain what a 5-10 line block of assembly code does?

###### Does the student understand the purpose of individual instructions (MOV, CMP, JMP, CALL, etc.)?

###### Can the student explain how the assembly relates to the vulnerability being exploited?

###### Does the student demonstrate genuine understanding rather than memorized answers?

##### Exploit the Binaries

1. Navigate to `/opt/hole-in-bin` inside the VM.
2. Analyze and exploit each binary according to the guidelines in the corresponding `README.txt` files.

> **Note:** Students must not use external scripts, automated exploitation tools, or decompilers.

###### Did the student successfully exploit all the binaries?

###### Did the student demonstrate a clear understanding of binary exploitation techniques?

###### Can the student explain the vulnerability type and exploitation method for each binary?

#### Bonus

###### + Did the student explore alternative exploitation paths and document them?

###### + Did the student write custom shellcode for any of the exploits?

###### + Did the student demonstrate Return-Oriented Programming (ROP) techniques?

###### + Is this project an outstanding project that exceeds the basic requirements?

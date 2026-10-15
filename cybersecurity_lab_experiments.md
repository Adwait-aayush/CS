# 3. Command Injection

Command injection occurs when a web application passes unsanitized user input to an operating-system command. This experiment demonstrates the vulnerability using DVWA on Metasploitable 2.

## Requirements

- Oracle VirtualBox
- Kali Linux virtual machine
- Metasploitable 2 virtual machine
- Firefox in Kali Linux

## Procedure

### Step 1: Download Metasploitable 2

1. Open the official project page: https://sourceforge.net/projects/metasploitable/
2. Download `Metasploitable2-Linux.zip`.
3. Extract the ZIP file. The extracted folder should contain files such as `Metasploitable.vmdk` and `Metasploitable.vmx`.

### Step 2: Create the Metasploitable virtual machine

1. Open VirtualBox and click **New**.
2. Set the name to `Metasploitable`.
3. Select **Linux** as the type and **Ubuntu (32-bit)** or **Other Linux (32-bit)** as the version, depending on the options available.
4. Leave the ISO image empty.
5. Under the virtual hard disk options, choose **Use an existing virtual hard disk file**.
6. Browse to the extracted folder and select `Metasploitable.vmdk`.
7. Finish creating the VM.

### Step 3: Configure the virtual network

1. Power off both Kali and Metasploitable.
2. In VirtualBox, open **Metasploitable → Settings → Network → Adapter 1**.
3. Set **Attached to** to **Host-only Adapter** and select a host-only network.
4. Open **Kali → Settings → Network → Adapter 1** and select the same host-only network.
5. Make sure the host-only network has DHCP enabled, or configure both VMs with unique IP addresses on the same subnet.
6. Do not use Bridged Networking for this intentionally vulnerable VM; it may expose vulnerable services to other devices on your LAN.

### Step 4: Start the VMs and find the Metasploitable IP address

1. Start Metasploitable.
2. Log in using:
   - Username: `msfadmin`
   - Password: `msfadmin`
3. Run:
   ```bash
   ifconfig
   ```
4. Find the IPv4 address listed for `eth0`. Record that address; it will vary by setup.
5. Start Kali Linux.

### Step 5: Test connectivity from Kali

Open a terminal in Kali and run the following, replacing the placeholder with the address found in Step 4:

```bash
ping -c 4 <METASPLOITABLE-IP>
```

If the ping fails, verify that both VMs use the same host-only network and have addresses in the same subnet.

### Step 6: Open DVWA

1. In Kali, open Firefox.
2. Visit:
   ```text
   http://<METASPLOITABLE-IP>/dvwa
   ```
   Replace `<METASPLOITABLE-IP>` with the actual address from Step 4.
3. Log in with the default credentials used by this lab:
   - Username: `admin`
   - Password: `password`
4. If DVWA displays **Create / Reset Database**, click it to initialize the database. Then log in again if prompted.

DVWA is accessed from the Metasploitable 2 web server for this lab; a separate DVWA installation is not part of these steps.

### Step 7: Set DVWA security level

1. In DVWA, click **DVWA Security** in the left menu.
2. Select **Low**.
3. Click **Submit**.

Use this intentionally vulnerable setting only in the isolated lab.

### Step 8: Demonstrate command injection

1. Click **Command Execution** in the left menu.
2. Enter:
   ```text
   127.0.0.1
   ```
3. Click **Submit** and confirm that ping output appears.
4. Next, enter each of the following separately and click **Submit** after each:
   ```text
   127.0.0.1; pwd
   ```
   ```text
   127.0.0.1; whoami
   ```
   ```text
   127.0.0.1; ls -la
   ```
5. Observe whether the application executes the additional command and displays its output. The exact output depends on the VM and application configuration.

### Step 9: Result and mitigation

The lab demonstrates that unsanitized input can cause the web application to execute an additional operating-system command. The commands run with the privileges of the web-server process, not automatically as root.

In real applications, avoid building shell commands by concatenating user input. Prefer safe APIs and argument arrays, validate input against a strict allowlist, and run services with least privilege.

---

# 4. Buffer Overflow Illustration Using GCC

## Objective

Understand input handling and memory-safety concepts in a small C program.

## Step 1: Create the C program

Open a terminal in Kali Linux or Ubuntu and run:

```bash
nano buffer.c
```

Enter the following program:

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    char buff[15];
    int pass = 0;

    printf("Enter the password:\n");

    if (fgets(buff, sizeof(buff), stdin) == NULL) {
        return 1;
    }

    buff[strcspn(buff, "\n")] = '\0';

    if (strcmp(buff, "thegeekstuff") == 0) {
        printf("Correct Password\n");
        pass = 1;
    } else {
        printf("Wrong Password\n");
    }

    if (pass) {
        printf("Root privileges given to the user\n");
    }

    return 0;
}
```

Save with **Ctrl+O**, press **Enter**, and exit with **Ctrl+X**.

## Step 2: Compile the program

Run:

```bash
gcc -g -fno-stack-protector -z execstack -o buffer buffer.c
```

- `-g` includes debugging symbols.
- `-fno-stack-protector` disables stack-protector instrumentation.
- `-z execstack` marks the stack as executable where supported.

These options weaken security protections and are intended only for a disposable lab demonstration.

## Step 3: Run the program

Run:

```bash
./buffer
```

### Test 1: Correct password

Enter:

```text
thegeekstuff
```

Expected output:

```text
Correct Password
Root privileges given to the user
```

### Test 2: Incorrect password

Run `./buffer` again and enter:

```text
hello
```

Expected output:

```text
Wrong Password
```

### Test 3: Enter a long string

Run `./buffer` again and enter a long string, for example:

```text
Hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
```

**Important correction:** The exact program above uses `fgets(buff, sizeof(buff), stdin)`, which reads at most 14 input bytes plus the terminating NUL byte into the 15-byte buffer. A long line is truncated by this call; it does **not** overflow `buff` through this input path. Therefore, a crash or variable overwrite is not an expected result for this code. The message about “root privileges” is only printed text; the program does not grant operating-system privileges.

## Result

The program demonstrates password comparison and bounded input. As written, it does not demonstrate a working buffer overflow.

---

# 5. Compare Two Files for Forensic Investigation

## Objective

Compare two text files and identify added, removed, or modified content.

## Software required

- Windows operating system
- Notepad
- Compare It!: https://www.grigsoft.com/wincmp3.htm

## Step 1: Create the first text file

1. Open Notepad.
2. Enter the following content:

   ```text
   Student Name: Rahul
   USN: 1DS22CS001
   Department: CSE
   Semester: 7
   College: DSCE
   ```

3. Select **File → Save As**.
4. Save the file as `File1.txt` on the Desktop or in another easy-to-find folder.

## Step 2: Create the second text file

1. Open another Notepad window.
2. Enter the following content:

   ```text
   Student Name: Rahul Kumar
   USN: 1DS22CS001
   Department: Computer Science
   Semester: 7
   College: DSCE Bangalore
   ```

3. Save the file as `File2.txt` in the same folder as `File1.txt`.

## Step 3: Install Compare It!

1. Open the official website: https://www.grigsoft.com/wincmp3.htm
2. Download Compare It! from that page.
3. Run the installer.
4. Follow the installation prompts and finish setup.

## Step 4: Open Compare It!

1. Launch Compare It! from the Start Menu or Desktop shortcut.
2. The application should display two file panes.

## Step 5: Load the first file

1. Use the Open/Browse control for the left pane.
2. Navigate to the folder containing `File1.txt`.
3. Select `File1.txt` and open it.

## Step 6: Load the second file

1. Use the Open/Browse control for the right pane.
2. Select `File2.txt` from the same folder.
3. Open it.

## Step 7: Compare the files

The application compares the files and highlights differences. The exact highlight colors may vary by version and theme.

## Step 8: Analyze the differences

Observe changes such as:

- `Rahul` changed to `Rahul Kumar`
- `CSE` changed to `Computer Science`
- `DSCE` changed to `DSCE Bangalore`

## Step 9: Navigate between differences

Use the toolbar controls to move to the next and previous differences and inspect each change.

## Step 10: Generate a report if required

If your installed version offers a print, print-preview, or report option, use it to save or print the comparison results for documentation.

## Step 11: Close the application

Review the results, save any required report, and close Compare It!.

## Result

The two files were compared and their modified fields identified. For actual forensic investigations, preserve the original files, work on verified copies, and document file hashes and chain of custody.

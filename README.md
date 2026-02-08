
## Project Overview
This project simulates a network traffic analysis scenario to demonstrate the security risks of unencrypted network protocols (HTTP). The objective was to provision a corporate-grade Windows 10 Enterprise endpoint from scratch, integrate it into a live network environment using bridged networking, and intercept cleartext authentication credentials using packet analysis tools.

This lab emphasizes **virtualization infrastructure**, **network engineering**, and **packet-level forensics**.

## Tools & Technologies
* **Hypervisor:** Oracle VirtualBox (Version 7.x)
* **Operating System:** Windows 10 Enterprise (Evaluation ISO)
* **Network Analysis:** Wireshark
* **Protocols:** HTTP, TCP, IPv4, DHCP
* **Target Application:** `testphp.vulnweb.com` (Intentional vulnerable web application)

## Architecture
* **Host Machine:** Workstation (The "Analyst")
* **Guest Machine:** Windows 10 Enterprise VM (The "Victim")
* **Network Config:** Bridged Adapter with Promiscuous Mode enabled (Allow All)

---

## Challenges & Troubleshooting (The "Real World" Experience)
This project required significant troubleshooting to overcome infrastructure and configuration hurdles. Below are the specific challenges faced and how they were resolved:

### 1. Provisioning Failure & ISO Manual Build
* **The Issue:** The initial plan to import a pre-packaged VM failed due to broken vendor links and version incompatibilities.
* **The Solution:** Pivoted strategy to **manual provisioning**. I located the official Windows 10 Enterprise ISO from the Microsoft Evaluation Center and built the VM from the ground up, manually configuring the virtual CPU/RAM and attaching the ISO to the virtual optical drive.

### 2. Network Driver Incompatibility
* **The Issue:** The VM initially failed to bridge to the Wi-Fi adapter, preventing the host from capturing traffic.
* **The Solution:** I manually installed the **VirtualBox NDIS6 Bridged Networking Driver** within the host's network adapter properties, forcing the physical card to support the virtual bridge.

### 3. Signal-to-Noise Ratio in Packet Capture
* **The Issue:** Early Wireshark captures were flooded with SSDP and UPnP "noise" from the home router (Destination `10.0.0.1`), making it impossible to find the target traffic.
* **The Solution:** I implemented precise display filters to isolate the relevant data.
    * *Attempt 1:* Filtered by IP (`ip.addr == 10.0.0.2`), but this still included background OS chatter.
    * *Attempt 2:* Filtered by payload content (`frame contains "uname"`), but this captured encrypted HTTPS retransmissions.
    * *Final Success:* Applied the application-layer filter `http.request.method == POST`. This instantly isolated the specific HTTP packet containing the login form submission.

---

## Evidence & Analysis

### 1. Infrastructure Provisioning
**Status:** *Success*
The following screenshots demonstrate the successful manual installation of the Windows 10 Enterprise environment.
* **Boot Process:** The VM successfully booting from the manually attached ISO file.
* **OS Installation:** The "Installing Windows" status screen confirming the custom partition setup was accepted.

<img width="1052" height="924" alt="screenshot 7" src="https://github.com/user-attachments/assets/5a84ec6c-4e1f-4438-ab90-1318a5b7095a" />

### 2. Network Engineering & Troubleshooting
**Status:** *Resolved*
Getting the VM to communicate on the physical network required manual driver configuration.
* **Driver Install:** Manually adding the "VirtualBox NDIS6 Bridged Networking Driver" to the host Wi-Fi properties to enable traffic bridging.
* **Adapter Config:** Configuring the VirtualBox network settings to "Bridged Adapter" and—crucially—setting Promiscuous Mode to "Allow All" so the host could sniff the guest's packets.
* **Verification:** The `ipconfig` command inside the VM confirms it pulled a real LAN IP (`10.0.0.2`) from the physical router, bypassing the default NAT isolation.


<img width="778" height="976" alt="screenshot 5" src="https://github.com/user-attachments/assets/ec61bc4f-a87c-4e95-a522-a85cb0b2048f" />

<img width="1158" height="906" alt="Screenshot 9" src="https://github.com/user-attachments/assets/cb84fc48-0c51-4b4c-b2ae-90931e448083" />

### 3. The Interception (Packet Analysis)
**Status:** *Compromised*
The final phase involved capturing the authentication traffic.
* **The Target:** The vulnerable web application `testphp.vulnweb.com` accessed via HTTP.
* **The Capture:** Using Wireshark with the filter `http.request.method == POST` to ignore background router noise and isolate the login request.
* **The Payload:** The "Follow TCP Stream" window reveals the reconstructed data stream. The username (`admin`) and password (`MySecret123`) are clearly visible in the body of the POST request, proving the vulnerability of unencrypted authentication.

<img width="2266" height="1584" alt="screenshot 13" src="https://github.com/user-attachments/assets/ee30fe51-69a1-4377-b151-bc1a25145530" />

---

## Key Skills Learned
* **Endpoint Provisioning:** Configuring virtual hardware and installing enterprise operating systems from raw media (ISO).
* **Network Troubleshooting:** Diagnosing IP addressing issues, installing NDIS drivers, and understanding the difference between NAT and Bridged networking.
* **Packet Analysis:** Reading TCP/IP headers and payloads to reconstruct user sessions.
* **Protocol Knowledge:** Practical demonstration of why HTTP (Port 80) is insecure compared to HTTPS (Port 443).
* **Persistence:** Successfully navigating vendor errors and configuration failures to achieve the lab objective.

# Cyber Security Lab 2
## Man in the Middle MITM Attack

## Lab Description
The objective of this lab is to execute a Man-in-the-Middle (MITM) attack within a switched network environment consisting of three nodes: Alice, Bob, and Mallory.

The goal is to insert Mallory between Alice and Bob to intercept communications. This is a "passive" interception in terms of data handling—the objective is to steal messages and save them to `secret.txt` without alerting Alice or Bob to the breach. To achieve this, an ARP spoofing technique is employed.

## About ARP Spoofing
Address Resolution Protocol (ARP) spoofing is a technique where an attacker sends forged ARP messages onto a Local Area Network (LAN). The aim is to associate the attacker's MAC address with the IP address of a legitimate network device (the target).

In this lab, Mallory sends forged ARP replies to Alice. These replies falsely assert that Mallory’s MAC address belongs to Bob’s IP address. Consequently, Alice updates her ARP cache with this malicious entry. When Alice attempts to send data to Bob, the Ethernet frames are addressed to Mallory’s MAC address instead of Bob's, allowing Mallory to intercept the packets.

## Preparation

Before launching the attack, network information for all parties was gathered.

### Step 1 Attacker Information (Mallory)

Using the command `ifconfig -a`, Mallory's network details was identified on interface `eth1`:
* **IP Address:** `10.10.55.4`
* **MAC Address:** `02:42:0a:0a:37:04`

### Step 2 Target Information (Alice and Bob)
To identify the targets, `tcpdump` was used to passively sniff network traffic on `eth1`:
sudo tcpdump -i eth1 -n -XX

### Step 3 To understand Alice's ARP request and how to prepare the ARP reply
Use "sudo tcpdump -i eth1 -n -XX" command to capture and display the packets through eth1. This command can display the data content of the ARP request.

## Step 4 Commands and Tools
Tools used in the setup:
- "tcpdump" command to capture and display the live packets through the specific network interface and host ip.
- "grep" command to search a specific text string (i.e. "CTF{") and output the lines into a file (i.e. secret.txt)
- "raw_packet" script to generate and send a ARP reply to Alice's ARP request.  Four parameters are required for raw_packet, they are: 1) eth1 (network interface), 2) 02:42:0a:0a:37:02 (destination (Alice) Mac address), 0x080 (ARP protocol type) and payload file (data.bin, ARP reply data).  
- "hexedit" to edit the ARP reply payload file (i.e. data.bin) in hex based on ARP reply to cheat Alice's request that Mac address of IP 10.10.55.3 is 02:42:0a:0a:37:04 (Mallory's Mac address)
- "arp" to display the ARP cache which maps ip address to Mac address (if any)
- "tail -f secret.txt" to view the content of the secret.txt to ensure that packets are captured in the file.

## Step 5 Steps to solve the task
After completing the preparations stated above:

1. Use "hexedit"to edit the ARP reply payload file (data.bin) based on ARP reply standard
2. Write and execute a bash script "mitm_script.sh" to send the forged ARP reply and capture the secret packets to save them into "secret.txt"

  About "mitm\_script.sh" script
  i. Initialization and preparation
    The script defines variables required. The script removes any previous output file to ensure a clean capture environment.
  ii. Starting Packet Capture

  A background tcpdump process is launched to monitor packets addressed to Bob’s IP. Output is displayed in ASCII mode and piped through grep to detect specific patterns (e.g., "CTF{"). Matching lines are saved  into a results file (secret.txt)
"sudo tcpdump -i "$INTERFACE" host "$BOBIP" -l -A -n 2>/dev/null | grep --line-buffered "CTF{" > "$OUTPUT_FILE" \&"
  -l : Make stdout line buffered
  -i : Network Interface eth1
  -A : output data in ASCII (to see the "CTF{" text)
  -n : Don't convert addresses to names

  iii. ARP Spoofing Phase
  
  In parallel, Mallory sends multiple forged ARP reply packets to Alice for every 2 seconds for 5 times. It can ensure that packets are captured during this phase. Each packet asserts that Bob’s IP corresponds to Mallory’s MAC address, causing Alice to overwrite her ARP table entry with this spoofed mapping.

  iv. User-Controlled Termination
    The script waits for user input. When stopped, it terminates the packet capture process and clears command history.

  v. Observed Results
    When Alice transmits traffic intended for Bob, the spoofing causes those packets to be routed through Mallory.  Any matching data extracted by tcpdump (with text string "CTF{") is stored in the output file, confirming that the MITM attack succeeded under the lab conditions.

3. In another windows, while "mitm_script.sh" is running, use "tail -f secret.txt" to view content of secret.txt that ensure the capture is successful.
4. Captured secret messages saved in "secret.txt"

### Other settings: Forwarding captured packet to Bob
From the captured packet at Bob IP, it noticed that the captured packets are automatically forwarded from Mallory's Mac address to Bob's Mac address. This forwarding can make Bob and Alice not notice the capture.
The forwarding was done automatically and should be configurated in the packet forward table but not set by this task.

### Other settings: Bob's IP is used for tcpdump to capture the packets
The capture must happen on Mallory's device. Mallory is the one running the script. The task is to capture the packet from Alice to Bob (the Destination IP is 10.10.55.3). Hence, tcpdump is capturing at host 10.10.55.3.


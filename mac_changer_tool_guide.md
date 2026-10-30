# 🛡️ Ultimate MAC Changer - Comprehensive Development & Usage Guide

---

## 📌 Table of Contents
1. [Introduction & Overview](#-introduction--overview)
2. [Manual vs. Automated Approach](#-manual-vs-automated-approach)
3. [Accessing and Running Your Project via Terminal](#-accessing-and-running-your-project-via-terminal)
4. [Final Python Script Code](#-final-python-script-code)
5. [Code Architecture & Advanced Features](#-code-architecture--advanced-features)
6. [Quick Usage Guide & Command Examples](#-quick-usage-guide--command-examples)
7. [Future Roadmap & Enhancement Ideas](#-future-roadmap--enhancement-ideas)

---

## 🎯 Introduction & Overview

A **MAC Address** (Media Access Control) is a unique physical identifier assigned to network interfaces, consisting of 12 hexadecimal characters divided into 6 pairs (e.g., `00:0c:29:95:73:6a`).

### 💡 Why Change the MAC Address?
- **Privacy & Security:** Prevents device tracking across local networks (LAN/Wi-Fi).
- **Penetration Testing:** Bypasses MAC filtering mechanisms or simulates authorized network nodes.
- **Digital Hygiene:** Obscures the original hardware identity during assessments and security research.

---

## ⚔️ Manual Approach vs. Automated Tool

### 1️⃣ Manual Terminal Workflow
To manually change a MAC address step-by-step using native Linux commands:

```bash
# 1. Display current MAC and verify interface name
ifconfig eth0

# 2. Bring down the network interface
sudo ifconfig eth0 down

# 3. Assign the new MAC address
sudo ifconfig eth0 hw ether 00:11:22:33:44:55

# 4. Verify the modification
ifconfig eth0

# 5. Bring the network interface back up
sudo ifconfig eth0 up
```

---

### 2️⃣ Comparison Table

| Comparison Criteria | Manual Method (`ifconfig`) | Python Tool (`mac_changer.py`) |
| :--- | :--- | :--- |
| **Speed** | Slow, requires executing 5 sequential commands | Instantaneous execution via automated flags |
| **Safety & Stability** | Risk of network disconnection if a malformed MAC is used | Automatically enforces **Unicast** generation to ensure stable routing |
| **Privilege Check** | Fails abruptly if `sudo` is omitted without proper warning | Validates root privileges upfront to prevent unhandled exceptions |
| **Input Validation** | No built-in validation against typo-prone inputs | Strictly validates syntax using Regular Expressions (Regex) |
| **Activity Logging** | No persistent history or operational audit trail | Automatically logs operations into `mac_changer.log` with precise timestamps |
| **User Interface** | Plain, unstyled text output | Interactive CLI menu featuring ANSI-colorized, clean output |

---

## 🖥️ Accessing and Running Your Project via Terminal

To navigate to your project directory and execute the tool on Kali Linux:

1. **Navigate to the Project Directory:**
   ```bash
   cd /home/kali/PycharmProjects/PythonProject
   ```

2. **List Directory Contents:**
   ```bash
   ls -la
   ```

3. **Run the Script:**
   ```bash
   sudo python3 mac_changer.py
   ```

---

## 💻 Final Python Script Code

```python
import subprocess
import argparse
import re
import random
import os
import sys
import datetime

# --- ANSI Terminal Colors for Professional Output ---
GREEN = "\033[92m"
RED = "\033[91m"
BLUE = "\033[94m"
YELLOW = "\033[93m"
RESET = "\033[0m"

# 1. Privilege Check: Ensure the script is run with Root (sudo) privileges
if os.geteuid() != 0:
    print(f"{RED}[-] Error: This script must be run as root (sudo).{RESET}")
    sys.exit(1)

# Function to check if the network interface actually exists on the system
def interface_exists(interface):
    try:
        return os.path.exists(f"/sys/class/net/{interface}")
    except Exception:
        return False

# Function to get the current MAC address of the interface using 'ip link'
def get_current_mac(interface):
    try:
        result = subprocess.check_output(["ip", "link", "show", interface], universal_newlines=True)
        mac_match = re.search(r"link/ether\s+([0-9a-fA-F:]+)", result)
        if mac_match:
            return mac_match.group(1)
    except subprocess.CalledProcessError:
        return None
    return None

# Function to validate the format of a given MAC address using Regular Expressions
def is_valid_mac(mac):
    pattern = r'^([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})$'
    return re.match(pattern, mac) is not None

# Function to generate a random Unicast MAC address and ensure stability
def generate_random_unicast_mac():
    # Generate 6 random bytes represented as two-digit hexadecimal strings
    mac_parts = [format(random.randint(0, 254), '02x') for _ in range(6)]
    
    # Convert the first byte from hex to integer and apply a bitwise AND with 0xFE
    # This guarantees the least significant bit of the first octet is 0 (Unicast address)
    first_byte_int = int(mac_parts[0], 16) & 0xFE 
    mac_parts[0] = format(first_byte_int, '02x')
    
    return ":".join(mac_parts)

# Function to log tool activities into a text file (mac_changer.log)
def log_activity(interface, old_mac, new_mac):
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    log_entry = f"[{timestamp}] Interface: {interface} | Old MAC: {old_mac} | New MAC: {new_mac}\n"
    try:
        with open("mac_changer.log", "a") as log_file:
            log_file.write(log_entry)
    except Exception as e:
        print(f"{YELLOW}[-] Warning: Could not write to log file: {e}{RESET}")

# Function to change the MAC address of a specified network interface
def change_mac(interface, new_mac):
    if not interface_exists(interface):
        print(f"{RED}[-] Error: Network interface '{interface}' does not exist!{RESET}")
        sys.exit(1)

    old_mac = get_current_mac(interface)
    if old_mac:
        print(f"{BLUE}[*] Current MAC for {interface}: {old_mac}{RESET}")
    else:
        print(f"{YELLOW}[-] Warning: Could not retrieve current MAC for {interface}.{RESET}")

    try:
        print(f"{BLUE}[*] Changing MAC for {interface} to {new_mac}...{RESET}")
        
        # Step 1: Bring the network interface down
        subprocess.run(["sudo", "ip", "link", "set", "dev", interface, "down"], check=True)
        
        # Step 2: Assign the new MAC address to the interface
        subprocess.run(["sudo", "ip", "link", "set", "dev", interface, "address", new_mac], check=True)
        
        # Step 3: Bring the network interface back up
        subprocess.run(["sudo", "ip", "link", "set", "dev", interface, "up"], check=True)
        
        print(f"{GREEN}[+] Success! MAC address for {interface} is now {new_mac}{RESET}")

        # Step 4: Verify if the change was successful
        updated_mac = get_current_mac(interface)
        if updated_mac and updated_mac.lower() == new_mac.lower():
            print(f"{GREEN}[+] Verification Passed: MAC address successfully updated and verified.{RESET}")
        else:
            print(f"{YELLOW}[-] Warning: Verification failed or mismatch detected.{RESET}")

        # Step 5: Log the operation
        if old_mac:
            log_activity(interface, old_mac, new_mac)
        
    except subprocess.CalledProcessError as e:
        print(f"{RED}[-] Error executing system command: {e}{RESET}")

# Function to reset the interface
def reset_interface(interface):
    if not interface_exists(interface):
        print(f"{RED}[-] Error: Network interface '{interface}' does not exist!{RESET}")
        sys.exit(1)
        
    print(f"{BLUE}[*] Resetting interface {interface}...{RESET}")
    try:
        subprocess.run(["sudo", "ip", "link", "set", "dev", interface, "down"], check=True)
        subprocess.run(["sudo", "ip", "link", "set", "dev", interface, "up"], check=True)
        print(f"{GREEN}[+] Interface {interface} reset completed (reverted to default kernel state).{RESET}")
    except subprocess.CalledProcessError as e:
        print(f"{RED}[-] Error during reset: {e}{RESET}")

# Interactive Menu when running without arguments
def interactive_mode():
    print(f"{BLUE}=== ULTIMATE MAC CHANGER (INTERACTIVE MODE) ==={RESET}")
    interface = input(f"{YELLOW}[?] Enter network interface name (e.g., eth0, wlan0): {RESET}").strip()
    
    if not interface_exists(interface):
        print(f"{RED}[-] Error: Interface '{interface}' does not exist!{RESET}")
        sys.exit(1)
        
    print("\nChoose an option:")
    print("1. Generate and set a Random Unicast MAC")
    print("2. Set a Custom MAC address")
    print("3. Reset Interface")
    
    choice = input(f"{YELLOW}[?] Enter your choice (1-3): {RESET}").strip()
    
    if choice == "1":
        target_mac = generate_random_unicast_mac()
        change_mac(interface, target_mac)
    elif choice == "2":
        custom_mac = input(f"{YELLOW}[?] Enter custom MAC address: {RESET}").strip()
        if is_valid_mac(custom_mac):
            change_mac(interface, custom_mac)
        else:
            print(f"{RED}[-] Error: Invalid MAC address format!{RESET}")
    elif choice == "3":
        reset_interface(interface)
    else:
        print(f"{RED}[-] Invalid choice. Exiting.{RESET}")

# Main execution block
if __name__ == "__main__":
    try:
        parser = argparse.ArgumentParser(description="Ultimate Unicast MAC Changer with Advanced Features")
        
        parser.add_argument("-i", "--interface", dest="interface", help="Network interface name (e.g., eth0, wlan0)")
        parser.add_argument("-m", "--mac", dest="mac", help="Specific custom MAC address to set")
        parser.add_argument("-r", "--random", dest="random", action="store_true", help="Generate and set a random Unicast MAC")
        parser.add_argument("-R", "--reset", dest="reset", action="store_true", help="Reset MAC address back to default")

        args = parser.parse_args()

        # If no arguments provided, launch Interactive Mode
        if len(sys.argv) == 1:
            interactive_mode()
        else:
            if not args.interface:
                print(f"{RED}[-] Error: Network interface (-i/--interface) is required!{RESET}")
                sys.exit(1)
                
            if args.reset:
                reset_interface(args.interface)
            elif args.random:
                target_mac = generate_random_unicast_mac()
                change_mac(args.interface, target_mac)
            elif args.mac:
                if is_valid_mac(args.mac):
                    change_mac(args.interface, args.mac)
                else:
                    print(f"{RED}[-] Error: '{args.mac}' is not a valid MAC address format!{RESET}")
            else:
                print(f"{RED}[-] Error: Missing action option (-m, -r, or -R). Use -h for help.{RESET}")

    except KeyboardInterrupt:
        print(f"\n{YELLOW}[!] Program interrupted by user. Exiting safely...{RESET}")
        sys.exit(0)
```

---

## 🔍 Code Architecture & Advanced Features

1. **Automatic Root Enforcement (`os.geteuid() != 0`):**
   Prevents unauthorized execution by ensuring the script runs with administrative privileges, failing gracefully with a readable prompt instead of raw system stack traces.

2. **Secure Unicast Generation (`& 0xFE`):**
   Applies a bitwise AND operation with `0xFE` on the first octet, clearing the multicast/locally administered bit. This ensures generated addresses are valid Unicast MACs that prevent routing drops or adapter conflicts.

3. **Operational Audit Logging:**
   Appends all successful state changes to `mac_changer.log`, capturing timestamps, interface names, and historical transitions (Old vs. New MAC).

4. **Post-Execution Verification:**
   Queries the interface state post-assignment to confirm that the OS kernel successfully applied the requested alteration.

5. **Professional CLI Experience:**
   Utilizes ANSI escape sequences for color-coded status messages and implements a robust interactive fallback menu when initialized without command-line arguments.

---

## 🚀 Quick Usage Guide & Command Examples

- **Launch Interactive Menu:**
  ```bash
  sudo python3 mac_changer.py
  ```

- **Assign a Random Unicast MAC:**
  ```bash
  sudo python3 mac_changer.py -i eth0 -r
  ```

- **Assign a Custom MAC Address:**
  ```bash
  sudo python3 mac_changer.py -i eth0 -m 00:11:22:33:44:55
  ```

- **Reset Interface to Default State:**
  ```bash
  sudo python3 mac_changer.py -i eth0 -R
  ```

---

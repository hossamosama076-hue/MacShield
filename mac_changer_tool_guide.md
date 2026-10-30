# 🛡️ الدليل الشامل لإنشاء وتطوير أداة تغيير الـ MAC Address (Ultimate MAC Changer)

---

## 📌 جدول المحتويات
1. [المقدمة والنظرة العامة](#-المقدمة-والنظرة-العامة)
2. [طريقة التغيير اليدوي مقارنة بالأداة البرمجية](#-طريقة-التغيير-اليدوي-مقارنة-بالأداة-البرمجية)
3. [كيفية الوصول إلى مشروعك وتشغيله عبر التيرمينال](#-كيفية-الوصول-إلى-مشروعك-وتشغيله-عبر-التيرمينال)
4. [الكود البرمجي النهائي (Python Script)](#-الكود-البرمجي-النهائي-python-script)
5. [شرح مكونات الكود والميزات المتقدمة](#-شرح-مكونات-الكود-والميزات-المتقدمة)
6. [دليل الاستخدام السريع والأوامر التجريبية](#-دليل-الاستخدام-السريع-والأوامر-التجريبية)
7. [أفكار لتطوير الأداة مستقبلاً (Future Roadmap)](#-أفكار-لتطوير-الأداة-مستقبلاً-future-roadmap)

---

## 🎯 المقدمة والنظرة العامة

عنوان **MAC Address** (Media Access Control) هو المعرف الفيزيائي الخاص بكرت الشبكة ويتكون من 12 رمزاً سداسي عشر (Hexadecimal) مقسمة على 6 أجزاء (مثل: `00:0c:29:95:73:6a`).

### 💡 لماذا نغير الـ MAC Address؟
* **الخصوصية والحماية:** لمنع تتبع الجهاز داخل الشبكات المحلية (LAN/Wi-Fi).
* **اختبارات الاختراق (Penetration Testing):** لتجاوز فلاتر الشبكة (MAC Filtering) أو محاكاة أجهزة أخرى مصرح لها بالاتصال.
* **الأمان الرقمي:** إخفاء الهوية الأصلية للجهاز أثناء التحقيقات أو التقييمات الأمنية.

---

## ⚔️ طريقة التغيير اليدوي مقارنة بالأداة البرمجية

### 1️⃣ الطريقة اليدوية (Manual Terminal Steps)

إذا أردت تغيير الـ MAC يدويًا خطوة بخطوة من التيرمينال:

```bash
# 1. عرض الـ MAC الحالي ومعرفة اسم الكرت
ifconfig eth0

# 2. إيقاف كرت الشبكة مؤقتاً
sudo ifconfig eth0 down

# 3. إدخال عنوان الـ MAC الجديد
sudo ifconfig eth0 hw ether 00:11:22:33:44:55

# 4. التأكد من التغيير قبل التشغيل
ifconfig eth0

# 5. إعادة تشغيل كرت الشبكة
sudo ifconfig eth0 up
```

---

### 2️⃣ جدول مقارنة: الطريقة اليدوية vs الأداة البرمجية المتقدمة

| وجه المقارنة | الطريقة اليدوية (`ifconfig`) | أداة بايثون (`mac_changer.py`) |
| :--- | :--- | :--- |
| **السرعة** | بطيئة وتتطلب كتابة 5 أوامر متتالية | سريعة جداً بضغطة زر واحدة أو أمر واحد |
| **الأمان والسلامة** | قد تسبب مشاكل بالشبكة لو الـ MAC عشوائي وغير صحيح | تضمن توليد MAC من نوع **Unicast** لضمان استقرار الاتصال |
| **فحص الصلاحيات** | يفشل الأمر إذا نسيت `sudo` دون تنبيه مسبق | تفحص صلاحيات الـ Root تلقائياً وتمنع الأخطاء |
| **التحقق من المدخلات** | لا توجد حماية ضد كتابة MAC خاطئ | تفحص الـ Syntax باستخدام التعبيرات المنتظمة (Regex) |
| **التوثيق (Logging)** | لا توجد سجلات للأعمال السابقة | تقوم بتسجيل كل عملية في ملف `mac_changer.log` بوقت وتاريخ دقيق |
| **الواجهة** | نصية جافة | تحتوي على **Interactive Menu** وألوان مريحة ومبهرة |

---

## 🖥️ كيفية الوصول إلى مشروعك وتشغيله عبر التيرمينال

للوصول إلى المجلد الذي تحتوي عليه الأدوات الخاصة بك وتشغيلها في نظام كالي لينكس (Kali Linux):

1. **الانتقال إلى مسار المشروع:**
   ```bash
   cd /home/kali/PycharmProjects/PythonProject
   ```

2. **عرض الملفات الموجودة بالمجلد:**
   ```bash
   ls -la
   ```

3. **تشغيل الأداة البرمجية:**
   ```bash
   sudo python3 mac_changer.py
   ```

---

## 💻 الكود البرمجي النهائي (Python Script)

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

## 🔍 شرح مكونات الكود والميزات المتقدمة

1. **فحص الـ Root تلقائياً (`os.geteuid() != 0`):**
   يمنع تشغيل السكربت دون صلاحيات الجذر ويخرج برسالة خطأ واضحة بدلاً من ظهور أخطاء نظام غير مفهومة.

2. **توليد عنوان Unicast آمن (`& 0xFE`):**
   تطبيق عملية Bitwise AND مع `0xFE` على أول بايت لتصفير البت الأخير، مما يضمن أن الـ MAC المولّد عشوائياً هو Unicast آمن ولا يتسبب في فصل كرت الشبكة أو مشاكل في الـ Routing.

3. **حفظ السجلات (Activity Logging):**
   تسجيل كل عملية تغيير ناجحة داخل ملف `mac_changer.log` متبوعة بالتاريخ، الوقت،اسم الكرت، والـ MAC القديم والجديد.

4. **التحقق المستمر (Verification Step):**
   تقوم الأداة بإعادة قراءة كرت الشبكة بعد التغيير للتحقق الفعل من استجابة النظام للتعديل.

5. **الألوان والواجهة التفاعلية:**
   استخدام ANSI Escape Sequences لتلوين مخرجات التيرمينال وتوفير قائمة تفاعلية عند التشغيل السريع المباشر `sudo python3 mac_changer.py`.

---

## 🚀 دليل الاستخدام السريع والأوامر التجريبية

* **التشغيل بالوضع التفاعلي (Interactive Menu):**
  ```bash
  sudo python3 mac_changer.py
  ```

* **التغيير لعنوان عشوائي سريع:**
  ```bash
  sudo python3 mac_changer.py -i eth0 -r
  ```

* **التغيير لـ MAC مخصص:**
  ```bash
  sudo python3 mac_changer.py -i eth0 -m 00:11:22:33:44:55
  ```

* **إعادة ضبط الكرت (Reset):**
  ```bash
  sudo python3 mac_changer.py -i eth0 -R
  ```

---

## 🔮 أفكار لتطوير الأداة مستقبلاً (Future Roadmap)

إذا أردت تطوير الأداة في المستقل لإضافتها لمشاريع أعمق:
1. **OUI Vendor Spoofing:** إمكانية جعل الـ MAC يبدو وكأنه مصنع من شركة محددة (مثل جعل الجهاز يظهر كـ Apple أو Cisco أو Samsung).
2. **Scheduled Timer:** تغيير الـ MAC تلقائياً كل زمن محدد (مثلاً كل 10 دقائق) لزيادة الخصوصية.
3. **GUI Interface:** إضافة واجهة رسمية بسيطة باستخدام مكتبة `PyQt` أو `Tkinter`.
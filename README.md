# linux-cs2-macro
CS2 Macro for Linux made in Python 3
This cheat is impossible to be detected by VAC (UNLESS YOU SET THE INTERVALS TO THE SAME VALUE which the code wont let you do.)
Press Ctrl + C to exit
This script uses the `evdev` interface to capture and handle input events for macro operations. 

Because this script interacts directly with the Linux kernel's input subsystem (`/dev/input/`), it must be executed with **root/sudo** privileges.

## Prerequisites

Before installing the Python dependencies, your Linux system requires specific build tools, C compilers, and kernel development headers to successfully compile the `evdev` C-extensions.

### System Packages

Install the required tools (`gcc`, Python development headers, and kernel headers) using your system's package manager:

#### Debian / Ubuntu / Raspberry Pi OS
```bash
sudo apt update
sudo apt install python3 python3-pip python3-dev build-essential linux-headers-\$(uname -r)
```

#### Fedora / RHEL / CentOS
```bash
sudo dnf install python3 python3-pip python3-devel gcc kernel-devel
```

### Python Dependencies

Once the system prerequisites are installed, you can build and install the `evdev` package. Since the script must run as root, install the package globally using `sudo`:

```bash
sudo pip3 install evdev
```

## Usage

Run the macro script with `sudo` so it has permission to read the kernel input devices:

```bash
sudo python3 counter-macro.py
```

#### VERIFY REQUIRED SOFTWARE:
+Check Python version

    python3 --version

+Check if evdev is installed

    python3 -c "import evdev; print('evdev installed')"

+Check for input devices

    ls -la /dev/input/event*

+Check if uinput exists

    ls -la /dev/uinput

+Check group membership

    groups $USER  (Should show 'input' and 'uinput')

### THE CODE (FOR VISUAL PURPOSES ONLY, THE REAL CODE IS IN THE RELEASE PAGE!!!)

!/usr/bin/env python3

import evdev

from evdev import uinput, ecodes as e

import threading

import time

import random

import sys


### -------------------- Config --------------------

 YOU CAN CHANGE THESE VARIABLES TO YOUR LIKING!

MIN_INTERVAL = 0.15

MAX_INTERVAL = 0.35

### -------------Interval Value Warning-------------

if MIN_INTERVAL == MAX_INTERVAL:

print("Do NOT set the intervals the same value.")

sys.exit(1)

### -----------------Mouse Search-------------------

def find_mouse():

for path in evdev.list_devices():

dev = evdev.InputDevice(path)
        
if e.EV_KEY in dev.capabilities():
        
caps = dev.capabilities()[e.EV_KEY]
            
if e.BTN_LEFT in caps:
            
return dev
                
return None

real_mouse = find_mouse()

if not real_mouse:

print("No mouse found", file=sys.stderr)
sys.exit(1)

print(f"Running...")

real_mouse.grab()

### -----------------Virtual Mouse-----------------
 
virtual = uinput.UInput({

e.EV_KEY: [e.BTN_LEFT, e.BTN_RIGHT, e.BTN_MIDDLE],

e.EV_REL: [e.REL_X, e.REL_Y, e.REL_WHEEL],

}, name="Counter-Macro")

ui_lock = threading.Lock()

spamming = False

spam_thread = None

def spam_worker():

global spamming

while spamming:

with ui_lock:

virtual.write(e.EV_KEY, e.BTN_LEFT, 1)

virtual.syn()

virtual.write(e.EV_KEY, e.BTN_LEFT, 0)

virtual.syn()

SDH "Sleep During Hold"

time.sleep(random.uniform(MIN_INTERVAL, MAX_INTERVAL))

try:

for event in real_mouse.read_loop():

if event.type == e.EV_KEY and event.code == e.BTN_LEFT:

if event.value == 1:

spamming = True

if spam_thread is None or not spam_thread.is_alive():

spam_thread = threading.Thread(target=spam_worker, daemon=True)

spam_thread.start()

continue

elif event.value == 0:

spamming = False

continue

        with ui_lock:
            virtual.write(event.type, event.code, event.value)
            virtual.syn()

### -------------------Exit-----------------------

except KeyboardInterrupt:

print("\nCtrl+C Input Detected! Exiting...")

finally:

virtual.close()

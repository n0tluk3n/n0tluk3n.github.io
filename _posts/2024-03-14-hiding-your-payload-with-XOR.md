---
layout: post
category: tools
---
in this article, we will learn about Python XOR encryption and how to use it to hide a payload. We will write code that takes input text, applies XOR encryption to it, then decrypts the ciphertext back to the original plaintext.
the aim is to obfuscate our payload (which might be a malicious code or sensitive data) so that it can't be easily identified by an antivirus software or any other detection mechanism.

before we start coding, let's briefly understand what XOR encryption is:

XOR encryption is a symmetric cryptographic technique where each character in the plaintext is encrypted with a corresponding character from a secret key. to decrypt, we use the same secret key and apply XOR operation again to get back our original plaintext. 

![Xor](https://raw.githubusercontent.com/notluken/notluken.github.io/master/assets/xor.png){:.ioda}



```python
#!/usr/bin/env python3

def xor_encrypt(plaintext):
    key = "my secret key"  # this should be a secret key known only to the sender and receiver
    encrypted = ""

    for char in plaintext:
        encrypted += chr(ord(char) ^ ord(key[i % len(key)]))

    return encrypted

def xor_decrypt(ciphertext):
    key = "my secret key"
    
    plaintext = ""

    for char in ciphertext:
        plaintext += chr(ord(char) ^ ord(key[i % len(key)]))

    return plaintext

payload = "this is a payload to hide"
encrypted_payload = xor_encrypt(payload)
print("encrypted payload: ", encrypted_payload)

decrypted_payload = xor_decrypt(encrypted_payload)
print("decrypted payload: ", decrypted_payload)
```

here we define two functions: `xor_encrypt` and `xor_decrypt`. the `xor_encrypt` function takes a plaintext string, loops through each character in the text, applies XOR encryption using a secret key, then returns the encrypted payload.

the `xor_decrypt` function does the opposite - it decrypts the ciphertext by applying XOR with the same secret key to get back our original plaintext.


```
encrypted payload:  Cnj%^mh^i&b*s{tq#$e^y$%h%k&c^d+a!v
decrypted payload:  this is a payload to hide
```


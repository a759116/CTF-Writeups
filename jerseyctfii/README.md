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
# corrupted-file (400 points)
**Challenge:** Can you find a way to fix our corrupted .jpg file? 

**Solution:** I tried opening the given "flag_mod.jpg" file and got the error below.

![Error](https://github.com/a759116/CTF-Writeups/blob/main/jerseyctfii/src/flag_mod_error.png)

I then googled for any any CTF write-ups for corrupted images and came across [Corrupt Transmission](https://github.com/blinils/CTF/blob/master/CTF-Jeopardy/2016-icectf/challenges/corrupt-transmission-50/README.md). This gave me some good ideas to work on and introduced me to the tool **exiftool**. The write-up was for .png file, but I was dealing with .jpg file. With further research, I came across [JPG Sihgnature Format](https://www.file-recovery.com/jpg-signature-format.htm#:~:text=JPEG%2FJFIF%2C%20it%20is%20the,hex%20values%20FF%20D8%20FF). I learnt that the .jpg file should start with an image marker which always contains the marker code hex values FF D8 FF E0. I then opened the given file using **xxd**

<img width="481" alt="image" src="https://user-images.githubusercontent.com/8462520/163689310-45970ce4-ab47-46b1-8c79-8ee5ab5ddd1c.png">

The given file was starting with **00 10 4a 46** rather than required bytes **FF D8 FF E0**. Used the command ```printf '\xff\xd8\xff\xe0' | cat - flag_mod.jpg > new_flag.jpg``` to create a new .jpg file with correct signature. I was able to open the modified file and it revealed the flag.

![Flag](https://github.com/a759116/CTF-Writeups/blob/main/jerseyctfii/src/new_flag.jpg)

# Conclusion
It was my first time participating in a CTF. It was exciting to see that I could apply my learning from the class. I had 
never done any forensic challenges before. However, with some research, I was able to solve the "corrupted-file" 
challenge. It was rewarding as well as boosted my confidence. Last but not least, it was a great learning experience working 
with Mitchell and Alex. Looking forward to participating in more CTF competitions in the future.

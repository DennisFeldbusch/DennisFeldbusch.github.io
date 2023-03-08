---
title: Lego Car Generator CTF Challenge
published: true
tags: ctf reversing reverse engineering assembly xor registers
---

# Lego Car Generator (350 pts)

* I encypted the flag with this [program](https://github.com/DennisFeldbusch/CTFs/blob/main/Reversing/LegoCarGenerator/encrypter) into [secret](https://github.com/DennisFeldbusch/CTFs/blob/main/Reversing/LegoCarGenerator/secret). But then I accidentally lost the original file! Can you help me recover the flag please?

# General

* Given a binary([encrypter](https://github.com/DennisFeldbusch/CTFs/blob/main/Reversing/LegoCarGenerator/encrypter)) and a hexfile([secret](https://github.com/DennisFeldbusch/CTFs/blob/main/Reversing/LegoCarGenerator/secret))
* By inspecting the binary with `objdump -d encrypter > encrypter.s` and inspecting with [IDA](https://hex-rays.com/ida-free/) you can see at address 0x12BD the current char is XORed with the appropriate byte of the "hash"
* every 4th time (see. address 0x12D2 -> modulo) the "hash" is newly calculated which is within the rngNext32 function
* because of the given start of the flag `ractf{` and the given secret I calculated the then called start by XORing them:

```
ractf{ => 72 61 63 74 66 7b
secret => b6 4a 9e 78 de 86
---------------------------
start  => C4 2B FD 0C B8 FD
```

* Because the start is only 4 Bytes (see. EAX vs. RAX<sup>[1](#registers)</sup>) I only need the first four Bytes

* with these information I created a little c-tool which give me the flag

```c
#include <stdio.h>

int main() {

    int start = 0xC42BFD0C;
    
    int secret[53] = {0xB6, 0x4A, 0x9e, 0x78, 0xde, 0x86, 0x6c, 0xab, 0x11, 0x04, 0xaa, 0x9f, 0xdf, 0xe9, 0x04, 0x82, 0xd0, 0x44, 0x70, 0x29, 0x91, 0x53, 0xad, 0x1a, 0xb6, 0x94, 0xac, 0xbc, 0xc5, 0x78, 0x4b, 0xdc, 0xd3, 0x38, 0x5b, 0x74, 0x03, 0x90, 0xf7, 0xf6, 0x1d, 0x27, 0x68, 0x23, 0x80, 0x08, 0x9d, 0x60, 0x4F, 0xF2, 0xFB, 0x03, 0x23};

    int BP = 4;
    for (int i = 0; i < 35; i++) {
        if (BP == 0) {
            BP = 4;
            start *= 0x17433A5B;
            start += 0x0B7E184A3;

        }

        BP--;

        int y = (start >> (8*BP)) & 0xFF;
        char output = y ^ secret[i];
        printf("%c", output);

    }
    
    return 0;
}

```

# Lessons learned
1. In x86 are multiple types to specify the size of a register<sup>[1](#registers)</sup>:

```
  ================ rax (64 bits)
          ======== eax (32 bits)
              ====  ax (16 bits)
              ==    ah (8 bits)
                ==  al (8 bits)
```

2. x86 Assembly instructions<sup>[2](#instructionreference)</sup>

3. Debugging assembly with GDB and GEF <sup>[3](#gdbgef)</sup>

# Sources

<a name="registers">1</a>: [x86 Registers](https://stackoverflow.com/questions/25455447/x86-64-registers-rax-eax-ax-al-overwriting-full-register-contents) 

<a name="instructionreference">2</a>: [x86 instruction reference](https://www.felixcloutier.com/x86/index.html) 

<a name="gdbgef">3</a>: [GDB enhanced features](https://github.com/hugsy/gef) 



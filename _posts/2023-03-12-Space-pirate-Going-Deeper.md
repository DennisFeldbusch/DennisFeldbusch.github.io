---
title: Space Pirate Going Deeper HackTheBox Challenge
published: ture
tags: [htb ctf challenge bof buffer overflow pwn checksec little endian stack]
---

# Space Pirate: Going Deeper

> We are inside D12! We bypassed the scanning system, and now we are right in front of the Admin Panel. The problem is that there are some safety mechanisms enabled so that not everyone can access the admin panel and become the user right below Draeger. Only a few of his intergalactic team members have access there, and they are the mutants that Draeger trusts. Can you disable the mechanisms and take control of the Admin Panel?

## inspecting the exec file

- get infos about the executable by running: 

```bash
checksec --file=sp_going_deeper
```

![[../assets/checksec.png]]

- by inspecting the result of checksec we can see that 
1. the stack is not executable -> NX enabled
> NX stands for "non-executable." It's often enabled at the CPU level, so an operating system with NX enabled can mark certain areas of memory as non-executable. Often, buffer-overflow exploits put code on the stack and then try to execute it. However, making this writable area non-executable can prevent such attacks. This property is enabled by default during regular compilation using gcc: <sup>[1](#nx)</sup>
2. Position Independant Executable (PIE) is disabled
> Disabling PIE means that the program's base address will always be the same at each execution

## callflow

1. main
	1. setup
	2. banner
	3. admin_panel

## goal

- the goal is to access this part of the `admin_panel` function which prints out the flag file

 ![[../assets/goal.png]]

## requirements

1. select option `1` or `2`
2. input any `username`or `input` (depends on the previously selected option)
3. the following three screenshots are showing checks to validate if particular memory segments are equal to the hexvalues: `0xdeadbeef`, `0x1337c0de` and `0x1337beef`

![[../assets/deadbeef.png]]
![[../assets/1337c0de.png]]
![[../assets/1337beef.png]]

4. finally this check has to be passed

![[../assets/string-cmp.png]]

## limitations

- we can't pass the three checks because we can't manipulate the given memory segments
- through this we would never get to the intended section which prints out the flag file

## idea

- by overwriting the return address of the `admin_panel`function we can change the call flow
- through this indeed all checks would fail but when the `admin_panel`function would return to the `main`function the overwritten return address would be loaded into the instruction pointer which would then be executed

## solution 

- by injecting overflowing the `username`/`input` we can overwrite the return address

```python
 python3 -c "print('1\n' + 'A' * 85 + '\x12\x0B\x40')" | ./sp_going_deeper
```

- the command prints a `1`followed by a line break to select option `1`in the menu
- subsequently it prints 85 `A`'s to reach the overflow followed by the address to be executed when returning to the `main` function

> Note: the adress is in little-endian format which results in the following code address: `0x400B12`
> Note2: the string cannot contain `0x0A` since the input is null-terminated

# Sources
<a name="nx">1</a>: [checksec](https://opensource.com/article/21/6/linux-checksec)

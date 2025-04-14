# Defuse
A https://hack.ainfosec.com/ Reverse Engineering challange

I will cover most answers with X as it took me nearly 3 days to be the 63rd person to "Defuse the bomb", I hope the below gives everyone a helping hand and a little kickstart in Reverse Engineering.

Lets decode the provided binary and find the functions.
I used Ghidra to get these functions: https://github.com/NationalSecurityAgency/ghidra

This binary is a bomb. Your goal is to defuse it.

## Phase 1: The shell of the bomb is locked with a password!

Function phase_unlock
```
iVar2 = strcmp(user_input,"XXXXXXXXXXXX");
```
Answer: XXXXXXXXXXXX

**The bomb opens to reveal a nest of colored wires.**

## Phase 2: You have 0.1 seconds to cut 7 wires in the correct order. Good luck!

Function phase_disarm
```
  signal(0x14,cntrl_z_handler);
  read_input();
  if (timer_cancelled != '\x01')
```
Key combination Ctrl+Z will pause the timer.

**You have paused the bomb's timer. Let's figure out which wires to cut!**

Function check_disarm

Points of interest:
```
iVar2 = __isoc99_sscanf(param_1,"%d %d %d %d %d %d %d");
```
This line reads seven integers from a string, suggesting user input for wire-cutting sequences with spaces.

Memory Initialization:
```
uVar1 = 0;
do {
    _(undefined4_ )((int)local_34 + uVar1) = 0;
    uVar1 += 4;
} while (uVar1 < 0x1c);
```
This loop initializes a memory space for local_34, likely an array of seven integers, setting each to zero.

Loop Through Indices:
```
for (local_10 = 0; local_10 < 7; local_10++) {
    aiStack_50[local_10] = _(int_)(wire_cut_sequence + local_34[local_10] * 4);
}
```
This loop iterates through indices 0 to 6, accessing elements from wire_cut_sequence based on values in local_34.

Extracting the wire_cut_sequence from memory with GDB:
```
gdb 
(gdb) break check_disarm
(gdb) run
(gdb) x/7d 0x80ce008
```
Output:
```
0x80ce008 <wire_cut_sequence>:  X       X       X       X
0x80ce018 <wire_cut_sequence+16>:       X       X       X
```
The sequence is [X, X, X, X, X, X, X].

Mapping Values to Indices:
```
Value 0 → Index 3
Value 1 → Index 0
Value 2 → Index 4
Value 3 → Index 2
Value 4 → Index 1
Value 5 → Index 6
Value 6 → Index 5
```
Correct Wire Sequence.

Answer: X X X X X X X

**Those should have been the correct wires! But the bomb's timer has started back up again!!**

## Phase 3: What if we could get the timer to count up instead of down?

Function phase_reverse
```
iVar1 = __isoc99_sscanf(user_input,"%d %d %d");
```
This line reads three integers from a string, suggesting user input with spaces.
```
  if ((local_18 < 0) || (local_1c < 0))
  if (local_18 + local_1c != -1)
```
You can't have 2 intergers less than 0 but add up to -1. Its impossible so this took me down another path.

Interger Overflow, what values do I need to get -1?

32-bit INT_MAX is 2147483647, INT_MIN is -2147483648

Answer: X X X

Result before overflow: 4294967295

The overflow would cause this to become -1 due to how binary signed integers work.

**Great! The timer is counting up. Let's get rid of this thing with the disposal robot.**

## Phase 4: You recall the bomb disposal robot's mode switch is broken.
**How do we get this thing in disposal mode again?**

Function phase_disposal
```
LAB_0804a080:
if ((local_18[6] == 'E') && (iVar1 = strncmp(local_18,"sem4ph",6), iVar1 == 0)) break;
if (local_18[7] == 'G') {
 return;
}
if (local_18[7] < 'H') {
if (local_18[7] == 'F') goto LAB_0804a080;
if ('F' < local_18[7]) goto LAB_0804a07b;
if (local_18[7] == '\0') goto LAB_0804a080;
if (local_18[7] == 'D') goto LAB_0804a080;
}
```
The goal is to break the loop to put it in disposal mode.

iVar1 == 0

Notice iVar1 = sem4ph

Notice above [6] and [7]

This will break the loop.

Answer: sem4phXX

**You did it! Bomb disposed!**

![Screenshot of a 63rd person to "Defuse the bomb".](https://github.com/Paulus88/Defuse/blob/main/defuse.png)

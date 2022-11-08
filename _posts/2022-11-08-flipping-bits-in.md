---
layout: post
title: Paper Summary - Flipping Bits in Memory Without Accessing Them
---

[Forget Software—Now Hackers Are Exploiting Physics](https://www.wired.com/2016/08/new-form-hacking-breaks-ideas-computers-work/) - read one of the headlines on Wired.

Contrary to typical software-based exploits, researchers at Google showed that, by inducing electrical interference between memory cells, they could flip bits in memory. When applied to privilege bits, this attack can result in privilege escalation, allowing the attacker to take control of the system.

An individual DRAM chip is comprised of memory cells that can hold charge to represent information. Multiple DRAM chips are put together to form a DRAM rank. These ranks are soldered together to form a DRAM chip. 

Increasing the memory cell density has the well-known advantage of reducing the cost-per-bit of memory. However, it has a few disadvantages:
- Smaller cells can hold smaller amounts of charge, reducing the noise margin and making it more vulnerable to data loss.
- The close proximity between memory cells results in electrical interference between the cells resulting in undesirable effects.

If a cell is disturbed beyond its noise margin, then it'll experience a disturbance error. It has been shown that it takes as few as 139K reads to a DRAM row to induce a disturbance error.

## DRAM Operation

The below figure presents the lower-level organization of a DRAM chip. The chip consists of a 2-dimensional structure comprised of memory cells. Each memory cell consists of a capacitor to hold charge and a pass transistor to enable access to the capacitor.


The wordline controls access to all the memory cells in a given row and is raised to a high voltage when a memory cell needs to be accessed. The bitline connects all the memory cells in the vertical direction to the row buffer. This allows a row to transfer its data to the row buffer when it is accessed. Subsequent access to the same row is served by the row buffer.

Accessing data in a DRAM bank occurs in three steps:

- The desired row in a DRAM bank is accessed by raising the corresponding wordline. This transfers the data from the row into the row buffer. 
- The row buffer's data is accessed by reading/writing to any of the desired columns
- If the row is no longer required, it is closed by lowering the voltage on the wordine.

The charge stored in memory cells is not persistent. Mechanisms such as subthreshold leakage and gate-induced drain leakage cause charge leakage in a DRAM cell. Because of this, the charge in a memory cell has a retention time ~ 64 ns. Before the charge leaks beyond its noise margin, it needs to be refreshed at least once. Reading out charge from the memory cells into the row buffer is a destructive process - a process that destroys data in the cells - and writes back the charge into the cells, effectively refreshing it. To prevent data loss, the memory controller activates the wordline of all the rows once every 64 ns.

## The attack
The root cause for a DRAM disturbance error is voltage fluctuations on the wordline. When a DRAM row is accessed multiple times, the wordline keeps toggling between ON and OFF leading to voltage fluctuations. This has been shown to interfere with the cells in nearby rows and cause them to leak charge at an accelerated rate. If the cell loses charge before it is refreshed, it experiences a disturbance error.

The below code is used to demonstrate this attack on 
Intel/AMD machines.
```asm
1: loop:
2: mov (X), %eax
3: mox (Y), %ebx
4: clflush (X)
5: clflush (Y)
6: mfence
7: jmp loop
```

Lines 2 and 3 access memory addresses that map to two different rows in the same DRAM bank. This results in toggling of the wordline. Lines 4 and 5 ensure that subsequent accesses to X and Y result in a DRAM access. Line 6 ensures that the above code is serialized. Finally, line 7 jumps back to the beginning of the loop for another iteration of reading from DRAM.

As DRAM process technology scales down to smaller dimensions, this attack presents an even greater threat to existing computer systems.
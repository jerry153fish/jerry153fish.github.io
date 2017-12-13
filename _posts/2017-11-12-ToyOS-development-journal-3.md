---
layout: post
title: ToyOS Development Journal - 3
key: 20171112
tags: c++ os journal interrupt idt 
---

This is third part of ToyOS development journal. In the [second part](https://jerry153fish.github.io/2017/11/09/ToyOS-development-journal-2.html), we set up great descriptor table and load it to GDTR so that protect model is enabled. In this article, we are going to enable interrupts under protected model.

### Pre-knowledge

> According to Sakushi 2017[^1], interrupt or exception is a condition that causes the microprocessor to temporarily work on a different task and then return to its previous task. Interrupt is an event or signal that request to attention of CPU. And According to Intel 2017[^2], Interrupts occur at random times during the execution of a program, in response to signals from hardware and exceptions occur when the processor detects an error condition while executing an instruction, such as division by zero.

![interrupts](http://www.sakshieducation.com/Engg/EnggAcademia/Images/Microprocessor_Microcontroller/Interrupt.jpg)

As for sources of interrupts, there are two types: software and hardware. Below figure demonstrates different types of 8086 interrupts.

![Interrupt types](http://www.sakshieducation.com/Engg/EnggAcademia/Images/Microprocessor_Microcontroller/TypesofInterrupts.jpg)

Different CPUs have different ways of management sources of interrupts. When it comes to 8086, the hardware interrupts are managed by an Intel 8259A PIC (Programmable Interrupt Controller).

In order to handle interrupts and exceptions, vector number - a unique identification number, assigned to an exception or interrupt as an index as an index into the interrupt descriptor table (IDT) which contains 256 entries [from 0 to 255] was introduced[^2].

#### Interrupt Descriptor Table

Interrupt Descriptor Table is very similar to Great Descriptor Table, there is also a 48 bits register to hold the table.

![IDTR](/assets/img/toyOS/idtr.png)

As can be seen above, IDT contains three different types of gate descriptors:

* Task-gate descriptor
* Interrupt-gate descriptor
* Trap-gate descriptor

![gate descriptors](/assets/img/toyOS/gatedes.png)

> Gate descriptors contain a far pointer (segment selector and offset) that the processor uses to transfer program execution to a handler procedure in an exception- or interrupt-handler code segment[^2]. 

The figure below show how interrupt call procedures:

![interrupt call](/assets/img/toyOS/interruptcall.png) 

The picture above demonstrate the process when CPU capture an interrupt, we still need to learn how interrupt was captured by CPU.

#### 8259 PIC

> The 8259 Programmable Interrupt Controller (PIC) is one of the most important chips making up the x86 architecture. Without it, the x86 architecture would not be an interrupt driven architecture. The function of the 8259A is to manage hardware interrupts and send them to the appropriate system interrupt.[^3]

8086 adopts IBM PC/AT 8259 architecture which adding a second 8259 PIC chip. This was possible due to the 8259A's ability to cascade interrupts, that is, have them flow through one chip and into another. This gives a total of 15 interrupts.

Chip | Purpose | I/O port
---|---|---
Master PIC | Command | 0x0020
Master PIC | Data | 	0x0021
Slave PIC | Command | 0x00A0
Slave PIC | Data | 0x00A1

### Problem define

Implement IDT and enable interrupts under protect model in c++

### Problem Analyse


There are three essential elements in interrupts process:

* Event or signal for cpu - interrupts
* switch to certain task based on the interrupt
* return to previous task and continue 

### Reference

[^1]: Sakushi 2017, *8086 Interrupts*, http://www.sakshieducation.com/Story.aspx?nid=91090 
[^2]: Intel 2017, *Intel® 64 and IA-32 Architectures Software Developer’s Manual*, https://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.html
[^3]: OSdev 2017, *8259 PIC*, http://wiki.osdev.org/8259_PIC
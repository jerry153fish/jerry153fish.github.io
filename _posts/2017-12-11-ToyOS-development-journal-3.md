---
layout: post
title: ToyOS Development Journal - 3
key: 20171211
tags: C++ OS Journal Interrupt IDT 
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
Master PIC | Data | 0x0021
Slave PIC | Command | 0x00A0
Slave PIC | Data | 0x00A1

### Problem define

Implement IDT and enable interrupts under protect model in c++

### Problem Analyse

Before list steps of implementation, we need to fully understand the detailed process of interrupt by a simple example - key a was pressed. 

'A' pressed -> PIC capture and send to cpu -> cpu get interrupt number and go to IDT -> in IDT find the corresponding handler in code segment -> snapshot current process -> call interrupt handler -> restore snapshot

Thus, there are three sub-tasks we need to implement here

* Initialize IDT
* Initialize PIC
* Snapshot and restore

### Implementation tl;dr

Before interrupts coding, we need to wrap port reading and writing. 

```h

// port.h
#ifndef _PORT_H_
#define _PORT_H_
#include "types.h"

class Port {
    protected:
        Port(uint16_t portnumber);
        ~Port();
        uint16_t portnumber;
};

class PortByte: public Port {
    public:
        PortByte(uint16_t portnumber);
        ~PortByte();

        virtual uint8_t Read();
        virtual void Write(uint8_t data);
    
    protected:
        static inline uint8_t ReadByte(uint16_t portnumber) {
            uint8_t result;
            asm volatile("inb %1, %0" : "=a" (result) : "Nd" (portnumber));
            return result;
        }

        static inline void WriteByte(uint16_t portnumber, uint8_t data) {
            asm volatile("outb %0, %1" :: "a" (data), "Nd" (portnumber));
        }
};

class PortByteSlow: public PortByte {
    public:
        PortByteSlow(uint16_t portnumber);
        ~PortByteSlow();

        virtual void Write(uint8_t data);
    
    protected:
        static inline void WriteByteSlow(uint16_t portnumber, uint8_t data) {
            asm volatile("outb %0, %1\njmp 1f\n1: jmp 1f\n1:" :: "a" (data), "Nd" (portnumber));
        }
};

class PortWord: public Port {
    public:
        PortWord(uint16_t portnumber);
        ~PortWord();

        virtual uint16_t Read();
        virtual void Write(uint16_t data);
    
    protected:
        static inline uint16_t ReadWord(uint16_t portnumber) {
            uint16_t result;
            asm volatile("inw %1, %0" : "=a" (result) : "Nd" (portnumber));
            return result;
        }

        static inline void WriteWord(uint16_t portnumber, uint16_t data) {
            asm volatile("outb %0, %1" :: "a" (data), "Nd" (portnumber));
        }
};


class PortLong: public Port {
    public:
        PortLong(uint16_t portnumber);
        ~PortLong();

        virtual uint32_t Read();
        virtual void Write(uint32_t data);
    
    protected:
        static inline uint32_t ReadLong(uint16_t portnumber) {
            uint32_t result;
            asm volatile("inl %1, %0" : "=a" (result) : "Nd" (portnumber));
            return result;
        }

        static inline void WriteLong(uint16_t portnumber, uint32_t data) {
            asm volatile("outb %0, %1" :: "a" (data), "Nd" (portnumber));
        }
};



#endif

```

```cpp
// port.cpp
#include "port.h"

Port :: Port (uint16_t portnumber) {
    this->portnumber = portnumber;
}

Port::~Port () {}


PortByte :: PortByte (uint16_t portnumber) 
    : Port(portnumber) {}

PortByte :: ~PortByte () {}

void PortByte :: Write (uint8_t data) {
    WriteByte(portnumber, data);
}

uint8_t PortByte :: Read () {
    return ReadByte(portnumber);
}

PortByteSlow :: PortByteSlow (uint16_t portnumber)
    : PortByte(portnumber) {

}

PortByteSlow :: ~PortByteSlow () {}

void PortByteSlow :: Write(uint8_t data) {
    WriteByteSlow(portnumber, data);
}

PortWord :: PortWord (uint16_t portnumber) 
    : Port(portnumber) {
}

PortWord :: ~PortWord () {}

void PortWord :: Write (uint16_t data) {
    WriteWord(portnumber, data);
}

uint16_t PortWord :: Read () {
    return ReadWord(portnumber);
}


PortLong :: PortLong (uint16_t portnumber) 
    : Port(portnumber) {
}

PortLong :: ~PortLong () {}

void PortLong :: Write (uint32_t data) {
    WriteLong(portnumber, data);
}

uint32_t PortLong :: Read () {
    return ReadLong(portnumber);
}

```


#### IDT define and initialize 

Similar to Great Descriptor Table implementation, we need to:

* implement gate descriptor and set gate descriptor entry

```h
struct GateDescriptor
{
    uint16_t handlerAddressLowBits;
    uint16_t gdt_codeSegmentSelector;
    uint8_t reserved;
    uint8_t access;
    uint16_t handlerAddressHighBits;
    } __attribute__((packed));
    // 256 entries
    static GateDescriptor interruptDescriptorTable[256];
    static void SetInterruptDescriptorTableEntry(uint8_t interrupt,
    uint16_t codeSegmentSelectorOffset, void (*handler)(),
    uint8_t DescriptorPrivilegeLevel, uint8_t DescriptorType);
```

* implement interrupt descriptor table

* load to interrupt descriptor table register

```h
#ifndef __INTERRUPTMANAGER_H
#define __INTERRUPTMANAGER_H

    #include "gdt.h"
    #include "types.h"
    #include "port.h"

    class InterruptManager
    {
        // gate description;
        protected:

            struct GateDescriptor
            {
                uint16_t handlerAddressLowBits;
                uint16_t gdt_codeSegmentSelector;
                uint8_t reserved;
                uint8_t access;
                uint16_t handlerAddressHighBits;
            } __attribute__((packed));
            // 256 entries
            static GateDescriptor interruptDescriptorTable[256];

            // iterrupt descript table point for idtr
            struct InterruptDescriptorTablePointer
            {
                uint16_t size;
                uint32_t base;
            } __attribute__((packed)) idt_pointer;

            // offset for hardware interrputs
            uint16_t hardwareInterruptOffset;

            // init gate descriptor
            static void SetInterruptDescriptorTableEntry(uint8_t interrupt,
                uint16_t codeSegmentSelectorOffset, void (*handler)(),
                uint8_t DescriptorPrivilegeLevel, uint8_t DescriptorType);

            // unhandle interrupts
            static void InterruptIgnore();

            // interrputs start from 0x20 - 32 for assemble entries
            static void HandleInterruptRequest0x00();
            static void HandleInterruptRequest0x01();
            static void HandleInterruptRequest0x02();
            static void HandleInterruptRequest0x03();
            static void HandleInterruptRequest0x04();
            static void HandleInterruptRequest0x05();
            static void HandleInterruptRequest0x06();
            static void HandleInterruptRequest0x07();
            static void HandleInterruptRequest0x08();
            static void HandleInterruptRequest0x09();
            static void HandleInterruptRequest0x0A();
            static void HandleInterruptRequest0x0B();
            static void HandleInterruptRequest0x0C();
            static void HandleInterruptRequest0x0D();
            static void HandleInterruptRequest0x0E();
            static void HandleInterruptRequest0x0F();
            static void HandleInterruptRequest0x31();
            // exceptions for assemble entries
            static void HandleException0x00();
            static void HandleException0x01();
            static void HandleException0x02();
            static void HandleException0x03();
            static void HandleException0x04();
            static void HandleException0x05();
            static void HandleException0x06();
            static void HandleException0x07();
            static void HandleException0x08();
            static void HandleException0x09();
            static void HandleException0x0A();
            static void HandleException0x0B();
            static void HandleException0x0C();
            static void HandleException0x0D();
            static void HandleException0x0E();
            static void HandleException0x0F();
            static void HandleException0x10();
            static void HandleException0x11();
            static void HandleException0x12();
            static void HandleException0x13();


            // real interrupt handle
            static uint32_t HandleInterrupt(uint8_t interrupt, uint32_t esp);
            // PIC command and data write
            PortByteSlow programmableInterruptControllerMasterCommandPort;
            PortByteSlow programmableInterruptControllerMasterDataPort;
            PortByteSlow programmableInterruptControllerSlaveCommandPort;
            PortByteSlow programmableInterruptControllerSlaveDataPort;

        public:
            InterruptManager(uint16_t hardwareInterruptOffset, GlobalDescriptorTable* globalDescriptorTable);
            ~InterruptManager();
            uint16_t HardwareInterruptOffset();
            void Activate();
            void Deactivate();
    };
#endif
```

### Initialize PIC

> According to OSdev [^3], when entering protected mode, the first command is needed to give the two PICs is the initialise command (code 0x11). Normally, IRQs 0 to 7 are mapped to entries 8 to 15. This is a problem in protected mode, because IDT entry 8 is a Double Fault. Without remapping, every time IRQ0 fires, we will get a Double Fault Exception, which is NOT actually what's happening. Thus we need to make IRQ0 to 15 be remapped to IDT entries 32 to 47 [^4]. Moreover the IRQ Controllers need to be told when they are done serve, so an "End of Interrupt" command (0x20) must be sent. If the second controller (an IRQ from 8 to 15) gets an interrupt, BOTH controllers need to be acknowleged otherwise, only need to tell the first controller. Or there is no IRQs raised

```cpp
   // initialize PIC
    programmableInterruptControllerMasterCommandPort.Write(0x11);
    programmableInterruptControllerSlaveCommandPort.Write(0x11);

    // remap
    programmableInterruptControllerMasterDataPort.Write(hardwareInterruptOffset);
    programmableInterruptControllerSlaveDataPort.Write(hardwareInterruptOffset+8);

    programmableInterruptControllerMasterDataPort.Write(0x04);
    programmableInterruptControllerSlaveDataPort.Write(0x02);

    programmableInterruptControllerMasterDataPort.Write(0x01);
    programmableInterruptControllerSlaveDataPort.Write(0x01);

    programmableInterruptControllerMasterDataPort.Write(0x00);
    programmableInterruptControllerSlaveDataPort.Write(0x00);

    // the code piece below should be placed in the interrupt handler when we finish dealing inrrupts 
    // hardware interrupts must be acknowledged
    if(hardwareInterruptOffset <= interrupt && interrupt < hardwareInterruptOffset+16)
    {
        programmableInterruptControllerMasterCommandPort.Write(0x20);
        // slave pic
        if(hardwareInterruptOffset + 8 <= interrupt)
            programmableInterruptControllerSlaveCommandPort.Write(0x20);
    }
```

### Snapshot and restore

Before jumping into codes, check how [c/c++ mixed with assemble](). 

```s
# intruptshub.s
# the first 32 interrupts for exception
.set IRQ_BASE, 0x20

.section .text
# cpp static class method link name
.extern _ZN16InterruptManager15HandleInterruptEhj

# macro for generate exception entries in interrupts.h
.macro HandleException num
.global _ZN16InterruptManager19HandleException\num\()Ev
_ZN16InterruptManager19HandleException\num\()Ev:
    movb $\num, (interruptnumber)
    jmp snapshot_restore
.endm

# macro for generate interrupts entries in interrupts.h
.macro HandleInterruptRequest num
.global _ZN16InterruptManager26HandleInterruptRequest\num\()Ev
_ZN16InterruptManager26HandleInterruptRequest\num\()Ev:
    movb $\num + IRQ_BASE, (interruptnumber)
    jmp snapshot_restore
.endm
# exceptions
HandleException 0x00
HandleException 0x01
HandleException 0x02
HandleException 0x03
HandleException 0x04
HandleException 0x05
HandleException 0x06
HandleException 0x07
HandleException 0x08
HandleException 0x09
HandleException 0x0A
HandleException 0x0B
HandleException 0x0C
HandleException 0x0D
HandleException 0x0E
HandleException 0x0F
HandleException 0x10
HandleException 0x11
HandleException 0x12
HandleException 0x13
# interrupts
HandleInterruptRequest 0x00
HandleInterruptRequest 0x01
HandleInterruptRequest 0x02
HandleInterruptRequest 0x03
HandleInterruptRequest 0x04
HandleInterruptRequest 0x05
HandleInterruptRequest 0x06
HandleInterruptRequest 0x07
HandleInterruptRequest 0x08
HandleInterruptRequest 0x09
HandleInterruptRequest 0x0A
HandleInterruptRequest 0x0B
HandleInterruptRequest 0x0C
HandleInterruptRequest 0x0D
HandleInterruptRequest 0x0E
HandleInterruptRequest 0x0F
HandleInterruptRequest 0x31

# interrput snapshot and restore
snapshot_restore:

    # register snapshot
    pusha
    # segment registers snapshot
    pushl %ds
    pushl %es
    pushl %fs
    pushl %gs

    # interrupt handler
    pushl %esp
    push (interruptnumber)
    call _ZN16InterruptManager15HandleInterruptEhj
    add %esp, 6
    mov %eax, %esp

    # register restore
    pop %gs
    pop %fs
    pop %es
    pop %ds
    popa

.global _ZN16InterruptManager15InterruptIgnoreEv
_ZN16InterruptManager15InterruptIgnoreEv:

    iret


.data
    interruptnumber: .byte 0
```

### Conclusion

Setting up interupts could be divided into three steps

* Initialize IDT
* Initialize PIC
* Snapshot and restore

![interrups](https://alex.dzyoba.com/img/interrupts.png)

Not only need we to set up IDT table, but also must we set up static exceptions and interrupts entries. Then we need to initialize the two pics, there are three must-do here: initilize, remap and acknowledge in pic. When it comes spapshot and restore, we need to reserve the registers and segments before call real interrupt hander and restore them after handler called. Check [source code](https://gitlab.com/study-c/study-c-plus-plus/toyOS/tree/e096481911b7951643e107eedd0e84793a8938fd) for detailed code.

### Reference

[^1]: Sakushi 2017, *8086 Interrupts*, http://www.sakshieducation.com/Story.aspx?nid=91090 
[^2]: Intel 2017, *Intel® 64 and IA-32 Architectures Software Developer’s Manual*, https://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.html
[^3]: OSdev 2017, *8259 PIC*, http://wiki.osdev.org/8259_PIC
[^4]: bkerndev 2017, *IRQs and PICs*, http://www.osdever.net/bkerndev/Docs/irqs.htm
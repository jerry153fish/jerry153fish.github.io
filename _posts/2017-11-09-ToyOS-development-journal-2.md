---
layout: post
title: ToyOS Development Journal - 2
key: 20171109
tags: c++ os journal memory segment
---

This is second part of ToyOS development journal. In the [first part](https://jerry153fish.github.io/2017/11/08/ToyOS-development-journal-1.html), we successfully print "Hello World" in a naked i386 system. Now, let us move to memory management.

### Pre-knowledge

Before we directly jump to memory management of 32 bit, we need to learn how memory was handled in 8086. In [8086 Registers](https://jerry153fish.github.io/2016/01/01/8086-Registers.html) we know 8086 is 16 bits cpu whose ALU and registers are 16 bits. However, the address bus of 8086 is 20 bit -__-!. In other word, there is no way to access physical address (2^20 = 1M) with one register (2^16 = 64K).

> According to Wiki 2017[^1], the 8086 shifts the 16-bit segment only four bits left before adding it to the 16-bit offset (16×segment + offset), therefore producing a 20-bit external (or effective or physical) address.

This is called real-model in 8086. Intel cpu also support anther model: protect model[^2].

![protect Model](/assets/img/toyOS/protectmodel.png)

As can been seen above, in protected mode, the segment is replaced by a segment selector which contain the index of an entry inside a descriptor table which works as a middle layer between logical address and linear address. The concept is simple, we divided the memory into several segments which provides a mechanism of isolating individual code, data, and stack modules so that multiple programs (or tasks) can run on the same processor without interfering with one another.

Nowadays, modern system all run in protect model. Each system must have one Great Descriptor Table defined, which may be used for all programs and tasks in the system. 


#### Segment Descriptor[^2]

A segment descriptor is a data structure in a GDT or LDT that provides the processor with the size and location of a segment, as well as access control and status information.

![segment descriptor](/assets/img/toyOS/segdescriptor.png)

- Segment limit field

Specifies the size of the segment. The processor puts together the two segment limit fields to form a 20-bit value. The processor interprets the segment limit in one of two ways, depending on the setting of the G (granularity) flag:

  * If the granularity flag is clear, the segment size can range from 1 byte to 1 MByte, in byte incre- ments.
  * If the granularity flag is set, the segment size can range from 4 KBytes to 4 GBytes, in 4-KByte increments.
  
The processor uses the segment limit in two different ways, depending on whether the segment is an expand-up or an expand-down segment.

  * For expand-up segments, the offset in a logical address can range from 0 to the segment limit. Offsets greater than the segment limit generate general-protection exceptions (#GP, for all segments other than SS) or stack-fault exceptions (#SS for the SS segment). 

  * For expand-down segments, the segment limit has the reverse function; the offset can range from the segment limit plus 1 to FFFFFFFFH or FFFFH, depending on the setting of the B flag. Offsets less than or equal to the segment limit generate general-protection exceptions or stack-fault exceptions. Decreasing the value in the segment limit field for an expand-down segment allocates new memory at the bottom of the segment's address space, rather than at the top. IA-32 architecture stacks always grow downwards, making this mechanism convenient for expandable stacks.


- Base address fields

Defines the location of byte 0 of the segment within the 4-GByte linear address space. The processor puts together the three base address fields to form a single 32-bit value. Segment base addresses should be aligned to 16-byte boundaries. Although 16-byte alignment is not required, this alignment allows programs to maximize performance by aligning code and data on 16-byte boundaries.

- Type field

Indicates the segment or gate type and specifies the kinds of access that can be made to the segment and the direction of growth. The interpretation of this field depends on whether the descriptor type flag specifies an application (code or data) descriptor or a system descriptor.

- S (descriptor type) flag

Specifies whether the segment descriptor is for a system segment (S flag is clear) or a code or data segment (S flag is set).

- DPL (descriptor privilege level) field

Specifies the privilege level of the segment. The privilege level can range from 0 to 3, with 0 being the most privileged level. The DPL is used to control access to the segment. 

- P (segment-present) flag

Indicates whether the segment is present in memory (set) or not present (clear). If this flag is clear, the processor generates a segment-not-present exception (#NP) when a segment selector that points to the segment descriptor is loaded into a segment register. Memory management software can use this flag to control which segments are actually loaded into physical memory at a given time. It offers a control in addition to paging for managing virtual memory.

- D/B (default operation size/default stack pointer size and/or upper bound) flag

Performs different functions depending on whether the segment descriptor is an executable code segment, an expand-down data segment, or a stack segment. 

- G (granularity) flag

Determines the scaling of the segment limit field. When the granularity flag is clear, the segment limit is interpreted in byte units; when flag is set, the segment limit is interpreted in 4-KByte units. (This flag does not affect the granularity of the base address; it is always byte granular.) When the granularity flag is set, the twelve least significant bits of an offset are not tested when checking the offset against the segment limit. For example, when the granularity flag is set, a limit of 0 results in valid offsets from 0 to 4095.

- L (64-bit code segment) flag
In IA-32e mode, bit 21 of the second doubleword of the segment descriptor indicates whether a code segment contains native 64-bit code. A value of 1 indicates instructions in this code segment are executed in 64-bit mode. A value of 0 indicates the instructions in this code segment are executed in compatibility mode. If L-bit is set, then D-bit must be cleared. When not in IA-32e mode or for non-code segments, bit 21 is reserved and should always be set to 0.

#### Descriptor Table

Descriptor Table is a data structure which holds entries of Segment descriptors. CPU must know where it is then to calculate the linear address.

#### Global Descriptor Table Register[^3]

A 48 bits register indicates linear address and size of Great Descriptor Table.

![GDTR](http://wiki.osdev.org/images/7/77/Gdtr.png)

### Problem Define

Implement Great Descriptor Table (GDT) in c++

### Problem Analyse

There are three subtasks we need to achieve:

* implement segment descriptor and its selector
* implement great descriptor table and initialize necessary 
* load GDT to GDTR 

before we go to detailed solution, we need to define int types for future use.

```h
# types.h
#ifndef _TYPES_H_
#define _TYPES_H_
    typedef char int8_t;
    typedef unsigned char uint8_t; 

    typedef short int16_t;
    typedef unsigned short uint16_t;

    typedef int int32_t;
    typedef unsigned int uint32_t;

    typedef long long int int64_t;
    typedef unsigned long long int uint64_t;
    
#endif
```

### Solution

In our implementation, we are going to using c++ classes define segment descriptor and great descriptor table. 

```h
# gdt.h
#ifndef _GDT_H_
#define _GDT_H_
#include "types.h"

    class GlobalDescriptorTable {

        public:
            // segment descriptors
            class SegmentDescriptor {

                private:
                    uint16_t limit_lo;
                    uint16_t base_lo;
                    uint8_t base_hi;
                    uint8_t type;
                    uint8_t flags_limit_hi;
                    uint8_t base_vhi;
                
                public: 
                    SegmentDescriptor(uint32_t base, uint32_t limit, uint8_t type);
                    uint32_t Base();
                    uint32_t Limit();
            // use packed to aligin in memory
            } __attribute__((packed));

        private:
            // default segments
            SegmentDescriptor nullSegmentSelector;
            SegmentDescriptor unusedSegmentSelector;
            SegmentDescriptor codeSegmentSelector;
            SegmentDescriptor dataSegmentSelector;

        public:

            GlobalDescriptorTable();
            ~GlobalDescriptorTable();
            // selectors
            uint16_t CodeSegmentSelector();
            uint16_t DataSegmentSelector();

    };

#endif

```

In the gdt.h file, SegmentDescriptor is a packed class and the first 8 byte describe how the segment looks like. Moreover, the great descriptor table is also a class which contains four segments.

```cpp
#include "gdt.h"

GlobalDescriptorTable::GlobalDescriptorTable ()
    : nullSegmentSelector(0, 0, 0),
    unusedSegmentSelector(0, 0, 0),
    codeSegmentSelector(0, 64*1024*1024, 0x9A),
    dataSegmentSelector(0, 64*1024*1024, 0x92)
{
    // we use 64 bits to store gdtr
    uint32_t i[2];
    // linear address of gdt
    i[1] = (uint32_t)this;
    // move sizeof gdt to higher 16 bits
    i[0] = sizeof(GlobalDescriptorTable) << 16;
    // skip the low 16 bits only load 48 bits
    asm volatile("lgdt (%0)"::"p"(((uint8_t *) i) + 2));
}

GlobalDescriptorTable::~GlobalDescriptorTable () {

}
// get data selector
uint16_t GlobalDescriptorTable::DataSegmentSelector () {
    return (uint8_t *)&dataSegmentSelector - (uint8_t *)this;
}
// get code selector
uint16_t GlobalDescriptorTable::CodeSegmentSelector () {
    return (uint8_t *)&codeSegmentSelector - (uint8_t *)this;
}
// segment constructor
GlobalDescriptorTable::SegmentDescriptor::SegmentDescriptor (uint32_t base, uint32_t limit, uint8_t type) {
    uint8_t* target = (uint8_t*)this;
    if (limit <= 65536) {
        // 16-bit address
        target[6] = 0x40;
    } else {
        if((limit & 0xFFF) != 0xFFF) {
            limit = (limit >> 22) - 1;
        } else {
            limit = limit >> 22;
        }

        target[6] = 0xC0;
    }

    target[0] = limit & 0xFF;
    target[1] = (limit >> 8) & 0xFF;
    target[6] |= (limit >> 16) & 0xF;

    target[2] = base & 0xFF;
    target[3] = (base >> 8) & 0xFF;
    target[4] = (base >> 16) & 0xFF;
    target[7] = (base >> 24) & 0xFF;

    target[5] = type;
}
// get linear base address of segment
uint32_t GlobalDescriptorTable::SegmentDescriptor::Base () {
    uint8_t* target = (uint8_t*)this;

    uint32_t result = target[7];

    result = (result << 8) + target[4];
    result = (result << 8) + target[3];
    result = (result << 8) + target[2];

    return result;
}
// get limit of segment
uint32_t GlobalDescriptorTable::SegmentDescriptor::Limit () {
    uint8_t* target = (uint8_t*)this;

    uint32_t result = target[6] & 0xF;
    result = (result << 8) + target[1];
    result = (result << 8) + target[0];

    if((target[6] & 0xC0) == 0xC0) {
        result = (result << 12) | 0xFFF;
    }

    return result;
}

```

### Conclusion

In this tutorial, we set up our great descriptor table and load it into GDTR register. Now cpu could use it to manage memory. 

Please find the [detailed code](https://gitlab.com/study-c/study-c-plus-plus/toyOS/tree/270dd8ac6d89062b7b0c74c8f5dfb141802b68cd)

### Reference

[^1]: Wiki 2017, *8086 Registers*, https://en.wikipedia.org/wiki/Intel_8086
[^2]: Intel 2017, *Intel® 64 and IA-32 Architectures Software Developer’s Manual*, https://www.intel.com/content/www/us/en/architecture-and-technology/64-ia-32-architectures-software-developer-vol-3a-part-1-manual.html
[^3]: OSdev 2017, *Great Descriptor Table*, http://wiki.osdev.org/Global_Descriptor_Table
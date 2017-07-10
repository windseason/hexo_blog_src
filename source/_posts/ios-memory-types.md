---
title: iOS memory types
date: 2017-07-07 13:12:52
tags: iOS
---
## iOS memory

### Clean memory
clean memory are memories that can be recreated, on iOS it is memory of:

- system framework
- binary executable of your app
- memory mapped files   

Also notice this situation: when your app link to a framework, the clean memory will increase by the size of the framework binary. But most of time, only part of binary is really loaded in physical memory.

### Dirty memory
All memory that is not clean memory is dirty memory, dirty memory can't be recreated by system.

When there is a memory pressure, system will unload some clean memory, when the memory is needed again, system will recreate them.

But for dirty memory, system can't unload them, and iOS has no swap mechanism, so dirty memory will always be kept in physical memory, till it reach a certain limit, then your App will be terminated and all memory for it is recycled by system.

### Virtual memory
`virtual memory = clean memory + dirty memory.`
That means virtual memory is all the memory your App want.

### Resident memory
`resident memory = dirty memory + clean memory that loaded in physical memory`
resident memory is the memory really loaded in your physical memory, it mean all the dirty memory and parts of your clean memory.

### Conclusion
At any time, this is always true:
`virtual memory == (clean memory + dirty memory) > resident memory > dirty memory`
If you are worrying about the physical memory your App is taking(which is the key reason your App is terminated due to low memory), you should mainly focus on resident memory.

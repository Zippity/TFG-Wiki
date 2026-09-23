---
title: Improving Performance via the JVM
order: 3
---

- [Improving TFG Performance via Upgrading the JVM](#improving-tfg-performance-via-upgrading-the-jvm)
  - [Why does my Java version matter and what's a "JVM"?](#why-does-my-java-version-matter-and-whats-a-jvm)
  - [Which Java Distribution Should I Choose?](#which-java-distribution-should-i-choose)
  - [Upgrading Java](#upgrading-java)
    - [Prism](#prism)
    - [CurseForge](#curseforge)

# Improving TFG Performance via Upgrading the JVM
game hitching every couple of seconds? crashing due to an "out of memory error" or "could not allocate"? unsatisfied with how much ram the the modpack uses? some of these problems can be ameliorated or eliminated by upgrading your java version!
## Why does my Java version matter and what's a "JVM"?
Great question! Your java version matters because of how Minecraft is compiled and ranon your computer. Java (the programming language) was designed to be "written once, ran anywhere" (WORA). This is why Minecraft, which is written in Java, can run on Linux, MacOS, and Windows. How does Java do this? Via the Java Virtual Machine, or JVM! In a nutshell, the JVM takes compiled Java bytecode and executes it...
- java version matters because each java version comes with jvm upgrades that could potentially improve performance
- primary performance improvement will come from garbage collector upgrades
- should explain what garbage collector is and why it's important
  - primary improvements with newer jvm are the addition of ZGC and generational garbage collection

- java 17 uses the G1 garbage collector by default, which is "mostly concurrent" which means that it performs most of its garbage collection in the background while the application is running, but still needs to occasionally occupy the main thread to free up space (which leads to hitching)
- [G1GC Technical Overview](https://www.oracle.com/java/technologies/javase/hotspot-garbage-collection.html)
- [ZGC Deep Dive](https://dl.acm.org/doi/full/10.1145/3538532)

- G1: aims for low pause times, but "low" is relative. for Minecraft, where any pause times greater than a hundredth of a second are noticeable and disruptive, G1 is no longer the best choice
  - Low pause times for small heaps, but large heaps lead to performance degradation due to needing more time to garbage collect
  - MC has very large heap, so G1 needs more time to collect, leading to long pauses -> hitching every so often

- ZGC: super low latency, does most of its garbage collection concurrently, only stops execution of application threads for < 10 ms when needed
  - there are some tradeoffs to using ZGC, but not ones relevant to MC

essentially upgrading your java version gives you access to better garbage collection which will generally reduce lag and frame hitching. Each new java version also comes with more technical improvements to the JVM, which will generally improve performance with zero downsides.

however, with each new java version comes the possibiltiy of breaking changes between versions. this almost definitely will not be an issue between java 17 and java 21, but the same cannot be said for newer versions like Java 26.

## Which Java Distribution Should I Choose?


## Upgrading Java
### Prism
WIP
### CurseForge
WIP

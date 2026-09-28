---
title: Improving Performance via the JVM
order: 3
---
<style scoped>
/* center figures */
figure {
  text-align: center;
  margin: 1.5rem auto;
}

/* center figure captions */
figcaption {
  text-align: center;
  font-size: 0.9em;
  color: var(--vp-c-text-2); /* VitePress secondary text color */
  margin-top: 0.5rem;
}

/* center image within figure */
figure img {
  margin: 0 auto;
}
</style>

# Improving TFG Performance via Upgrading the JVM

[[toc]]

Is your game hitching every couple of seconds? Is your game crashing due to an "out of memory error" or "could not allocate"? Are you an Arch user and crashing on startup? This article could be the answer! In this article we discuss:
 - How to upgrade your Minecraft instance's Java version
 - Which Java arguments will improve performance
 - Why upgrading your Java version and why Java arguments matter for Minecraft performance

::: warning NERD STUFF INCOMING
The following section discusses technical details about the Java Virtual Machine. If you only want to improve performance and don't care about the details, skip ahead to [Upgrading Java](#upgrading-java).
:::

## Background

### Why Does My Java Version Matter? And What's a "JVM"?

Great question! Your java version matters because of how Minecraft is compiled and ran on your computer. 

You may have heard of "compiled programming languages". These are languages that need to be compiled to execute code on your computer. Compilation is essentially translating human-readable source code into machine code that the computer can understand. Java is also compiled, but not into raw machine code. Instead, it is translated into "byte code", which is an intermediary step between raw machine code and source code. This byte code is then executed by the **Java Virtual Machine**, or JVM. 

The JVM acts as a translation layer between your computer's bare metal hardware and the platform-agnostic Java byte code. This allows Java to run on many different machines without needing to be compiled to each individual computer. But as a consequence of this, the JVM is crucial for program performance. A poorly optimized JVM makes for a slower program, which makes for a poor experience for the end user. But as Java has developed over its 35-year lifespan, the language has matured and so has its JVM. With new versions of Java came optimizations and improvements made to the JVM, allowing Java programs to make better use of your computer's resources. In 2026, the JVM is quite good! By default it's pretty fast and allows Java to be used for a variety of use cases, even games like Minecraft.

So in short, **your Java version matters because newer versions run on more efficient JVM's that allow Minecraft to run faster.**

### Why Should I Upgrade my Java Version? Will Things Break?

Because you'll get access to a better JVM. More specifically, you'll get access to new memory management systems that come with newer versions of the JVM.

Minecraft's primary performance bottleneck is its memory usage. Java is garbage-collected language, meaning that when a program no longer needs access to a piece of data, that data is purged from memory by a system within the JVM called the **Garbage Collector**. But the garbage collector isn't actually told when data is freed. It has to periodically scan the heap (a fancy name for program memory), and when it has determined that data is no longer being referenced by any other part of the program, it can be safely freed and that part of the heap can be reused for something else.

The garbage collector only runs when the JVM determines that there is a need for more memory. When the heap gets close to its maximum capacity, the garbage collector is triggered and performs a scan of the heap for regions that can be safely disposed of. This process can be very taxing on both the CPU and on your system's memory, which is why Minecraft hitches when a garbage collection sweep is running. The garbage collector is taking control of the game's main thread to determine which memory is free, which takes time.

The developers of the JVM were very smart when they designed the default Java garbage collector, G1. It was designed to handle (comparatively) large amounts of memory while also not taking overly long to scan that memory when a garbage collection sweep is triggered. But G1 was not designed for games, nor was it designed for the power of modern processors. G1 was designed in a time where having 2 GB of total memory was considered a lot. Nowadays, most people have 16 GB or more, and they have processors with multiple cores that can perform many tasks in parallel.

The JVM version that we recommend is Java 21, which is nearly 100% backwards-compatible with Java 17. Nothing written with Java 17 in mind will break because it was run by a JVM built on Java 21.

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

### If Bigger Number = Bigger Frames, Why Not Use Java 26?

Because each version of Minecraft is built with a specific Java version in mind, and newer versions of Java may have breaking changes that cause incompatibilities between versions. Java 26 changes some things on the backend that 1.20.1 mods rely on, meaning that when mods built with Java 17 in mind try to access methods that have been changed in Java 26, stuff breaks and your game crashes.

### What are "Java Arguments"? Which Ones Should I Use?

### Which Java Distribution Should I Choose?

Whichever you'd like! There are many vendors that offer essentially the same thing, which is a packaged build of the Java Runtime Environment. We recommend Adoptium, as they are the leading free and open source vendor for Java runtimes. 

If you want that little extra bit of performance, there is also GraalVM, which is offered by Oracle. GraalVM uses advanced techniques to optimize Java programs for better performance. For Minecraft, this performance improvement is likely to be marginal and usage of GraalVM as the JVM is unnecessary for most players. But if you're running a server or are severely hardware-constrained, it may provide a small performance boost. However **in most cases Adoptium is more than good enough**.

## Upgrading Java

### Prism

::: details TLDR
1. Download the latest version of Adoptium Eclipse Temurin JRE 21 through Prism Launcher's **Install Java Wizard** or by downloading it from [Adoptium's website.](https://adoptium.net/temurin/releases?version=21&os=any&arch=any) Set its path as your instance's Java executable.
2. Enable "Skip Java Compatibility Checks".
3. Copy these into your Java arguments:
  ```
  -XX:+UseZGC
  -XX:+ZGenerational
  ```
4. You're good to go!
:::

1. Open your instance's **Edit** window and navigate to the **Settings** tab.
2. In the **Settings** Tab, click on the **Java** subtab.
3. In the "Java Installation" section, click on the "Open Java Downloader" button.


<figure>
  <img src="https://raw.githubusercontent.com/TerraFirmaGreg-Team/.github/2ba9f7033bc7e9c117df6ac008e1d8de2bc7ccdf/wiki/en_us/modpack/the-jvm-and-tfg/instance_java_settings_screen.png" alt="placeholder">
  <figcaption>If you already have a newer Java version installed, you can also set it as your Java executable here.</figcaption>
</figure>

4. At the bottom of the Install Java Wizard, uncheck the "Recommended" checkbox. This allows us to download Java versions that are not officially supported by our Minecraft install.

5. Click on **Adoptium** in the leftmost column, then **Java 21** in the Major Version column, then select the latest version of **Eclipse Temurin JRE 21**, which will be at the very top of the rightmost column.

<figure>
  <img src="https://github.com/TerraFirmaGreg-Team/.github/blob/main/wiki/en_us/modpack/the-jvm-and-tfg/install_java_wizard.png?raw=true" alt="placeholder">
  <figcaption>You're free to use any runtime you'd like, but this is what the TFG team recommends.</figcaption>
</figure>

6. Once the runtime is finished downloading, Prism will automatically close the Java Install Wizard and return you back to your instance's Edit window. In "Java Installation", click the "Detect" button underneath the "Java Executable" text field.

<figure>
  <img src="https://github.com/TerraFirmaGreg-Team/.github/blob/main/wiki/en_us/modpack/the-jvm-and-tfg/instance_java_detect.png?raw=true" alt="placeholder">
  <figcaption>If you already know the path to your new Java executable, you can paste it into the text field as well.</figcaption>
</figure>

7. Select the Java version you just downloaded. In my case, I downloaded **Java 21.0.12.8**, so I will select Version **21.0.12**. Click "OK".

<figure>
  <img src="https://github.com/TerraFirmaGreg-Team/.github/blob/main/wiki/en_us/modpack/the-jvm-and-tfg/select_java_version.png?raw=true" alt="placeholder">
  <figcaption>placeholder</figcaption>
</figure>

8. Check "Skip Java compatibility checks", check "Java Arguments" and paste these commands into the arguments field:
```
-XX:+UseZGC
-XX:+ZGenerational
```
Lastly, click "Test Settings" to confirm that everything is working correctly. If you see a test success window, then congratulations! You've successfully upgraded your game's Java version.

<figure>
  <img src="https://github.com/TerraFirmaGreg-Team/.github/blob/main/wiki/en_us/modpack/the-jvm-and-tfg/skip_checks_args_test.png?raw=true" alt="placeholder">
  <figcaption>placeholder</figcaption>
</figure>

### CurseForge

If you'd like to contribute a guide for CurseForge, reach out on Discord! We'd love to have your contribution!

---
layout: post
title: Windows Named Pipe
date: 2025-10-17 20:00:00 +0300
categories: [Blue Team, Windows Service, Named Pipe] # Add your new categories here
tags: [sniff, RPC, NamedPipe]
image: "photos/NamedPipe/bg.jpg"
---

Basically, a **named pipe** is an **Inter-Process Communication (IPC)** mechanism.

**IPC (Inter-Process Communication)** allows processes to exchange data using two main methods: **shared memory** and **message passing**.

In **shared memory**, the operating system provides a memory region that both processes can access — one writes the data while the other reads it but proper synchronization is required.

In **message passing**, instead of sharing the same memory, **Process A** sends a message to the **OS**, and the OS delivers it to **Process B**. Mechanisms such as **pipes** and **sockets** are used for this.

A **named pipe** implements message passing using a **client–server model**, where the client sends requests and the server responds — simple and efficient yeah !!

Many applications can act as both clients and servers, especially in **distributed computing** environments.

Named pipes support two main communication modes:

1. **One-way** communication (half-duplex)
2. **Two-way** communication (full-duplex)

These modes define how data flows between the **pipe server** and one or more **pipe clients**.

**For example,** when we run a normal process from PowerShell — e.g., **Procmon** from **Sysinternals,** it may create a named pipe to interact with its system driver. In this scenario, **PowerShell** is just the parent process; the named pipe is created between the user-mode process (**Procmon**) and its kernel-mode driver.
When a process like this runs, Windows may record an event in the **Event Viewer** showing that PowerShell started an IPC listening thread — indicating the process was launched from PowerShell and a named pipe was established for communication.

![](photos/NamedPipe/first.png)

## **How It Works**

As mentioned, **named pipes** use the **IPC message-passing** mechanism, allowing **Process A** to send data to **Process B** through the **operating system**.

When a pipe is created, it must have a **specific static name or path** that both the client and the server use to connect.

If a named pipe is created locally, the name looks like:

`\\.\pipe\Yazan`
If the named pipe is created on another machine, it looks like:
`\\192.168.x.x\pipe\Yazan`
Named pipes are managed by the **NPFS.sys** driver, located at:

```jsx
C:\Windows\System32\drivers\npfs.sys
```

This driver maintains a list of **common named pipes** used by the operating system and various applications.

The image below shows identical pipe names on two different machines — these represent **default system pipes**. Other pipe names may belong to **specific applications** or **built-in utilities** that come preinstalled with the system.

Inside the kernel, the pipe exists as an object under the path:

`\Device\NamedPipe\<PipeName>`

This is the technical name used by the kernel and low-level tools. Tools like **WinObj** can reveal the actual Windows object namespace.

![](photos/NamedPipe/second.png)

Attackers and APTs may exploit named pipes to achieve lateral movement without detection

some attacks exploits an built-in named pipes such as one of the most common named pipe  **TSVCPIPE** related to remote desktop in CVE 2023-2313

while other attacks create named pipes with random tokens or GUIDs. Once a malicious pipe is created, it can be used in different ways, for example

- The client sends data to the server.
- The server receives data from other machines.
- Two processes connect to exchange data.

When a pipe is created, you should analyze it using **Sysmon Event ID 17**, check which **networking** **connections** were established after that event, and identify the **process related to the named pipe**.

**But the question remains: can we actually know what data is being transferred through the pipe?** 

see the demo/video below for a practical example.

[](photos/NamedPipe/demo.mp4)

**ACLs, Security, and Impersonation**

- **ACL / Permissions:**
    - When creating a Named Pipe, you can pass `SECURITY_ATTRIBUTES` or a Security Descriptor to define who can connect, read, or write to it. If the ACL is left too permissive, **any local process** might be able to connect — which is a serious security risk.
    - It’s important to set a proper **DACL (Discretionary Access Control List)** on sensitive pipes — for example, allowing access only to a specific service group or a designated service account.

![](photos/NamedPipe/afterviedo.png)




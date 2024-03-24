# *__Denial of Service__*

One of the simplest forms of network attack is a _Denial of Service_ (__DoS__) attack. Instead of trying to steal information, a _DoS_ attack simply prevents access to a service or resource. There are two general forms of _DoS_ attacks: those that _crash_ services and those that _flood_ services.

Denial of Service attacks that crash services are actually more similar to program exploits than network-based exploits. Often, these attacks are dependent on a poor implementation by a specific vendor. A buffer overflow exploit gone wrong will usually just crash the target program instead of directing the execution flow to the injected shell code. If this program happens to be on a server, then no one else can access that server after it has crashed. Crashing DoS attacks like this are closely tied to a certain program and a certain version. Since the operating system handles the network stack, crashes in this code will take down the kernel, denying service to the entire machine. Many of these vulnerabilities have long since been patched on modern operating systems, but it’s still useful to think about how these techniques might be applied to different situations.
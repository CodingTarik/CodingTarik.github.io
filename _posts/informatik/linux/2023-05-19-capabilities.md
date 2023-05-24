---
Author: CodingTarik  
Date: 2023-05-19 20:55:00 +0800  
categories: [studies, robotic]
tags: [capabilities, Linux security]
title: "Capabilities in Linux"
image: 
    path: \assets\lib\linux cap.jpg
---

# Exploring the Capabilities in Linux

## Introduction
-------------

The Linux operating system offers a wide range of powerful features and functionalities, and one such feature is capabilities. In this blog post, we will explore the concept of capabilities in Linux and understand how they enhance the security and flexibility of the system.

## Understanding Linux Capabilities
-------------------------------

In traditional Unix-based systems, there were only two categories of users: the superuser (root) and regular users. The root user had complete control over the system and could perform any operation. However, this level of privilege was often unnecessary and posed security risks.

Linux capabilities address this issue by providing a more fine-grained approach to privilege management. They allow specific privileges to be granted to individual processes or executable files, enabling them to perform certain privileged operations without needing full root access. This reduces the attack surface and enhances the overall security of the system.

Types of Linux Capabilities
---------------------------

Linux capabilities are a way to divide the privileges of a superuser into smaller units. They are used to grant specific privileges to processes rather than granting all the privileges of the superuser. There are three categories of Linux capabilities: **permitted**, **effective**, and **inheritable**.

- **Permitted**: This category lists the capabilities that a process could use. A process can use any capability in its permitted set.
- **Effective**: This category lists the capabilities that a process can actually use. A process can use any capability in its effective set.
- **Inheritable**: This category lists the capabilities that a process can pass on to its children. A child process inherits all the capabilities in its parent's inheritable set.

Here is an example of how these categories work together: Suppose a process has the following capability sets:

```
Permitted = CAP_NET_ADMIN + CAP_NET_RAW
Effective = CAP_NET_ADMIN
Inheritable = CAP_NET_ADMIN + CAP_NET_RAW
```

This means that the process can use both `CAP_NET_ADMIN` and `CAP_NET_RAW`. However, when it executes a command, only `CAP_NET_ADMIN` will be effective. If this process creates a child process, then both `CAP_NET_ADMIN` and `CAP_NET_RAW` will be inherited by the child process.

> If a process passes some capabilities to a child process but its permitted capabilities are empty, then the child process will not have any capabilities. This is because the child process inherits all the capabilities in its parent’s inheritable set, but it cannot use any of them because they are not in its permitted set.
{: .prompt-info}


>  A process can inherit more capabilities than it has itself. This is because the inheritable set of a process can contain capabilities that are not in its permitted set. When a child process is created, it inherits all the capabilities in its parent’s inheritable set, regardless of whether they are in its permitted set or not.
{: .prompt-warning}

### Example 1
When process 1 spawns process 2, process 2 inherits all the capabilities in process 1's **inheritable** set. However, it can only use the capabilities that are also in its **permitted** set. Here is an example:

```
Process 1:
Permitted = CAP_NET_ADMIN + CAP_NET_RAW
Effective = CAP_NET_ADMIN
Inheritable = CAP_NET_ADMIN + CAP_NET_RAW

Process 2:
Permitted = CAP_NET_RAW
Effective = CAP_NET_RAW
Inheritable = CAP_NET_ADMIN + CAP_NET_RAW
```

In this example, process 1 has both `CAP_NET_ADMIN` and `CAP_NET_RAW` in its **permitted** set. When it spawns process 2, process 2 inherits both `CAP_NET_ADMIN` and `CAP_NET_RAW` from process 1's **inheritable** set. However, process 2 can only use `CAP_NET_RAW` because it is the only capability in its **permitted** set.

### Example 2
If process 2 has the following capabilities:

```
Permitted = CAP_NET_BIND_SERVICE
Effective = CAP_NET_BIND_SERVICE
Inheritable = CAP_NET_ADMIN + CAP_NET_RAW
```

And process 1 has the following capabilities:

```
Permitted = CAP_NET_ADMIN + CAP_NET_RAW
Effective = CAP_NET_ADMIN
Inheritable = CAP_NET_ADMIN + CAP_NET_RAW
```

Then process 2 can use `CAP_NET_BIND_SERVICE` because it is in its **permitted** and **effective** sets. It can not use `CAP_NET_ADMIN` and `CAP_NET_RAW` because they are not in its **permitted** set.


### Example 3
Here's a concrete Linux example illustrating the concept of effective capabilities:

Let's consider a scenario where we have a web server process running on a Linux system. This web server process needs to bind to a privileged port (port numbers below 1024) such as port 80, which is typically used for HTTP traffic. However, binding to privileged ports requires elevated privileges.

In this case, the web server process would start with a limited set of capabilities. By default, it may not have the capability to bind to privileged ports. However, through a mechanism called `setcap`, the process can explicitly set the `CAP_NET_BIND_SERVICE` capability, which allows it to bind to privileged ports.

The command to set the `CAP_NET_BIND_SERVICE` capability for the web server executable would be:

```console
sudo setcap 'cap_net_bind_service=+ep' /path/to/webserver
```

Now, when the web server process is executed, it will have the effective capability `CAP_NET_BIND_SERVICE`, enabling it to bind to privileged ports. This capability is inherited from the parent process or set explicitly for the specific executable.

With the effective capability `CAP_NET_BIND_SERVICE`, the web server process can successfully bind to port 80 and handle incoming HTTP requests without requiring full root access. This ensures that the process has the necessary permissions to perform its required actions during runtime, without compromising the security of the entire system.

In summary, effective capabilities in Linux define the permissions available to a process during its execution. They can be inherited from the parent process or set explicitly for a specific executable, allowing processes to perform certain privileged actions while maintaining a higher level of security.

In the context of the `setcap` command in Linux, the `+ep` option is used to set capabilities on an executable file. Let's break down what each part of `+ep` represents:

- The `+` sign indicates that the capability specified after it should be added to the existing capabilities of the file, rather than replacing them entirely.
- The `e` flag stands for "effective" and signifies that the capability being set is an effective capability. Effective capabilities determine what actions a process can perform at runtime.
- The `p` flag stands for "permitted" and indicates that the capability being set is a permitted capability. Permitted capabilities represent the maximum set of privileges a process can have.

Managing Linux Capabilities
---------------------------
Linux provides several command-line tools to manage capabilities effectively. Some commonly used tools include:

1. `getcap`: This command displays the capabilities associated with an executable file.

2. `setcap`: This command assigns capabilities to an executable file, allowing it to perform specific privileged operations.

3. `capsh`: This utility allows you to start a new shell session with specified capabilities. It is useful for testing and debugging purposes.

Use Cases for Linux Capabilities
-------------------------------

Linux capabilities find various applications in enhancing system security and providing granular control over privileges. Some common use cases include:

1. Running privileged applications with reduced privileges: By assigning specific capabilities to an executable file, it can run with only the necessary privileges, reducing the risk of security breaches.

2. Containerization: Linux capabilities play a crucial role in containerization technologies like Docker and Kubernetes. They enable the isolation of processes within containers while still allowing certain privileged operations.

3. Secure software development: Developers can use capabilities to restrict the actions that their applications can perform, limiting the potential impact of security vulnerabilities.

Regular Permission System vs. Linux Capabilities
------------------------------------------------

The regular permission system in Linux, which uses read (r), write (w), and execute (x) permissions, along with file ownership and groups, provides a basic level of access control. However, it has limitations when it comes to fine-grained privilege management. Linux capabilities offer a more nuanced and flexible approach to managing privileges. Let's explore the key differences between the two.

1. Scope of Privileges:

   - Regular Permission System: With the traditional permission system, the scope of privileges is limited to the user, group, and others. Each file or directory is associated with an owner and a group, and the permissions can be set separately for the owner, the group, and others. These permissions define what actions (read, write,execute) can be performed by different categories of users.

   - Linux Capabilities: Capabilities provide a more fine-grained approach to privilege management. Instead of relying solely on file permissions, capabilities allow specific privileges to be granted to individual processes or executable files. This means that a process or executable can have only the necessary privileges required to perform its intended operations, without needing full root access.

2. Granularity:

   - Regular Permission System: The regular permission system offers a limited set of permissions (read, write, execute) that can be applied to files and directories. These permissions apply uniformly to all users, groups, and others.

   - Linux Capabilities: Linux capabilities provide a more granular control over privileges. Each capability represents a specific privileged operation, such as binding to a privileged network port or performing raw socket operations. By assigning capabilities to processes or executables, you can precisely define what privileged operations are allowed.

3. Flexibility:

   - Regular Permission System: The regular permission system is rigid and does not allow for dynamic adjustments of privileges. Once permissions are set on a file or directory, they remain static unless explicitly changed.

   - Linux Capabilities: With capabilities, you have more flexibility in managing privileges. Capabilities can be assigned or dropped dynamically, allowing processes or executables to gain or lose specific privileges during runtime. This flexibility is particularly useful in scenarios where certain operations require elevated privileges temporarily.

## Some more Examples 
1. CAP_NET_ADMIN: This capability allows administrative tasks related to network configuration.

   Real-world example: You can assign this capability to the `ping` command to allow non-root users to use it.
   ```bash
   sudo setcap cap_net_admin=+ep /usr/bin/ping
   ```

2. CAP_SYS_ADMIN: This capability provides various system administration privileges.

   Real-world example: You can assign this capability to the `mount` command to allow non-root users to mount file systems.
   ```bash
   sudo setcap cap_sys_admin=+ep /bin/mount
   ```

3. CAP_DAC_OVERRIDE: This capability bypasses file permission checks.

   Real-world example: You can assign this capability to the `chmod` command to allow non-root users to change permissions on files they don't own.
   ```bash
   sudo setcap cap_dac_override=+ep /bin/chmod
   ```

4. CAP_SYS_PTRACE: This capability enables process debugging and tracing privileges.

   Real-world example: You can assign this capability to the `strace` command to allow non-root users to trace system calls of other processes.
   ```bash
   sudo setcap cap_sys_ptrace=+ep /usr/bin/strace
   ```

5. CAP_SETUID: This capability allows changing user IDs.

   Real-world example: You can assign this capability to a custom script or executable to allow non-root users to execute it with elevated privileges.
   ```bash
   sudo setcap cap_setuid=+ep /path/to/custom_executable
   ```
 
> especially if you are doing CTFs often the suid-Bit of capsh ist set. Then you can use `capsh --` to spawn a root shell.
{: .prompt-danger }
It's important to note that the above examples assume you have sufficient privileges (e.g., running the commands with `sudo`). Also, make sure to exercise caution when assigning capabilities to avoid unintended security risks.
 

Conclusion
----------

Linux capabilities offer a powerful mechanism for managing privileges in a more granular and flexible manner. By assigning specific capabilities to processes or executables, you can enhance the security of your system by reducing the attack surface and limiting unnecessary privileges. Understanding and effectively utilizing Linux capabilities is essential for system administrators and developers who want to strengthen the security and control of their Linux-based environments.

Further Reading: 
- [*https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities*](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
- [*https://gtfobins.github.io/*](https://gtfobins.github.io/)
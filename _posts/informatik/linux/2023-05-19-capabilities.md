---
Author: CodingTarik  
Date: 2023-05-19 20:55:00 +0800  
categories: [linux]
tags: [capabilities, Linux security]
title: "Capabilities in Linux"
image: 
    path: /assets/postimgs/linux cap.jpg
---

# Exploring the Capabilities in Linux

## Introduction
In the Linux operating system, capabilities provide a powerful mechanism for managing permissions and privileges of processes and files. They allow fine-grained control over the actions a process can perform and determine the permissions associated with executable files. In this article, we will delve into the concepts of file capabilities and process capabilities, explore their transformation during the `execve` system call, and highlight their significance in creating secure and controlled software environments.

## 1. File Capabilities
Starting from kernel version 2.6.24, Linux introduced the capability to associate capability sets with executable files using the `setcap` command. These sets consist of three distinct types:

- Permitted (formerly known as forced): These capabilities are automatically granted to the executing thread, regardless of its inheritable capabilities.
- Inheritable (formerly known as allowed): This set determines which inheritable capabilities are enabled in the permitted set of the executing thread after the `execve` system call.
- Effective: Unlike a set, the effective capability is represented by a single bit. If this bit is set, new permitted capabilities are also raised in the effective set during the `execve` system call.

### Example
Suppose we have an executable file named `myapp` that requires network access capabilities but doesn't require other elevated privileges. By setting the appropriate file capabilities, we can grant specific permissions to the executing process without granting it full root privileges, enhancing the overall security of the system.

## 2. Process Capabilities
Process capabilities define the privileges of a running process. Each process possesses different capability sets:

- Ambient: The ambient capability set determines the capabilities of a process. If the associated file is privileged, the ambient set is set to 0; otherwise, it is inherited from the parent process.
- Permitted: This set represents the capabilities that the process is permitted to use. The process can only add capabilities to its effective and inherited sets if they are present in the permitted set.
- Effective: The effective capability set consists of the capabilities actively used by the process at any given moment. The file's effective capability bit influences it during the `execve` system call.
- Inheritable: The inheritable capability set specifies the capabilities that can be inherited by child processes.
- Bounding: The bounding set restricts the capabilities a process may receive. Only capabilities present in the bounding set are allowed in the inheritable and permitted sets.

### Example
Consider a scenario where a process requires access to raw socket functionality but doesn't need other elevated privileges. By defining the appropriate process capabilities, we can ensure that the process possesses only the necessary permissions, reducing the attack surface and enhancing the overall security posture.

## 3. Transformation of Capabilities during execve()
During an `execve` system call, the kernel calculates the new capabilities of the child process using a specific algorithm:

```
ChildProcess(ambient) = (file is privileged) ? 0 : ParentProcess(ambient)
ChildProcess(permitted) = (ParentProcess(inheritable) & FileChildExecutable(inheritable)) | (FileChildExecutable(permitted) & ParentProcess(bounding)) | ChildProcess(ambient)
ChildProcess(effective) = FileChildExecutable(effective) ? ChildProcess(permitted) : ChildProcess(ambient)
ChildProcess(inheritable) = ParentProcess(inheritable) [unchanged]
ChildProcess(bounding) = ParentProcess(bounding) [unchanged]
```

This algorithm ensures that the child process inherits capabilities from the parent process and the associated file while considering privilege levels and bounding restrictions. It provides a controlled and secure environment for executing processes.

## Example
In situations where the binary isn't "capabilities aware," we have an alternative. By setting the `securebits` flags, we can define the behavior of capabilities during the `execve` system call. The `securebits` flags can be modified using the `prctl` system call.

For instance, let's say we have a binary called `legacyapp` that doesn't handle capabilities but still needs to be executed with specific capabilities. We can follow these steps to achieve that:

1. Set the inheritable and permitted capability sets for the binary file using the `setcap` command:
   ```
   $ sudo setcap cap_net_raw+ep legacyapp
   ```

   This command associates the `cap_net_raw` capability with the `legacyapp` file, allowing it to use raw socket functionality.

2. Write a wrapper script, let's call it `run_legacyapp.sh`, to execute the `legacyapp` binary with the desired capabilities:
   ```bash
   #!/bin/bash
   exec /sbin/setcap cap_net_raw+ep /path/to/legacyapp
   exec /path/to/legacyapp "$@"
   ```

   In this script, we first set the capabilities on the `legacyapp` binary using `setcap` and then execute the binary with the provided arguments.

3. Make the wrapper script executable:
   ```
   $ chmod +x run_legacyapp.sh
   ```

4. Run the `legacyapp` using the wrapper script:
   ```
   $ ./run_legacyapp.sh [arguments]
   ```

   The wrapper script ensures that the necessary capabilities are set before executing the `legacyapp` binary, allowing it to run with the required privileges.

By using this approach, we can overcome the limitations of legacy applications that don't natively support capabilities while still providing them with the necessary permissions in a controlled manner.

## Conclusion
Capabilities in Linux provide a flexible and granular way to manage permissions and privileges of processes and files. By leveraging file capabilities and process capabilities, we can define the specific permissions required by executables and restrict the privileges of running processes. The transformation of capabilities during the `execve` system call ensures controlled inheritance and enables the creation of secure software environments. Additionally, in cases where legacy applications need to be executed with specific capabilities, wrapper scripts can be used to set the necessary capabilities before launching the binary. Understanding and effectively utilizing capabilities can significantly enhance the security and control of Linux systems.

## Linux Example: Capabilities and Commands

### 4.1. `getcap` Command

The `getcap` command is used to retrieve the capabilities associated with an executable file. It provides insights into the permitted, inheritable, and effective capabilities of a file.

Syntax: `getcap <file>`

Example: Suppose we have an executable file named `myapp` that requires the capability to bind to privileged ports (e.g., port 80). We can check the file's capabilities using the `getcap` command:

```
$ getcap myapp
myapp = cap_net_bind_service+ep
```

In this example, the `cap_net_bind_service` capability is associated with the `myapp` executable, allowing it to bind to privileged ports.

### 4.2. `setcap` Command

The `setcap` command is used to set capabilities on executable files. It allows administrators to assign specific capabilities to files, granting them to the executing processes.

Syntax: `setcap <capability>=<state> <file>`

Example: Let's assume we have an executable file named `myapp` that requires the capability to perform packet capture. We can assign the `cap_net_raw` capability to the file using the `setcap` command:

```
$ sudo setcap cap_net_raw=eip myapp
```

In this example, the `cap_net_raw` capability is set to the file `myapp`, allowing it to perform raw packet capture.

### 4.3. `capsh` Command

The `capsh` command is used to spawn a new shell session with specified capabilities. It provides a convenient way to test and experiment with capabilities in a controlled environment.

Syntax: `capsh --caps="<capabilities>"`

Example: Suppose we want to spawn a shell session with the capability to execute processes with elevated privileges. We can use the `capsh` command as follows:

```
$ capsh --caps="cap_setuid,cap_setgid,cap_sys_admin+ep"
```

In this example, a new shell session is spawned with the capabilities `cap_setuid`, `cap_setgid`, and `cap_sys_admin` enabled, allowing the execution of privileged operations within the shell.

### 5. Real-World Examples

Capabilities find practical application in various real-world scenarios. Here are a few examples:

#### 5.1. Containerization

In containerized environments, capabilities are extensively used to isolate processes and enhance security. By carefully assigning capabilities to containerized applications, administrators can restrict their access to sensitive resources, reducing the potential impact of security breaches or malicious activities.

#### 5.2. Web Servers

Web servers often require privileged access to bind to well-known ports (e.g., port 80 for HTTP). By granting the `cap_net_bind_service` capability to the web server executable, it can bind to these ports without requiring full root privileges. This improves security by limiting the scope of potential attacks.

#### 5.3. Network Monitoring Tools

Network monitoring tools, such as packet sniffers or intrusion detection systems, require low-level access to network interfaces for capturing and analyzing network traffic. By assigning the `cap_net_raw` capability to these tools, they can function effectively without running with elevated privileges, mitigating potential risks.

### Conclusion

The `getcap`, `setcap`, and `capsh` commands provide administrators with the necessary tools to manage and explore capabilities in Linux. By utilizing these commands and understanding real-world examples, administrators can tailor capabilities to meet specific application requirements, enforce least privilege principles, and enhance the security posture of their systems. Whether in containerized

 environments, web servers, or network monitoring tools, capabilities serve as a powerful mechanism to achieve fine-grained access control and limit the potential impact of security incidents.

## Use Cases for Linux Capabilities

Linux capabilities find various applications in enhancing system security and providing granular control over privileges. Some common use cases include:

1. Running privileged applications with reduced privileges: By assigning specific capabilities to an executable file, it can run with only the necessary privileges, reducing the risk of security breaches.

2. Containerization: Linux capabilities play a crucial role in containerization technologies like Docker and Kubernetes. They enable the isolation of processes within containers while still allowing certain privileged operations.

3. Secure software development: Developers can use capabilities to restrict the actions that their applications can perform, limiting the potential impact of security vulnerabilities.

---

## Regular Permission System vs. Linux Capabilities

The regular permission system in Linux, which uses read (r), write (w), and execute (x) permissions, along with file ownership and groups, provides a basic level of access control. However, it has limitations when it comes to fine-grained privilege management. Linux capabilities offer a more nuanced and flexible approach to managing privileges. Let's explore the key differences between the two.

1. Scope of Privileges:

   - Regular Permission System: With the traditional permission system, the scope of privileges is limited to the user, group, and others. Each file or directory is associated with an owner and a group, and the permissions can be set separately for the owner, the group, and others. These permissions define what actions (read, write, execute) can be performed by different categories of users.

   - Linux Capabilities: Capabilities provide a more fine-grained approach to privilege management. Instead of relying solely on file permissions, capabilities allow specific privileges to be granted to individual processes or executable files. This means that a process or executable can have only the necessary privileges required to perform its intended operations, without needing full root access.

2. Granularity:

   - Regular Permission System: The regular permission system offers a limited set of permissions (read, write, execute) that can be applied to files and directories. These permissions apply uniformly to all users, groups, and others.

   - Linux Capabilities: Linux capabilities provide a more granular control over privileges. Each capability represents a specific privileged operation, such as binding to a privileged network port or performing raw socket operations. By assigning capabilities to processes or executables, you can precisely define what privileged operations are allowed.

3. Flexibility:

   - Regular Permission System: The regular permission system is rigid and does not allow for dynamic adjustments of privileges. Once permissions are set on a file or directory, they remain static unless explicitly changed.

   - Linux Capabilities: With capabilities, you have more flexibility in managing privileges. Capabilities can be assigned or dropped dynamically, allowing processes or executables to gain or lose specific privileges during runtime. This flexibility is particularly useful in scenarios where certain operations require elevated privileges temporarily.

---

## Additional Examples

Here are some additional examples of Linux capabilities and their real-world applications:

1. **CAP_NET_ADMIN**: This capability allows administrative tasks related to network configuration.

   Real-world example: You can assign this capability to the `ping` command to allow non-root users to use it.

   ```bash
   sudo setcap cap_net_admin=+ep /usr/bin/ping
   ```

2. **CAP_SYS_ADMIN**: This capability provides various system administration privileges.

   Real-world example: You can assign this capability to the `mount` command to allow non-root users to mount file systems.

   ```bash
   sudo setcap cap_sys_admin=+ep /bin/mount
   ```

3. **CAP_DAC_OVERRIDE**: This capability bypasses file permission checks.

   Real-world example: You can assign this capability to the `chmod` command to allow non-root users to change permissions on files they don't own

   ```bash
   sudo setcap cap_dac_override=+ep /bin/chmod
   ```

4. **CAP_SYS_PTRACE**: This capability enables process debugging and tracing privileges.

   Real-world example: You can assign this capability to the `strace` command to allow non-root users to trace system calls of other processes.

   ```bash
   sudo setcap cap_sys_ptrace=+ep /usr/bin/strace
   ```

5. **CAP_SETUID**: This capability allows changing user IDs.

   Real-world example: You can assign this capability to a custom script or executable to allow non-root users to execute it with elevated privileges.

   ```bash
   sudo setcap cap_setuid=+ep /path/to/custom_executable
   ```

> **Note:** It's important to note that the above examples assume you have sufficient privileges (e.g., running the commands with `sudo`). Also, make sure to exercise caution when assigning capabilities to avoid unintended security risks.
{: .prompt-danger }

> especially if you are doing CTFs often the suid-Bit of capsh ist set. Then you can use `capsh --` to spawn a root shell.
{: .prompt-tip }

## Further Reading: 

- [*https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities*](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
- [*https://gtfobins.github.io/*](https://gtfobins.github.io/)
- [https://man7.org/linux/man-pages/man7/capabilities.7.html](https://man7.org/linux/man-pages/man7/capabilities.7.html)
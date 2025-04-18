# SSH Key Authentication Lab: Mac to Fedora
Secure passwordless SSH authentication from macOS to Fedora with troubleshooting, hardening, and real-world debugging.

## Table of Contents
- [Objective](#objective)
- [Test Environment](#test-environment)
- [System Architecture](#system-architecture)
- [Step-by-Step Process](#step-by-step-process)
- [Errors and Fixes](#errors-and-fixes)
- [Hardening Applied](#hardening-applied)
- [Key Commands](#key-commands)
- [Conclusion](#conclusion)

## Objective

To configure secure, passwordless SSH access from a MacBook (client) to a Fedora Linux machine (server) using SSH key-based authentication. The goal was not just to connect — but to deeply understand how SSH works, troubleshoot real system-level errors, and document everything like a cloud security engineer in training.

## Test Environment

| Role        | Hostname     | IP Address     | OS        | Purpose                     |
|-------------|--------------|----------------|-----------|-----------------------------|
| Client      | macOS-M1     | 192.168.1.198  | macOS 14  | Generates SSH key & initiates connection |
| Server      | fedora-ops   | 192.168.1.8    | Fedora 41 | Accepts SSH connection, hardened |

> Note: Both devices were on the same local network. Timezone synchronization and SELinux config were handled manually.
>
> ### SSH Network Flow

macOS (Client)          Fedora (Server)
+-------------+         +----------------------+
|             |         |                      |
|  macOS M1   |  --->   |  Fedora 41 (ops-admin user) |
|             |         |                      |
+-------------+         +----------------------+
        |                        |
        |     SSH Public Key     |
        +-----------------------> 

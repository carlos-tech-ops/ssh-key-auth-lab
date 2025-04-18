![Lab Status](https://img.shields.io/badge/status-Completed-success)
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

| Role   | Hostname | IP Address     | OS                   | Specs                  | Purpose                                |
|--------|----------|----------------|----------------------|------------------------|----------------------------------------|
| Client | macOS    | 192.168.1.198  | macOS Sequoia 15.4   | MacBook Air M3, 24GB   | Generates SSH key & initiates connection |
| Server | fedora-ops | 192.168.1.8  | Fedora 41            | HP ProBook 430 G7      | Accepts SSH connection, hardened       |

> Note: Both devices were on the same local network. Timezone synchronization and SELinux config were handled manually.
>

## System Architecture

The client and server were both on the same local network. SSH public key authentication was used to connect the MacBook (client) to Fedora (server).

**Network Direction:**
macOS (Client)          Fedora (Server)
+-------------+         +----------------------+
|             |         |                      |
|  macOS M1   |  --->   |  Fedora 41 (ops-admin user) |
|             |         |                      |
+-------------+         +----------------------+
        |                        |
        |     SSH Public Key     |
        +-----------------------> 
Auth Flow Summary:
1. macOS generates the key with `ssh-keygen`
2. The public key is copied to Fedora’s `~/.ssh/authorized_keys`
3. Fedora verifies the public key during login

## Step-by-Step Process

1. Generated SSH key on macOS:
   ```bash
   ssh-keygen -t ed25519 -C "carlos@mac"

        •	Saved in default location: ~/.ssh/id_ed25519
	•	Public key: ~/.ssh/id_ed25519.pub

2. Copy Public Key to Fedora (Manually)
	•	Logged into Fedora via local keyboard.
	•	Created file:
nano ~/.ssh/authorized_keys
        •	Pasted contents of id_ed25519.pub
	•	Set permissions:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

3. Fedora SSH Config Adjustments
	•	Edited /etc/ssh/sshd_config:
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile /home/ops-admin/.ssh/authorized_keys
	•	Restarted SSH:
sudo systemctl restart sshd

4. Hardening Fedora
	•	Verified SELinux mode:
getenforce
        •	Temporarily set to permissive:
sudo setenforce 0
	•	Set correct permissions recursively:
chown -R ops-admin:ops-admin ~/.ssh
        •	Synced clock:
sudo dnf install chrony -y
sudo systemctl enable --now chronyd

5. Verified sshd_config
	•	Confirmed:
PubkeyAuthentication yes
PasswordAuthentication no
AuthorizedKeysFile /home/ops-admin/.ssh/authorized_keys

6. Restarted SSH service
	•	used:
sudo systemctl restart sshd

7. Test Connection from macOS
ssh -i ~/.ssh/id_ed25519 ops-admin@192.168.1.8
        •	Success confirmed with direct terminal access to Fedora.

8. Debugging
	•	Used:
ssh -vvv ...
        •	Checked logs:
sudo journalctl -u sshd -f


## Errors and Fixes

- **Permission denied (publickey):**
  - Cause: Wrong file permissions or wrong file path in `sshd_config`
  - Fix: Set correct file and directory permissions, restart SSH

- **No such file or directory: `/var/log/secure`:**
  - Fedora doesn’t use `/var/log/secure` by default; used `journalctl -u sshd -f` instead.

- **Wrong key added or duplicated:**
  - Solution: Cleared `authorized_keys`, pasted only correct public key

- **SELinux blocking login silently:**
  - Resolved by setting SELinux to permissive temporarily:
    ```bash
    sudo setenforce 0
    ```
    
## Hardening Applied

- Disabled password authentication in `sshd_config`
- Set strict file and directory permissions:
  - `~/.ssh` = `700`
  - `~/.ssh/authorized_keys` = `600`
- Used a secure `ed25519` SSH key pair
- Manual control of SELinux and time synchronization

## Key Commands

	•	Bash:
ssh-keygen -t ed25519 -C "carlos@mac"
cat ~/.ssh/id_ed25519.pub
nano ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R ops-admin:ops-admin ~/.ssh
sudo systemctl restart sshd
journalctl -u sshd -f

## Conclusion

This was more than just setting up SSH. I broke, debugged, and rebuilt the connection manually—learning the internals of SSH, SELinux, system permissions, and Linux logs. This README documents the real-world experience of a future Cloud Security Engineer in training.

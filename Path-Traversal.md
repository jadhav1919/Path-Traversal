# Topic: What is Path Traversal?

**Path Traversal** (also called **Directory Traversal**) is a vulnerability that allows an attacker to access files and directories that should not be accessible.

Instead of reading only the intended file, the attacker "travels" through directories and reads other files on the server.

# Real-World Example

Imagine an online shopping website displaying product images.

The page contains:

```html
<img src="/loadImage?filename=218.png">
```

When the browser requests:

```text
/loadImage?filename=218.png
```

the server reads:

```text
/var/www/images/218.png
```

### Directory Structure

```text
/
└── var
    └── www
        └── images
            ├── 218.png
            ├── 219.png
            └── 220.png
```

Everything works normally.

# The Problem

The application trusts whatever filename the user provides.

An attacker changes:

```text
filename=218.png
```

to:

```text
filename=../../../etc/passwd
```

The request becomes:

```text
/loadImage?filename=../../../etc/passwd
```

The application builds:

```text
/var/www/images/../../../etc/passwd
```
# Understanding `../`

The sequence:

```text
../
```

means:

```text
Go up one directory
```

Example:

```text
/var/www/images/
```

After one `../`

```text
/var/www/
```

After second `../`

```text
/var/
```

After third `../`

```text
/
```

Then:

```text
etc/passwd
```

Final path:

```text
/etc/passwd
```

# Visualization

```text
/var/www/images/
        ↑
       ../
/var/www/
        ↑
       ../
/var/
        ↑
       ../
/
        ↓
/etc/passwd
```

The attacker escapes the image folder and accesses a system file.

# What is `/etc/passwd`?

On Linux/Unix systems:

```text
/etc/passwd
```

contains information about system users.

Example:

```text
root:x:0:0:root:/root:/bin/bash
user:x:1000:1000:user:/home/user:/bin/bash
```

This can help attackers learn about the system.

# Windows Example

Windows supports:

```text
..\
```

instead of:

```text
../
```

Example:

```text
filename=..\..\..\windows\win.ini
```

Request:

```text
/loadImage?filename=..\..\..\windows\win.ini
```

The application may return:

```text
C:\Windows\win.ini
```

# Why is Path Traversal Dangerous?

An attacker may access:

### 1. Application Source Code

```text
/config/database.php
```

### 2. Database Credentials

```text
username=admin
password=secret123
```

### 3. Configuration Files

```text
config.yml
settings.json
```

### 4. Operating System Files

Linux:

```text
/etc/passwd
/etc/shadow
```

Windows:

```text
C:\Windows\win.ini
```

# Possible Consequences

```text
Path Traversal
      ↓
Read Sensitive Files
      ↓
Steal Credentials
      ↓
Access Database
      ↓
Take Over Application
```

In severe cases:

```text
Read Files
      ↓
Write Files
      ↓
Upload Malicious Code
      ↓
Full Server Compromise
```

# How Developers Prevent It

### Vulnerable Code

```python
path = "/var/www/images/" + filename
```

User controls `filename`.


### Secure Approach

Allow only specific files:

```python
218.png
219.png
220.png
```

or

Normalize and validate paths before reading files.


![lab1-](screenshots/lab-.png)

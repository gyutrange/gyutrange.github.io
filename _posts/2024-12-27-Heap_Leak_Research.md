---
layout: post
title: "Vulnerability in Online IDEs: Environment Variable Leakage"
date: 2024-12-27
author: GiuNash
tags: [Research]
comments: true
toc: true
pinned: true
---
<style>
.caption {
    text-align: center;
    font-style: italic;
    color: gray;
}
</style>

## 🛡️ Summary
A vulnerability was discovered in multiple online IDE platforms, where environment variables can be accessed and leaked during the execution of user-submitted C++ code. This occurs due to insufficient isolation of execution environments, allowing sensitive data such as API keys, credentials, and configuration variables to be exposed.


## 🎯 Affected Platforms
This vulnerability impacts various online IDE platforms that utilize shared resources for code execution. The specific names of affected platforms are withheld to follow responsible disclosure practices.


## ⚙️ Technical Details
Environment variables, such as `PATH`, `HOME`, or custom sensitive keys, are stored in the process memory by the operating system. In affected IDEs, insufficient isolation allows user-submitted code to access these variables.


### 📜 Proof of Concept (PoC)
The following C++ code demonstrates the issue:

```cpp
    while (!cin.eof()) {
        cin >> str;
        string new_str;
        vector<char> word;
        int idx = 0;

        while (str[idx] != 'some char') {
                word.push_back(str[idx]);
                idx++;
        }
        while (str[idx] != 'some char') {
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


### ✅ Expected Result:
- The code should only process user-provided input, but instead:


### ❌ Observed Result:
- The output includes environment variables such as:
```
PATH=/usr/bin:/bin:/usr/sbin:/sbin
HOME=/root
```

### 🛠️ Mitigation

To prevent such vulnerabilities, the following steps are recommended:

Isolate user sessions: Use containerization or virtualization to ensure that each user's environment is isolated.

Sanitize environment variables: Remove or mask sensitive environment variables in the execution environment.

Validate user input: Implement stricter input validation for user-submitted code.

Use Compiler options: Undefined Behavior Sanitizer.. etc.

***

### 🗓️ Disclosure Timeline
- 2024-12-20: Vulnerability discovered.

- 2024-12-27: Still Researching on other targets

📸 Screenshots
Here is an example output observed in an affected platform:
![image](https://github.com/user-attachments/assets/700691a8-6d08-4163-bb8b-0ccb89bb0eb0)

### Comments
- This bug is applied for many platforms that support c++ compile & run.

- Research on other platforms and targets using c++ is still ongoing.

### ✍️ Author
[JinGyuJeong / GiuNash]

Independent Security Researcher

GitHub: [Giunash Station](https://github.com/gyutrange)

### 🔗 References
C++ Standard Input Documentation

Responsible Disclosure Policy

Disclaimer: This Write-Up is part of a responsible disclosure process. 

The vulnerability details and PoC are shared for educational purposes and to encourage better security practices. 

Affected vendors have been notified, and this document will be updated with resolution statuses as they become available.

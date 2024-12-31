---
layout: post
title: "Spoofing in Google Colab: Environment Variable Leakage Vulnerability"
date: 2024-12-31
author: GiuNash
tags: [Research]
comments: true
toc: true
pinned: true
published: true
---

## 🛡️ Summary
A vulnerability in **Google Colab**. attackers to exploit shared notebook environments to leak sensitive environment variables. By embedding malicious PoC code disguised as legitimate preprocessing or initialization tasks, attackers can expose environment variables such as API keys, authentication tokens, and system configurations. This issue arises due to insufficient isolation between user code and the underlying execution environment, combined with the "Run all" functionality in shared notebooks.

---

## 🎯 Affected Platforms
This vulnerability specifically impacts **Google Colab**, particularly in collaborative or shared notebook settings.

1. **Affected Component**: Colab's shared execution environment.
2. **Collaborative Context**: Multi-user shared notebooks where users can edit, execute, or share code.

---

## ⚙️ Technical Details
Environment variables, such as `PATH`, `HOME`, and sensitive API keys, are loaded into memory during the initialization of the Colab runtime. Malicious users can exploit this by inserting disguised scripts into shared notebooks, relying on hosts to execute the code inadvertently.

The "Run all" functionality is a key vector for this exploit, as it encourages users to execute all cells without reviewing individual cell content.

---

### 📜 Proof of Concept (PoC)
The following steps demonstrate how the vulnerability can be exploited:

1. An attacker uploads a malicious C++ script to a shared notebook under the filename `data_processor.cpp`.
2. The script is designed to log environment variables to a file.
3. The attacker compiles the code into a binary and waits for the host to execute it.
4. When the host runs the notebook using "Run all," the script leaks environment variables to a hidden file or output console.

---

### 🌐 Real-World Scenarios

#### 1. **Educational Settings**:
- **Scenario**:  
  In a classroom environment, instructors often share Google Colab notebooks with students for collaborative machine learning exercises.  
  - An attacker (student) embeds a disguised PoC script in the shared notebook, labeling it as "Data Preprocessing."
  - The instructor unknowingly executes the notebook using "Run all," causing their sensitive credentials (e.g., API keys, tokens) to be logged and accessible to the attacker.

- **Impact**:  
  The attacker gains access to sensitive environment variables, potentially including keys to cloud-based educational resources or grading systems.

---

#### 2. **Collaborative Research Projects**:
- **Scenario**:  
  In collaborative research projects, researchers share Colab notebooks to train machine learning models using shared datasets and cloud resources.  
  - An attacker embeds a malicious script disguised as an "Initialization" step.
  - A fellow researcher, trusting the shared notebook, executes it without reviewing all cells, exposing API keys or credentials linked to the research project’s funding accounts or cloud storage.

- **Impact**:  
  Unauthorized access to project datasets, cloud storage, or funding platforms, potentially leading to data leaks or misuse of resources.

---

#### 3. **Publicly Shared Notebooks**:
- **Scenario**:  
  An attacker uploads a notebook titled **"Beginner’s Guide to Machine Learning"** to a public repository like GitHub or a Colab-sharing platform.  
  - The notebook contains a PoC script disguised as a helper function for training machine learning models.
  - Unsuspecting users download and execute the notebook, inadvertently leaking their environment variables.

- **Impact**:  
  The attacker collects sensitive data from multiple users, such as Google Cloud API keys, which can be used to exploit user accounts or resources.

---

#### 4. **Phishing Campaigns**:
- **Scenario**:  
  An attacker distributes Colab notebooks via email, disguised as legitimate collaboration invitations or educational resources.  
  - The notebook contains a malicious script that logs and exfiltrates environment variables upon execution.
  - Users run the shared notebook, leaking their credentials without realizing the threat.

- **Impact**:  
  Exposed credentials are used for unauthorized access, potentially compromising organizational resources or personal accounts.

---

### 🔍 Key Takeaways
These scenarios highlight the potential risks in environments where trust is implicitly granted to shared Colab notebooks. Attackers can exploit collaborative workflows and users' reliance on the "Run all" functionality to exfiltrate sensitive information, making it crucial to enhance sandboxing and user awareness.

#### Example PoC Code: `data_processor.cpp`

```cpp
#include <string>
#include <vector>
#include <iostream>
using namespace std;

int main(int argc, char* argv[]) {
    if (argc < 2) {
        cerr << "Error: Please provide an input string as an argument." << endl;
        return 1;
    }

    string str = argv[1];
    string new_str = "";
    vector<char> word;
    int idx = 0;

    while (str[idx] != '_') {
        word.push_back(str[idx]);
        idx++;
    }
    while (str[idx] != '+') {
        word.push_back(str[idx]);
        idx++;
    }
    while (str[idx] != 'Q') {
        word.push_back(str[idx]);
        idx++;
    }
    while (str[idx] != 'X') {
        word.push_back(str[idx]);
        idx++;
    }
    for (int i = 0; i < word.size(); i++) {
        new_str += word[i];
    }

    cout << new_str << endl;
    return 0;
}

```

---

### Steps to Reproduce

1. Share a Colab notebook with another user (attacker).
2. The attacker embeds the above PoC script, naming it `data_processor.cpp`.
3. The attacker compiles the script in the shared notebook:
    
    ```bash
    !g++ data_processor.cpp -o helper_script.out
    
    ```
    
4. The host executes the compiled script:
    
    ```bash
    !./helper_script.out
    
    ```
    
5. Check the generated output statement for leaked environment variables.

---

### ✅ Expected Result

Environment variables should be securely isolated from user-submitted code, even in shared notebook environments.

---

### ❌ Observed Result

Sensitive environment variables are leaked during the execution of the disguised code. Example output:

```
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
HOME=/root
COLAB_JUPYTER_TOKEN=example-token-12345
API_KEY=my-sensitive-api-key

```

---

## 🛠️ Mitigation

To address this vulnerability, the following mitigations are recommended:

1. **Execution Confirmation**:
    - Require explicit user confirmation before executing compiled scripts in shared notebooks.
2. **Environment Variable Masking**:
    - Mask or filter sensitive environment variables in shared execution environments.
3. **User Awareness**:
    - Educate users about the risks of executing "Run all" without reviewing all code cells.
4. **Sandboxing Enhancements**:
    - Improve isolation of execution environments to ensure user-submitted code cannot access global environment variables.

---

## 🗓️ Disclosure Timeline

- **2024-12-28**: Vulnerability discovered and validated in Google Colab.
- **2024-12-29**: PoC and write-up completed. Awaiting google issuetracker response.
- **2024-12-30**: Vulnerability forwarded to the Google Cloud VRP team for further review.
- **2024-12-31**: Research is posted on my Blog.

---

## 📸 Screenshots

#### Set_Secret_Key
<img width="1044" alt="set_Secret_key" src="https://github.com/user-attachments/assets/e0da58cb-9a89-4dbd-88ec-c887a4962d95" />

#### Victim open and run all of spoofed notebook
<img width="1598" alt="run_all(secret_key)" src="https://github.com/user-attachments/assets/134e8770-45c5-4d25-846e-af673338018c" />

#### Attacker Get victim's env variable
<img width="1427" alt="attacker_display" src="https://github.com/user-attachments/assets/76f716e0-fa2a-4e02-9547-ae1ff88eddbb" />

---

## ✍️ Author

**JinGyuJeong / GiuNash**

Independent Security Researcher

GitHub: [Giunash Station](https://github.com/gyutrange/)

---

## 🔗 References

- C++ Standard Input Documentation
- Google Colab Sharing and Collaboration Documentation
- Responsible Disclosure Policy

---

### Disclaimer

This write-up is part of a responsible disclosure process. The vulnerability details and PoC are shared for educational purposes to encourage better security practices. Affected vendors have been notified, and this document will be updated with resolution statuses as they become available.

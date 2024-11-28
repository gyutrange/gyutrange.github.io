---
layout: post
title: "[Dreamhack] Basic_Crypto WriteUp"
date: 2024-11-28
author: GiuNash
tags: [Dreamhack, Crypto]
comments: true
toc: true
pinned: true
---

### 1. 문제 분석

**카이사르 암호 문제**  
**덧셈 암호 문제**

---

### 2. 기본 아이디어

알파벳은 `a-z` 즉, 26개.  
따라서 주어진 암호문을 26번 밀어본 후 그 결과를 확인.

---

### 3. 문제 풀이

문제 파일로는 다음과 같은 문자열이 주어진다.

![enc](DH_IMG/Crypto/enc.jpg)

문제에는 다음과 같이 적혀있다.

![problem](DH_IMG/Crypto/problem.jpg)

**힌트:** "로마 황제의 암호"  
딱 봐도 **카이사르 암호**임을 알 수 있다.

---

#### 풀이 과정

우선, 공백이 주어지면 `_`로 표기하고, 글자면 `j`칸 밀어주면 된다.  
이때 대문자 영역을 넘어서지 않도록 `mod 26`을 사용하고, `65`(아스키 코드 값에서 'A')를 더해준다.

그리고 결과 중에서 제일 수상하고 답 같아 보이는 것을 찾으면 된다.

![code](DH_IMG/Crypto/code.jpg)

#### 결과

제일 수상하고 답 같아 보이는 것을 찾았다.

![success](DH_IMG/Crypto/success.jpg)


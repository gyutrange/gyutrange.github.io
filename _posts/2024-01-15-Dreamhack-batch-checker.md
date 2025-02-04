---
layout: post
title: "[Dreamhack] batch_checker WriteUp"
date: 2025-01-15
author: GiuNash
tags: [Dreamhack, Reversing]
comments: true
toc: true
pinned: true
---

## 문제 접근법

배치파일을 텍스트 편집기로 열어본 결과, `@`가 붙은 문장이 있다는 것을 발견했다.  

`@`는 배치파일 실행 시 해당 라인의 출력이 숨겨지도록 설정하는 역할을 한다.  

따라서 해당 라인의 `@`를 제거하면 숨겨진 정보가 드러날 가능성이 있다고 판단했다.

---

## 문제풀이

1. **배치파일 분석**  
   - 배치파일을 메모장으로 열어 확인해보니, `@`가 붙어 있는 명령어가 존재했다.
   - 숨김처리가 된 코드가 실행 중 어떤 역할을 하는지 궁금증을 해결하기 위해 `@`를 제거했다.

2. **수정 및 실행**  
   - 배치파일에서 모든 `@`를 제거한 뒤 저장했다.
   - 수정된 배치파일을 실행하니 숨겨진 정보를 출력했고, 그 정보가 플래그였다.

3. **결과**  
   - 플래그를 획득하고 문제를 성공적으로 해결했다.

![image](https://github.com/user-attachments/assets/e55e9c15-60c8-46f2-aa91-c3dd5f5f930f)

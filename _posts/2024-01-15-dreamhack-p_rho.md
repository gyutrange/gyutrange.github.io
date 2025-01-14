---
layout: post
title: "[Dreamhack] p_rho WriteUp"
date: 2025-01-15
author: GiuNash
tags: [Dreamhack, Pwnable]
comments: true
toc: true
pinned: true
---

## 문제 접근법

### 코드 분석

`main` 함수의 구조를 Ghidra로 분석해 본 결과, 다음과 같은 사항을 발견했다:

1. `FUN_004010c0` 함수에서 값을 입력받아 `buf` 배열에 저장하고 있다.
2. 배열의 인덱스를 조작하는 로직으로 인해, **Out-of-Bounds Write**(OOB)가 가능하다.

![image](https://github.com/user-attachments/assets/96003289-094a-4631-b6b3-f8de1b770a60)

위 코드에서 local_20는 사용자 입력으로 채워지며, 이는 buf[local_18]에 저장된다.

local_18의 값이 buf[local_18]로 갱신되므로, 사용자가 입력값을 조작하여 주소 쓰기가 가능하다.

따라서 입력값을 조작하여 buf의 경계를 벗어난 메모리를 덮어쓸 수 있다.

이를 이용해 **Global Offset Table (GOT)**에 위치한 함수 포인터를 조작하여 임의의 코드를 실행할 수 있다.

마침 win 함수가 /bin/sh를 실행하고 있다.

![image](https://github.com/user-attachments/assets/5c0e97bb-4f60-4436-bbac-ee18ebc6367f)

## 문제풀이

### 버퍼 주소 확인

gdb를 사용해 win함수의 주소와 buf의 주소를 확인한다.

![image](https://github.com/user-attachments/assets/d77943e8-1e67-4a08-a463-e28566ae7b18)

![image](https://github.com/user-attachments/assets/a87f40b6-d4ec-42d7-bdf5-1454b7e3f70d)

이제 pwntools로 printf 함수의 GOT 주소를 확인하면 된다.

### GOT와 buf 사이의 거리 계산

printf_got와 buf 사이의 거리(offset)를 계산한다.

거리가 120이고, local_18 * 8 만큼 이동하므로 120/8=15 그러므로 -15만큼 이동해준 후

buf[offset]에 win 함수의 주소를 덮어쓴다.

![image](https://github.com/user-attachments/assets/16706e28-90c8-41c6-a406-e1ebced934cb)

/bin/sh 실행 후 쉘을 획득하여 플래그를 성공적으로 획득했다.







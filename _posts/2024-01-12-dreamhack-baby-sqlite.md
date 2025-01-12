---
layout: post
title: "[Dreamhack] baby-sqlite WriteUp"
date: 2025-01-12
author: GiuNash
tags: [Dreamhack, Web]
comments: true
toc: true
pinned: true
---

## Code
```python
#!/usr/bin/env python3
from flask import Flask, request, render_template, make_response, redirect, url_for, session, g
import urllib
import os
import sqlite3

app = Flask(__name__)
app.secret_key = os.urandom(32)
from flask import _app_ctx_stack

DATABASE = 'users.db'

def get_db():
    top = _app_ctx_stack.top
    if not hasattr(top, 'sqlite_db'):
        top.sqlite_db = sqlite3.connect(DATABASE)
    return top.sqlite_db


try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'


@app.route('/')
def index():
    return render_template('index.html')


@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')

    uid = request.form.get('uid', '').lower()
    upw = request.form.get('upw', '').lower()
    level = request.form.get('level', '9').lower()

    sqli_filter = ['[', ']', ',', 'admin', 'select', '\'', '"', '\t', '\n', '\r', '\x08', '\x09', '\x00', '\x0b', '\x0d', ' ']
    for x in sqli_filter:
        if uid.find(x) != -1:
            return 'No Hack!'
        if upw.find(x) != -1:
            return 'No Hack!'
        if level.find(x) != -1:
            return 'No Hack!'


    with app.app_context():
        conn = get_db()
        query = f"SELECT uid FROM users WHERE uid='{uid}' and upw='{upw}' and level={level};"
        try:
            req = conn.execute(query)
            result = req.fetchone()

            if result is not None:
                uid = result[0]
                if uid == 'admin':
                    return FLAG
        except:
            return 'Error!'
    return 'Good!'


@app.teardown_appcontext
def close_connection(exception):
    top = _app_ctx_stack.top
    if hasattr(top, 'sqlite_db'):
        top.sqlite_db.close()


if __name__ == '__main__':
    os.system('rm -rf %s' % DATABASE)
    with app.app_context():
        conn = get_db()
        conn.execute('CREATE TABLE users (uid text, upw text, level integer);')
        conn.execute("INSERT INTO users VALUES ('dream','cometrue', 9);")
        conn.commit()

    app.run(host='0.0.0.0', port=8001)
```

## 1. 문제 접근법

`level` 파라미터는 숫자형 데이터로 처리되어 쿼리에서 따옴표로 감싸지지 않았다. 

이를 이용해 SQL Injection을 수행할 수 있다고 판단했다.  

또한, 다음과 같은 제약 조건이 존재했다:

- `select`와 `admin` 같은 문자열이 필터링됨.
  
- 공백은 필터링되어 사용이 불가능함.

이를 해결하기 위해 다음과 같은 우회 기법을 사용했다:

- 공백은 `/ ** /`로 치환.

- `admin`은 `char()`와 `||` 연산자를 이용해 문자열 생성.

- `select`는 `UNION VALUES`를 사용해 우회.

이러한 분석을 바탕으로 SQL Injection 공격을 설계했다.

---

## 2. 문제풀이

**SQL Injection 가능성 확인**

   소스 코드를 분석한 결과, 로그인 로직에서 실행되는 SQL 쿼리는 다음과 같았다:
   
   ```sql
   SELECT uid FROM users WHERE uid='{uid}' and upw='{upw}' and level={level};
   ```
   
여기서 level 필드는 숫자형으로 처리되어 따옴표로 감싸지지 않으므로 SQL Injection 공격에 적합했다.

### 필터링 분석 및 우회 전략

공백 우회: 공백은 / ** /로 대체.

select 우회: UNION VALUES 구문 사용.

admin 우회: char()으로 문자열 생성

### Payload

```sql
uid=dream&upw=cometrue&level=123/**/union/**/values(char(0x61)||char(0x64)||char(0x6d)||char(0x69)||char(0x6e))
```

### Burp Suite를 이용한 공격 수행

Burp Suite를 이용해 HTTP 요청의 level 파라미터에 위 페이로드를 삽입.

플래그를 성공적으로 획득했다.

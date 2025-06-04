---
layout: post
title:  노말틱 모의해킹 스터디 8주차 강의 정리 
author: leewoojin
date: 2025-05-28 00:00:00 +0900
categories: [Pentest, W8]
tags: []
---

SQL Injection : DATA 추출 

### 우리가 원하는 select 문을 실행하는 것! 
- Union : SQL 질의 결과가 화면에 출력되는 경우 
- Error Based : SQL 에러가 발생하는 경우 
- Blind : 만능 


### SQL Injection Point 찾기 

- DB에게 SQL 질의문을 사용하는 곳 
- 파라미터 

예) `where user_id like '%__%' `를 사용한다고 예상 했을 때 
`normaltic` 과 `normaltic' and '1%'='1`이 같은 결과나오고 `normaltic' and '1%'='2` 다르다면 sqli 공격이 가능하다. 

### case when 
case when (조건) then (참일때) else (거짓일 때) end
- case when (1=2) then username else title end 
- case when (1=1) then 1 else (select 1 union select 2) end 


## SQL Injection Point 1
마이페이지에서 `woojin' and '1'='1`, `woojin' and '1'='2`와 같이 입력했을 때 `Nothing Here` 메시지가 다르게 출력되는 것으로 보아 `SQL Injection 취약점`이 존재한다고 판단됨

→ 이 차이로 인해 해당 입력 필드가 **Blind SQLi** 테스트에 활용 가능하다는 걸 확인할 수 있습니다.

<div style="display: flex; justify-content: space-between; align-items: flex-start;">
  <img src="https://github.com/user-attachments/assets/26d04666-a71d-44b4-957e-4eed2e0d1590" alt="Left Image" width="300" height="300">
  <img src="https://github.com/user-attachments/assets/d811ab52-d6b3-4fdb-b5bd-3cebb53d3351" alt="Right Image" width="300" height="300">
</div>

### 블라인드 sqli
db : sqli7 <br>
table : board, flag_table, member <br>
flag_table 안에 컬럼명 : idx, flag <br>
flag 컬럼 안에 데이터 : segfault{youdidit <br>
-> 나머지 절반은 어디가고 .. 

### 문제 해결 방법
1)  특수문자 `!@#$%^&*()` 추가 
2) 문자열 비교로 하면 대소문자 구문 x , 아스키코드로 변경 


## SQL Injection Point 2
해당 문제는 게시판에서 sqli 공격이 가능한 걸로 확인됩니다. 

<img width="1216" alt="Image" src="https://github.com/user-attachments/assets/18f7b4a6-c369-4199-9e9b-433d5b6a89d8" />
<img width="1212" alt="Image" src="https://github.com/user-attachments/assets/33a4a7f1-28a9-4725-845f-ed8bf667e5a1" />

블라인드 sqli 시작

### 공격 페이로드 
`option_val=(1=1)and(__sql__)and+username`

### database 
sqli7

### table
board, flagTable, member

### column
idx, flag


## SQL Injection Point 3

###
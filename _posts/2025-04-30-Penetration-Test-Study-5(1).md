---
title: 모의해킹 스터디 5주차 CTF 문제 풀기 
author: leewoojin
date: 2025-05-01 00:00:00 +0900
categories: [Pentest, W5]
tags: []
---
# 과제 
```code 
1. 오늘 배운거 복습 하기 (로그인 우회, 세션 탈취 등)
2. 인증 우회 실습 문제 풀기 (라이트업)
+ 문제와 비슷한 환경 구성해보기 
```

# CTF 
### Login Bypass 1

해당 페이지에 들어가 보면 일반 로그인 화면과 유사하게 생긴 것을 확인할 수 있습니다. 강의와 같이 다음과 같은 인젝션을 사용하여 로그인 우회가 가능했습니다

<img src="https://github.com/user-attachments/assets/e2faed7b-9061-4a8a-afbb-c66b0964d8db" width="100%" alt="Image 1">

---
### Login Bypass 2

> **왜 `or`은 실패했는데 `#`은 성공했는가?**

<img width="387" alt="Image" src="https://github.com/user-attachments/assets/6306b63a-0f74-486d-8443-f6d9e7ed3d62" />


#### 🔎 SQL 쿼리 구조에 따른 차이

✅ `doldol' or '1'='1` 이 성공하는 경우

```php
# 인증 / 식별이 분리해서 처리하는 쿼리
$sql = "SELECT * FROM users WHERE username = '$id'"; 
```
- 이 구조에서는 사용자 존재 여부만 확인하고,비밀번호는 PHP 코드에서 별도로 비교합니다.
- 따라서 `'1'='1'`은 항상 참이므로 사용자가 있다고 판단되어 우회가 가능합니다.


✅ `doldol' #` 이 성공하는 경우

```php
# 인증 + 식별을 동시에 처리하는 쿼리 
$sql = "SELECT * FROM users WHERE username = '$id' AND password = '$pw'";
```
- 결과적으로 `AND password = ...` 조건이 **무시**되어
- 사용자 이름만 일치하면 로그인 우회 가능 

### Login Bypass 3

UserId=doldol' or '1'='1 입력은 실패하였으나, doldol' # 입력 시 로그인에 성공하였습니다. 이후 비밀번호를 변경하면 로그인에 실패하는 것으로 보아, 해당 로그인 로직이 식별과 인증이 분리된 구조임을 확인할 수 있었습니다.

<img width="746" alt="Image" src="https://github.com/user-attachments/assets/f4de9268-91fe-49d9-a8bd-5443f0a076cc" />

ORDER BY {숫자}# 구문을 통해 컬럼 수를 확인하였으며, 3 이상에서는 오류가 발생해 총 2개의 컬럼이 존재함을 파악할 수 있었습니다.

#### ❓ 왜 컬럼 수를 확인하는가?
`UNION SELECT` 구문이 정상적으로 작동하기 위해서는  
컬럼 수를 정확히 파악해야 원하는 데이터를 주입할 수 있으며, 그중 어떤 컬럼이 실제 페이지에 출력되는지도 함께 판단할 수 있습니다.


> 로그인을 하기 위해 컬럼 2개(`id`, `pass`)만 사용하는 게 아닌가? 

로그인 입력값은 일반적으로 아이디와 비밀번호 두 개지만, 실제 SQL 쿼리는 로그인 이후 사용자 정보를 활용하기 위해  `id`, `username`, `password`, `role` 등 **여러 컬럼을 함께 조회**하는 경우가 많습니다.

<table>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/efd3d16b-673b-4835-a7cf-019bbbec0877" width="300"></td>
    <td><img src="https://github.com/user-attachments/assets/0d234f54-65c5-46f6-be87-031e974f630e" width="300"></td>
    <td><img src="https://github.com/user-attachments/assets/2c9ee5f3-b520-4e93-aabd-727279f6d42b" width="300"></td>
  </tr>
</table>

#### ❓ UNION SELECT란?
`UNION SELECT`는 SQL 문법 중 하나로, **두 개 이상의 SELECT 쿼리 결과를 하나로 결합**할 때 사용됩니다.  

앞서 `ORDER BY` 구문을 통해 컬럼 수가 **2개**임을 확인하였습니다.   따라서 다음과 같이 2개의 값을 갖는 `UNION SELECT`를 사용할 수 있습니다: 

<img width="451" alt="Image" src="https://github.com/user-attachments/assets/3be904be-4446-4bf5-a560-06b263c1f275" />

#### 💡 컬럼 수가 3개 이상인 경우
컬럼 수가 3개 이상일 경우에는 UNION SELECT도 3개의 값을 반환해야 하며, 뒤에 붙는 쿼리의 조건(AND password='...' 등)을 제거하기 위해
반드시 주석 처리 구문 -- 또는 #를 붙여야 합니다.
> (예시) ' UNION SELECT 'normaltic3', '1111', 'test' -- 

### Login Bypass 4
해당 로그인 페이지는 `Login Bypass 3` 문제와 마찬가지로 **식별과 인증이 분리된 구조**로 보이며,  `UNION SELECT`를 통한 일반적인 우회가 통하지 않았습니다.

<img width="753" alt="Image" src="https://github.com/user-attachments/assets/0d0dd5ba-3e24-4a03-b4eb-c763646974fb" />

지난 시간에 학습한 로그인 로직을 바탕으로, 이 페이지는 **식별 / 인증 분리 구조이며, 해시 기반의 인증 방식**이 적용된 것으로 판단됩니다.  

> PHP에서 사용되는 주요 해시 함수

`md5()`, `sha1()`, `hash('sha256')`, `hash('sha512')` `password_hash()` 중 하나라고 생각해 첫번째부터 넣어봤습니다. 

<img width="1138" alt="Image" src="https://github.com/user-attachments/assets/8205d001-4d28-48f0-8fc3-83c2250683cf" />

### Login Bypass 5
이제 이 정도의 구조는 빠르게 식별할 수 있는 수준에 도달하였습니다. 두 번째 쿠키 패킷에서 `UserID` 값을 `doldol`에서 `normaltic5`로 변조해 주었더니, **플래그를 획득**할 수 있었습니다.

<img width="745" alt="Image" src="https://github.com/user-attachments/assets/db7c3a4d-c629-45f6-90f6-0f18af4ef1c6" />

### Get Admin

인증 우회 세션 값인 loginUser를 doldol 대신 admiin 수정 하면 쉽게 접속 할 수 있습니다. (nomaltic1 인줄 알고 계속 시도했는데.. )

<img width="802" alt="Image" src="https://github.com/user-attachments/assets/03c48173-9397-481c-aba9-e7ec3f190614" />

### Admin is Mine
```
이 문제는 정상 사용자 계정으로 로그인한 후, 패킷을 조작하여 관리자 권한을 탈취하고 플래그를 획득하는 시나리오입니다.
```
처음 `doldol / dol1234` 계정으로 로그인하면, 패킷이 두 번 전송되는 것을 확인할 수 있습니다.

<img alt="Image" src="https://github.com/user-attachments/assets/12e5d6d8-e8cd-4e30-b711-f0bf42434c24" />

먼저 본인의 계정으로 로그인하고, 정상적인 첫 번째 로그인 요청 패킷을 그대로 두고 Forward 클릭합니다.

<img width="1132" alt="Image" src="https://github.com/user-attachments/assets/469e789f-e51d-40d4-acf7-cd40e5f423ba" />

두 번째 패킷에서 아이디를 admin으로 수정합니다.

<img alt="Image" src="https://github.com/user-attachments/assets/d76d2b89-4309-4706-bbdc-800f3030c6ba" />

이후 패킷을 확인해보면 세션이 2개 생성된 것을 확인할 수 있습니다. 마지막 패킷부터 (순서 중요) 웹사이트로 다시 전송하면 플래그를 획득할 수 있습니다.

<img width="1131" alt="Image" src="https://github.com/user-attachments/assets/57a7cca7-cd99-448c-b7e4-6592b09dfa72" />

### PIN CODE Bypass
step1.php가 수상해보여 1,2,3 각각 넣어보니 플래그를 획득할 수 있었습니다. 

<img width="977" alt="Image" src="https://github.com/user-attachments/assets/c551fa0d-e291-45ea-b62f-781f68598c9e" />

### Pin Code Crack

Burp Intruder를 활용해 문제를 해결하고자 하였으나, 속도가 너무 느려서 

<img width="1083" alt="Image" src="https://github.com/user-attachments/assets/9508377a-a210-45bc-9057-f7802cf4cb66" />

자동화된 스크립트를 직접 작성하여 시도해보는 방식이 보다 효과적이라 판단하였습니다.

<img width="873" alt="Image" src="https://github.com/user-attachments/assets/3a8ed69e-2bff-48d1-a392-49c58e7faf0b" />

```python
import requests

print("[*] 브루트포스 시작")

for i in range(10000):
    otp = str(i).zfill(4)
    response = requests.get(f"http://ctf.segfaulthub.com:1129/6/checkOTP.php?otpNum={otp}")

    if 'Login Fail...' not in response.text:
        print(f"[+] OTP 찾음: {otp}")
        break
    else:
        print(f"[-] 시도 중: {otp}", end="\r")

```
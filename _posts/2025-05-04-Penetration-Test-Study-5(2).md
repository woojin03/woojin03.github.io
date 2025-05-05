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

이 문제는 **Login Bypass 3**에서 사용된 `인증/식별 분리 구조`에  
**MD5 해시 기반의 비밀번호 검증**을 사용한 문제입니다. 

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

## 🔐 Login Bypass 5

이 문제는 로그인 후 쿠키의 `loginUser` 값을 이용해 **사용자 권한을 판단**하는 구조입니다.  
**guest 계정으로 로그인한 뒤, 쿠키 값을 `admin`으로 변경하면 플래그를 탈취할 수 있습니다.**

### ✅ 로그인 로직
이번 문제의 로그인 로직은 다음과 같이 구성되어 있습니다:
```php
// admin으로 로그인 시 플래그값 획득
if ($_COOKIE['loginUser'] === 'admin') {
    echo "FLAG!";
}
```
> 서버는 $_COOKIE['loginUser'] 값으로 관리자 권한 여부를 판단함
```php
//  Prepared Statement로 SQL Injection 방지
$stmt = $conn->prepare("SELECT * FROM ctf_users1 WHERE id = ? AND pass = ?");
$stmt->bind_param("ss", $id, $pw);     // 두 개의 문자열 파라미터 바인딩
$stmt->execute();                      // 쿼리 실행
$result = $stmt->get_result();         // 결과 가져오기
$row = $result->fetch_assoc();         // 연관 배열로 한 줄 가져옴
```
>  `id`와 `pw`를 SQL에 직접 삽입하지 않고 `?`로 바인딩함으로써, `' OR 1=1 -- `같은 공격도 문자열로 처리되어 SQL Injection을 원천 차단합니다.

```php 
// 로그인 성공 시 세션 저장 및 쿠키 설정 플래그
if ($row) {
    $login_id = $row['id'];
    $_SESSION['ctf_user5'] = $login_id;
    $should_set_cookie = true;
}

// 쿠키 설정 (HTML 출력 전)
if ($should_set_cookie) {
    setcookie("loginUser", $login_id, time() + 3600, "/");
}
```
> 로그인 성공 시 세션을 저장하고, 쿠키 설정 플래그를 통해 HTML 출력 전에 loginUser 쿠키를 안전하게 설정합니다.

```php
// loginUser 쿠키 삭제
if (isset($_COOKIE['loginUser'])) {
    setcookie("loginUser", "", time() - 3600, "/");
}
```
> 로그아웃 시 loginUser 쿠키를 삭제합니다

### ✅ 우회 입력 예시
<img width="1195" alt="Image" src="https://github.com/user-attachments/assets/f963c62e-d95d-43af-a833-e9f576f42a03" />

크롬 기준: F12 → Application → Cookies → loginUser → 값 변경 → 페이지 새로고침

Burp 기준: 로그인 후의 인증 요청을 Repeater로 보내고, Cookie 값을 수정

## 🔐 Admin is Mine
이 문제는 JSON 기반의 로그인 요청을 처리하는 API를 대상으로 하며, 로그인 결과는 HTTP 응답의 JSON 본문에 "result": "ok" 또는 "result": "fail"로 나타납니다. 공격자는 BurpSuite를 활용해 서버의 응답을 조작하하여 관리자 계정에서 플래그를 획득할 수 있습니다.


### ✅ 로그인 로직

```php
if ($row) {
    $response['result'] = "ok";

    if ($row['id'] === 'admin') {
        $response['flag'] = "flag{}";
    }
} else {
    $response['result'] = "fail";
}
```
> 서버는 POST /submit.php 요청으로 로그인 정보를 받아, 인증 성공 시 JSON 형태로 "result": "ok"을 응답합니다.

### ✅ 우회 입력 예시

- **정상 계정 세션 재사용**:  
  `guest` 계정으로 로그인하여 받은 인증 패킷(`{"result":"ok"}`)을 저장해두었다가, `admin` 계정으로 로그인 시 인증에 실패하면,  
  이전에 저장해둔 `ok` 응답 패킷을 **BurpSuite Repeater**에서 재전송하여 **관리자 권한을 탈취**합니다.

- **응답 위조**:  
  `admin` 계정으로 로그인하여 실패 응답(`{"result":"fail"}`)을 받은 후, **BurpSuite에서 응답 내용을 `{"result":"ok"}`로 수정**하여, 클라이언트 측에서 **인증에 성공한 것처럼 동작**하게 합니다.

### ✅  실습 문제
 BurpSuite와 가상머신 간 네트워크 연결 문제

- 로컬 환경의 BurpSuite는 일반적으로 `127.0.0.1:8080`에서 프록시를 바인딩하고 대기합니다.
- 하지만 가상머신에서 실행 중인 웹 애플리케이션이 `localhost` (`127.0.0.1`) 기준으로만 실행되면,
  **해당 요청은 가상머신 내부에서만 처리되며, 호스트(로컬)에서 실행 중인 BurpSuite에서는 관찰할 수 없습니다.**

## 🔐 PIN CODE Bypass
이 문제는 **페이지 간 이동 흐름을 활용한 CTF 문제**입니다.  
플레이어는 각 단계(step1 → step2 → step3 → step4)를 거치며 탈옥 시나리오를 진행하게 됩니다.
### ✅ 페이지 예시 
<table>
  <tr>
    <td>
      <img width="100%" alt="Image" src="https://github.com/user-attachments/assets/85785c90-8fea-4dfb-9d7c-0e0c11219601" />
    </td>
    <td>
      <img src="https://github.com/user-attachments/assets/7212745e-4748-44ca-bd61-1fe06b3cf43b" width="100%" />
    </td>
  </tr>
  <tr>
    <td>
      <img src="https://github.com/user-attachments/assets/abf88d43-10c8-48ac-89d8-9bf50835a287" width="100%" />
    </td>
    <td>
      <img src="https://github.com/user-attachments/assets/d2e8e64c-1865-4a5b-9dc2-4b3ff9fdea30" width="100%" />
    </td>
  </tr>
</table>

## 🔐 Pin Code Crack

이 문제는 **브루트포스**를 이용해 숫자 4자리 **PIN 코드**를 추측하여 금고를 해제하고 플래그를 획득하는 웹 보안 실습 문제입니다.

<img width="438" alt="Image" src="https://github.com/user-attachments/assets/d85dc8cd-36cc-44f7-bd9d-91386752b015" />

- 입력은 `0000`부터 `9999`까지의 4자리 숫자이며, 반복 요청을 통해 정답을 찾을 수 있습니다.
- 정답을 입력하면 플래그가 팝업으로 표시됩니다.
- BurpSuite나 Python `requests`를 활용해 자동화 공격이 가능합니다.


## 트러블 슈팅 (질문) 

### 🔸 1. 문제 유사성 관련 질문
- **Get Admin** 문제와 **Login Bypass 5** 문제는 풀이 방식이 동일한 것으로 보입니다. 두 문제의 차이점이 무엇인지 알고 싶습니다.

### 🔸 2. 로그인 우회 로직 작성 방식
- **Login Bypass 1** 문제를 구현하며 다음과 같은 로그인 로직을 작성했는데, 보안상 적절한 구현 방식인지 궁금합니다:

```php
if (preg_match("/--|#|\/\*/i", $id)) {
    $row = false; // 강제로 로그인 실패 처리
} else {
    $sql = "SELECT * FROM ctf_users1 WHERE id='$id' AND pass='$pw'";
    $result = mysqli_query($conn, $sql);
    $row = mysqli_fetch_array($result);
}
```
정규표현식으로 SQL 인젝션을 일부 막는 방식인데, 이 접근이 안전한지, 더 나은 방식이 있는지 궁금합니다.

### 🔸 3. 브루트포스 자동화 중 이상 현상
Pin Code Crack 문제를 풀기 위해 작성한 브루트포스 Python 코드에서 Found PIN: 0000 이 출력되며, 실제 플래그는 잘 출력되지만 PIN이 고정된 값처럼 나오는 문제가 있습니다.

매 요청마다 증가하면서 시도는 되지만 출력은 항상 0000이 나옵니다. 원인이 클라이언트 캐시인지, 응답 처리 로직 때문인지 확인 필요합니다.

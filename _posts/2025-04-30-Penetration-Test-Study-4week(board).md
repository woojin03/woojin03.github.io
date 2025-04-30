---
title: 모의해킹 스터디 4주차 게시판 만들기 
author: leewoojin
date: 2025-04-30
categories: [Pentest, W4]
tags: [board, sql, Pentest ]
---
이번 주차에서는 1~3주차 동안 구현한 웹페이지에 게시판 기능을 추가해보겠습니다.  

**1.유저 게시판 테이블 설계**

```sql 
CREATE TABLE user_board (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  content TEXT NOT NULL,
  author VARCHAR(50) NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  views INT DEFAULT 0
); 
```

**2. 게시판 목록**

<img width="1371" alt="게시판 목록 화면" src="https://github.com/user-attachments/assets/9d240686-02fa-48d7-a25a-93617f720117" />

```php
<table class="table table-hover text-center">
  <thead class="table-light">
    <tr>
      <th>번호</th>
      <th>제목</th>
      <th>작성자</th>
      <th>작성일</th>
      <th>조회수</th>
    </tr>
  </thead>
  <tbody>
    <?php if (isset($result) && $result->num_rows > 0): ?>
      <?php while($row = $result->fetch_assoc()): ?>
        <tr>
          <td><?= $row['id'] ?></td>
          <td><a href="board_view.php?id=<?= $row['id'] ?>"><?= htmlspecialchars($row['title']) ?></a></td>
          <td><?= htmlspecialchars($row['author']) ?></td>
          <td><?= $row['created_at'] ?></td>
          <td><?= $row['views'] ?></td>
        </tr>
      <?php endwhile; ?>
    <?php endif; ?>
  </tbody>
</table>
```

**3. 게시판 글 상세 보기**

<img width="1358" alt="Image" src="https://github.com/user-attachments/assets/37ce8b78-7a06-491f-827b-b710935165a1" />

```php
session_start();
if (!isset($_SESSION['name'])) {
    header("Location: login.php");
    exit;
}

require_once "db_conn.php";
$db = db_connect();

// GET 파라미터 확인
$post_id = $_GET['id'] ?? null;
if (!$post_id) {
    echo "<script>alert('잘못된 접근입니다.'); history.back();</script>";
    exit;
}

if (!isset($_SESSION['viewed_posts'])) {
  $_SESSION['viewed_posts'] = [];
}

if (!in_array($post_id, $_SESSION['viewed_posts'])) {
  // 조회수 증가
  $update_sql = "UPDATE user_board SET views = views + 1 WHERE id = ?";
  $stmt = $db->prepare($update_sql);
  $stmt->bind_param("i", $post_id);
  $stmt->execute();

  // 본 게시글 ID를 세션에 기록
  $_SESSION['viewed_posts'][] = $post_id;
}

// 게시글 가져오기
$sql = "SELECT * FROM user_board WHERE id = ?";
$stmt = $db->prepare($sql);
$stmt->bind_param("i", $post_id);
$stmt->execute();
$result = $stmt->get_result();
$post = $result->fetch_assoc();

if (!$post) {
    echo "<script>alert('게시글이 존재하지 않습니다.'); history.back();</script>";
    exit;
} 
```

**+ 댓글 기능도 추가**
<img width="1425" alt="Image" src="https://github.com/user-attachments/assets/b85126f5-de95-4f05-86ed-cd685455384c" />

```sql
CREATE TABLE user_comments (
  id INT AUTO_INCREMENT PRIMARY KEY,
  board_id INT NOT NULL,
  author VARCHAR(50) NOT NULL,
  content TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (board_id) REFERENCES user_board(id) ON DELETE CASCADE
);
```

```php
// 댓글 등록
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['comment'])) {
    $comment = trim($_POST['comment']);
    if ($comment !== '') {
        $stmt = $db->prepare("INSERT INTO user_comments (board_id, author, content) VALUES (?, ?, ?)");
        $stmt->bind_param("iss", $post_id, $username, $comment);
        $stmt->execute();
        header("Location: board_view.php?id=$post_id");
        exit;
    }
}
```

**3. 게시판 글 작성**

<img width="774" alt="Image" src="https://github.com/user-attachments/assets/486d9ba0-6f4f-4452-8beb-80a42df9e7c0" />

``` php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $title = trim($_POST['title'] ?? '');
    $content = trim($_POST['content'] ?? '');

    if ($title !== '' && $content !== '') {
        $stmt = $db->prepare("INSERT INTO user_board (title, content, author) VALUES (?, ?, ?)");
        $stmt->bind_param("sss", $title, $content, $username);
        $stmt->execute();
        header("Location: main.php?page=board");
        exit;
    } else {
        $error = "제목과 내용을 모두 입력해주세요.";
    }
}
```

**+ 게시판 글 수정**

<img width="1358" alt="Image" src="https://github.com/user-attachments/assets/81053b02-c561-45b3-9933-db7d6556800b" />

```php
$post_id = $_GET['id'] ?? null;

$stmt = $db->prepare("SELECT * FROM user_board WHERE id = ?");
$stmt->bind_param("i", $post_id);
$stmt->execute();
$result = $stmt->get_result();
$post = $result->fetch_assoc();

if (!$post || $post['author'] !== $_SESSION['name']) {
    echo "<script>alert('수정 권한이 없습니다.'); history.back();</script>";
    exit;
}if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $title = trim($_POST['title']);
    $content = trim($_POST['content']);

    if ($title !== '' && $content !== '') {
        $stmt = $db->prepare("UPDATE user_board SET title = ?, content = ? WHERE id = ?");
        $stmt->bind_param("ssi", $title, $content, $post_id);
        $stmt->execute();

        header("Location: board_view.php?id=$post_id");
        exit;
    }
}
```
**4. 페이징 구현**

<img width="1352" alt="Image" src="https://github.com/user-attachments/assets/45d71872-6400-4639-8b73-e810bb7f298a" />

```php
// 현재 페이지 번호 가져오기 (기본값은 1)
$page_num = isset($_GET['page_num']) ? max(1, (int)$_GET['page_num']) : 1;

// 한 페이지당 게시글 수 설정
$posts_per_page = 5;

// OFFSET 계산 (건너뛸 게시글 수)
$offset = ($page_num - 1) * $posts_per_page;

// 전체 게시글 수 조회
$count_result = mysqli_query($db, "SELECT COUNT(*) as total FROM user_board");
$total_posts = mysqli_fetch_assoc($count_result)['total'];

// 총 페이지 수 계산
$total_pages = ceil($total_posts / $posts_per_page);

// 현재 페이지에 해당하는 게시글 목록 조회 (최신순 정렬)
$sql = "SELECT * FROM user_board ORDER BY created_at DESC LIMIT $posts_per_page OFFSET $offset";
$result = mysqli_query($db, $sql);

// 페이지네이션 출력
if ($total_pages > 1): ?>
  <nav aria-label="페이지네이션">
    <ul class="pagination justify-content-center">
      <?php for ($i = 1; $i <= $total_pages; $i++): ?>
        <li class="page-item <?= $i === $page_num ? 'active' : '' ?>">
          <!-- 페이지 번호에 따라 page_num 값을 쿼리스트링으로 넘김 -->
          <a class="page-link" href="?page=board&page_num=<?= $i ?>"><?= $i ?></a>
        </li>
      <?php endfor; ?>
    </ul>
  </nav>
<?php endif; ?>
```

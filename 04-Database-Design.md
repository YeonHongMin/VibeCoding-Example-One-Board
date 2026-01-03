# Database Design (데이터베이스 설계)
## One Board 데이터베이스 설계서

### 1. 데이터베이스 개요

#### 1.1 데이터베이스 정보
- **DBMS**: SQLite 3
- **문자 인코딩**: UTF-8
- **트랜잭션**: 지원 (ACID 준수)
- **WAL 모드**: 활성화 권장 (성능 향상)

#### 1.2 명명 규칙
- 테이블명: 소문자, 복수형, 언더스코어 구분 (예: `members`, `board_posts`)
- 컬럼명: 소문자, 언더스코어 구분 (예: `user_id`, `created_at`)
- 인덱스명: `idx_테이블명_컬럼명` 형식
- 외래키명: `fk_테이블명_참조테이블명` 형식

---

### 2. ERD (Entity Relationship Diagram)

```
[members] ──┬── [board_posts] ──┬── [post_files]
            │                   │
            │                   └── [post_comments]
            │
            └── [member_sessions]

[boards] ───┴── [board_posts]
    │
    └── [board_skins]

[skins]
```

---

### 3. 테이블 상세 설계

#### 3.1 회원 테이블 (members)

##### 3.1.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | 회원 고유 ID |
| email | VARCHAR(255) | UNIQUE, NOT NULL | 이메일 주소 |
| password_hash | VARCHAR(255) | NOT NULL | 비밀번호 해시 |
| nickname | VARCHAR(50) | UNIQUE, NOT NULL | 닉네임 |
| name | VARCHAR(100) | NULL | 실명 |
| phone | VARCHAR(20) | NULL | 전화번호 |
| profile_image | VARCHAR(500) | NULL | 프로필 이미지 경로 |
| role | TINYINT | NOT NULL, DEFAULT 1 | 권한 (0:비회원, 1:일반회원, 2:관리자) |
| email_verified | BOOLEAN | NOT NULL, DEFAULT FALSE | 이메일 인증 여부 |
| email_verification_token | VARCHAR(100) | NULL | 이메일 인증 토큰 |
| status | TINYINT | NOT NULL, DEFAULT 1 | 상태 (0:탈퇴, 1:정상, 2:정지) |
| last_login_at | DATETIME | NULL | 최종 로그인 시간 |
| login_fail_count | INT | NOT NULL, DEFAULT 0 | 로그인 실패 횟수 |
| locked_until | DATETIME | NULL | 계정 잠금 해제 시간 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 가입일 |
| updated_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |

##### 3.1.2 인덱스

```sql
CREATE INDEX idx_members_email ON members(email);
CREATE INDEX idx_members_nickname ON members(nickname);
CREATE INDEX idx_members_status ON members(status);
CREATE INDEX idx_members_role ON members(role);
```

##### 3.1.3 테이블 생성 SQL (SQLite)

```sql
CREATE TABLE members (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    nickname TEXT NOT NULL UNIQUE,
    name TEXT,
    phone TEXT,
    profile_image TEXT,
    role INTEGER NOT NULL DEFAULT 1, -- 0:비회원, 1:일반회원, 2:관리자
    email_verified INTEGER NOT NULL DEFAULT 0, -- 0: FALSE, 1: TRUE
    email_verification_token TEXT,
    status INTEGER NOT NULL DEFAULT 1, -- 0:탈퇴, 1:정상, 2:정지
    last_login_at TEXT, -- ISO 8601 형식 또는 Unix timestamp
    login_fail_count INTEGER NOT NULL DEFAULT 0,
    locked_until TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- 인덱스 생성
CREATE INDEX idx_members_email ON members(email);
CREATE INDEX idx_members_nickname ON members(nickname);
CREATE INDEX idx_members_status ON members(status);
CREATE INDEX idx_members_role ON members(role);

-- updated_at 자동 업데이트 트리거 (SQLite는 ON UPDATE CURRENT_TIMESTAMP 미지원)
CREATE TRIGGER update_members_timestamp 
AFTER UPDATE ON members
BEGIN
    UPDATE members SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

---

#### 3.2 게시판 테이블 (boards)

##### 3.2.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | 게시판 고유 ID |
| name | VARCHAR(100) | NOT NULL | 게시판 이름 |
| description | TEXT | NULL | 게시판 설명 |
| board_key | VARCHAR(50) | UNIQUE, NOT NULL | 게시판 URL 식별자 |
| category | VARCHAR(50) | NULL | 게시판 카테고리 |
| icon | VARCHAR(500) | NULL | 게시판 아이콘 경로 |
| skin_id | BIGINT UNSIGNED | NULL | 적용된 스킨 ID |
| read_permission | TINYINT | NOT NULL, DEFAULT 0 | 읽기 권한 (0:전체, 1:회원, 2:관리자) |
| write_permission | TINYINT | NOT NULL, DEFAULT 1 | 쓰기 권한 (0:전체, 1:회원, 2:관리자) |
| comment_permission | TINYINT | NOT NULL, DEFAULT 1 | 댓글 권한 (0:전체, 1:회원, 2:관리자) |
| allow_file_upload | BOOLEAN | NOT NULL, DEFAULT TRUE | 파일 업로드 허용 |
| max_file_count | INT | NOT NULL, DEFAULT 5 | 최대 파일 개수 |
| max_file_size | INT | NOT NULL, DEFAULT 5242880 | 최대 파일 크기 (바이트, 기본 5MB) |
| allowed_file_types | VARCHAR(255) | NULL | 허용 파일 타입 (콤마 구분) |
| post_count | INT | NOT NULL, DEFAULT 0 | 게시글 수 |
| display_order | INT | NOT NULL, DEFAULT 0 | 표시 순서 |
| is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | 활성화 여부 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 생성일 |
| updated_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |

##### 3.2.2 인덱스

```sql
CREATE INDEX idx_boards_board_key ON boards(board_key);
CREATE INDEX idx_boards_category ON boards(category);
CREATE INDEX idx_boards_is_active ON boards(is_active);
CREATE INDEX idx_boards_display_order ON boards(display_order);
CREATE INDEX idx_boards_skin_id ON boards(skin_id);
```

##### 3.2.3 테이블 생성 SQL (SQLite)

```sql
CREATE TABLE boards (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    board_key TEXT NOT NULL UNIQUE,
    category TEXT,
    icon TEXT,
    skin_id INTEGER,
    read_permission INTEGER NOT NULL DEFAULT 0, -- 0:전체, 1:회원, 2:관리자
    write_permission INTEGER NOT NULL DEFAULT 1, -- 0:전체, 1:회원, 2:관리자
    comment_permission INTEGER NOT NULL DEFAULT 1, -- 0:전체, 1:회원, 2:관리자
    allow_file_upload INTEGER NOT NULL DEFAULT 1, -- 0: FALSE, 1: TRUE
    max_file_count INTEGER NOT NULL DEFAULT 5,
    max_file_size INTEGER NOT NULL DEFAULT 5242880,
    allowed_file_types TEXT,
    post_count INTEGER NOT NULL DEFAULT 0,
    display_order INTEGER NOT NULL DEFAULT 0,
    is_active INTEGER NOT NULL DEFAULT 1, -- 0: FALSE, 1: TRUE
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now')),
    FOREIGN KEY (skin_id) REFERENCES skins(id) ON DELETE SET NULL
);

-- 인덱스 생성
CREATE INDEX idx_boards_board_key ON boards(board_key);
CREATE INDEX idx_boards_category ON boards(category);
CREATE INDEX idx_boards_is_active ON boards(is_active);
CREATE INDEX idx_boards_display_order ON boards(display_order);
CREATE INDEX idx_boards_skin_id ON boards(skin_id);

-- updated_at 자동 업데이트 트리거
CREATE TRIGGER update_boards_timestamp 
AFTER UPDATE ON boards
BEGIN
    UPDATE boards SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

---

#### 3.3 게시글 테이블 (board_posts)

##### 3.3.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | 게시글 고유 ID |
| board_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY | 게시판 ID |
| member_id | BIGINT UNSIGNED | NULL, FOREIGN KEY | 작성자 ID (NULL:비회원) |
| author_name | VARCHAR(50) | NULL | 작성자명 (비회원용) |
| author_password | VARCHAR(255) | NULL | 비회원 비밀번호 (해시) |
| title | VARCHAR(255) | NOT NULL | 제목 |
| content | LONGTEXT | NOT NULL | 내용 |
| category | VARCHAR(50) | NULL | 카테고리 |
| tags | VARCHAR(500) | NULL | 태그 (콤마 구분) |
| view_count | INT | NOT NULL, DEFAULT 0 | 조회수 |
| like_count | INT | NOT NULL, DEFAULT 0 | 좋아요 수 |
| comment_count | INT | NOT NULL, DEFAULT 0 | 댓글 수 |
| is_notice | BOOLEAN | NOT NULL, DEFAULT FALSE | 공지사항 여부 |
| is_pinned | BOOLEAN | NOT NULL, DEFAULT FALSE | 고정글 여부 |
| is_secret | BOOLEAN | NOT NULL, DEFAULT FALSE | 비밀글 여부 |
| status | TINYINT | NOT NULL, DEFAULT 1 | 상태 (0:삭제, 1:정상, 2:숨김) |
| ip_address | VARCHAR(45) | NULL | 작성자 IP 주소 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 작성일 |
| updated_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |
| deleted_at | DATETIME | NULL | 삭제일 (Soft Delete) |

##### 3.3.2 인덱스

```sql
CREATE INDEX idx_posts_board_id ON board_posts(board_id);
CREATE INDEX idx_posts_member_id ON board_posts(member_id);
CREATE INDEX idx_posts_status ON board_posts(status);
CREATE INDEX idx_posts_created_at ON board_posts(created_at);
CREATE INDEX idx_posts_view_count ON board_posts(view_count);
CREATE INDEX idx_posts_is_notice ON board_posts(is_notice);
CREATE INDEX idx_posts_is_pinned ON board_posts(is_pinned);
CREATE FULLTEXT INDEX ft_posts_title_content ON board_posts(title, content);
```

##### 3.3.3 테이블 생성 SQL

```sql
CREATE TABLE board_posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    board_id INTEGER NOT NULL,
    member_id INTEGER,
    author_name TEXT,
    author_password TEXT,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    category TEXT,
    tags TEXT,
    view_count INTEGER NOT NULL DEFAULT 0,
    like_count INTEGER NOT NULL DEFAULT 0,
    comment_count INTEGER NOT NULL DEFAULT 0,
    is_notice INTEGER NOT NULL DEFAULT 0, -- 0: FALSE, 1: TRUE
    is_pinned INTEGER NOT NULL DEFAULT 0, -- 0: FALSE, 1: TRUE
    is_secret INTEGER NOT NULL DEFAULT 0, -- 0: FALSE, 1: TRUE
    status INTEGER NOT NULL DEFAULT 1, -- 0:삭제, 1:정상, 2:숨김
    ip_address TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now')),
    deleted_at TEXT,
    FOREIGN KEY (board_id) REFERENCES boards(id) ON DELETE CASCADE,
    FOREIGN KEY (member_id) REFERENCES members(id) ON DELETE SET NULL
);

-- 인덱스 생성
CREATE INDEX idx_posts_board_id ON board_posts(board_id);
CREATE INDEX idx_posts_member_id ON board_posts(member_id);
CREATE INDEX idx_posts_status ON board_posts(status);
CREATE INDEX idx_posts_created_at ON board_posts(created_at);
CREATE INDEX idx_posts_view_count ON board_posts(view_count);
CREATE INDEX idx_posts_is_notice ON board_posts(is_notice);
CREATE INDEX idx_posts_is_pinned ON board_posts(is_pinned);

-- FULLTEXT 검색을 위한 FTS5 가상 테이블 (선택사항)
CREATE VIRTUAL TABLE posts_fts USING fts5(title, content, content_rowid=id);

-- updated_at 자동 업데이트 트리거
CREATE TRIGGER update_posts_timestamp 
AFTER UPDATE ON board_posts
BEGIN
    UPDATE board_posts SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

---

#### 3.4 댓글 테이블 (post_comments)

##### 3.4.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | 댓글 고유 ID |
| post_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY | 게시글 ID |
| member_id | BIGINT UNSIGNED | NULL, FOREIGN KEY | 작성자 ID (NULL:비회원) |
| author_name | VARCHAR(50) | NULL | 작성자명 (비회원용) |
| author_password | VARCHAR(255) | NULL | 비회원 비밀번호 (해시) |
| parent_id | BIGINT UNSIGNED | NULL, FOREIGN KEY | 부모 댓글 ID (대댓글) |
| content | TEXT | NOT NULL | 댓글 내용 |
| like_count | INT | NOT NULL, DEFAULT 0 | 좋아요 수 |
| status | TINYINT | NOT NULL, DEFAULT 1 | 상태 (0:삭제, 1:정상, 2:숨김) |
| ip_address | VARCHAR(45) | NULL | 작성자 IP 주소 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 작성일 |
| updated_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |
| deleted_at | DATETIME | NULL | 삭제일 (Soft Delete) |

##### 3.4.2 인덱스

```sql
CREATE INDEX idx_comments_post_id ON post_comments(post_id);
CREATE INDEX idx_comments_member_id ON post_comments(member_id);
CREATE INDEX idx_comments_parent_id ON post_comments(parent_id);
CREATE INDEX idx_comments_status ON post_comments(status);
CREATE INDEX idx_comments_created_at ON post_comments(created_at);
```

##### 3.4.3 테이블 생성 SQL

```sql
CREATE TABLE post_comments (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    post_id INTEGER NOT NULL,
    member_id INTEGER,
    author_name TEXT,
    author_password TEXT,
    parent_id INTEGER,
    content TEXT NOT NULL,
    like_count INTEGER NOT NULL DEFAULT 0,
    status INTEGER NOT NULL DEFAULT 1, -- 0:삭제, 1:정상, 2:숨김
    ip_address TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now')),
    deleted_at TEXT,
    FOREIGN KEY (post_id) REFERENCES board_posts(id) ON DELETE CASCADE,
    FOREIGN KEY (member_id) REFERENCES members(id) ON DELETE SET NULL,
    FOREIGN KEY (parent_id) REFERENCES post_comments(id) ON DELETE CASCADE
);

-- 인덱스 생성
CREATE INDEX idx_comments_post_id ON post_comments(post_id);
CREATE INDEX idx_comments_member_id ON post_comments(member_id);
CREATE INDEX idx_comments_parent_id ON post_comments(parent_id);
CREATE INDEX idx_comments_status ON post_comments(status);
CREATE INDEX idx_comments_created_at ON post_comments(created_at);

-- updated_at 자동 업데이트 트리거
CREATE TRIGGER update_comments_timestamp 
AFTER UPDATE ON post_comments
BEGIN
    UPDATE post_comments SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

---

#### 3.5 파일 테이블 (post_files)

##### 3.5.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | 파일 고유 ID |
| post_id | BIGINT UNSIGNED | NULL, FOREIGN KEY | 게시글 ID |
| comment_id | BIGINT UNSIGNED | NULL, FOREIGN KEY | 댓글 ID |
| member_id | BIGINT UNSIGNED | NULL, FOREIGN KEY | 업로드한 회원 ID |
| file_type | VARCHAR(20) | NOT NULL | 파일 타입 (image/file) |
| original_name | VARCHAR(255) | NOT NULL | 원본 파일명 |
| stored_name | VARCHAR(255) | NOT NULL | 저장된 파일명 |
| file_path | VARCHAR(500) | NOT NULL | 파일 경로 |
| file_size | BIGINT UNSIGNED | NOT NULL | 파일 크기 (바이트) |
| mime_type | VARCHAR(100) | NOT NULL | MIME 타입 |
| width | INT | NULL | 이미지 너비 (이미지인 경우) |
| height | INT | NULL | 이미지 높이 (이미지인 경우) |
| thumbnail_path | VARCHAR(500) | NULL | 썸네일 경로 (이미지인 경우) |
| download_count | INT | NOT NULL, DEFAULT 0 | 다운로드 횟수 |
| is_temp | BOOLEAN | NOT NULL, DEFAULT FALSE | 임시 파일 여부 |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 업로드일 |
| deleted_at | DATETIME | NULL | 삭제일 (Soft Delete) |

##### 3.5.2 인덱스

```sql
CREATE INDEX idx_files_post_id ON post_files(post_id);
CREATE INDEX idx_files_comment_id ON post_files(comment_id);
CREATE INDEX idx_files_member_id ON post_files(member_id);
CREATE INDEX idx_files_file_type ON post_files(file_type);
CREATE INDEX idx_files_is_temp ON post_files(is_temp);
```

##### 3.5.3 테이블 생성 SQL

```sql
CREATE TABLE post_files (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    post_id BIGINT UNSIGNED NULL,
    comment_id BIGINT UNSIGNED NULL,
    member_id BIGINT UNSIGNED NULL,
    file_type VARCHAR(20) NOT NULL COMMENT 'image, file',
    original_name VARCHAR(255) NOT NULL,
    stored_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    file_size BIGINT UNSIGNED NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    width INT NULL,
    height INT NULL,
    thumbnail_path VARCHAR(500) NULL,
    download_count INT NOT NULL DEFAULT 0,
    is_temp BOOLEAN NOT NULL DEFAULT FALSE,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at DATETIME NULL,
    PRIMARY KEY (id),
    INDEX idx_files_post_id (post_id),
    INDEX idx_files_comment_id (comment_id),
    INDEX idx_files_member_id (member_id),
    INDEX idx_files_file_type (file_type),
    INDEX idx_files_is_temp (is_temp),
    CONSTRAINT fk_files_post_id FOREIGN KEY (post_id) REFERENCES board_posts(id) ON DELETE CASCADE,
    CONSTRAINT fk_files_comment_id FOREIGN KEY (comment_id) REFERENCES post_comments(id) ON DELETE CASCADE,
    CONSTRAINT fk_files_member_id FOREIGN KEY (member_id) REFERENCES members(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

#### 3.6 스킨 테이블 (skins)

##### 3.6.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | 스킨 고유 ID |
| name | VARCHAR(100) | NOT NULL | 스킨 이름 |
| skin_key | VARCHAR(50) | UNIQUE, NOT NULL | 스킨 식별자 |
| description | TEXT | NULL | 스킨 설명 |
| version | VARCHAR(20) | NULL | 스킨 버전 |
| author | VARCHAR(100) | NULL | 제작자 |
| is_system | BOOLEAN | NOT NULL, DEFAULT FALSE | 시스템 기본 스킨 여부 |
| is_active | BOOLEAN | NOT NULL, DEFAULT TRUE | 활성화 여부 |
| config | JSON | NULL | 스킨 설정 (JSON) |
| created_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 생성일 |
| updated_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |

##### 3.6.2 인덱스

```sql
CREATE INDEX idx_skins_skin_key ON skins(skin_key);
CREATE INDEX idx_skins_is_active ON skins(is_active);
```

##### 3.6.3 테이블 생성 SQL

```sql
CREATE TABLE skins (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    skin_key TEXT NOT NULL UNIQUE,
    description TEXT,
    version TEXT,
    author TEXT,
    is_system INTEGER NOT NULL DEFAULT 0, -- 0: FALSE, 1: TRUE
    is_active INTEGER NOT NULL DEFAULT 1, -- 0: FALSE, 1: TRUE
    config TEXT, -- JSON 문자열로 저장
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- 인덱스 생성
CREATE INDEX idx_skins_skin_key ON skins(skin_key);
CREATE INDEX idx_skins_is_active ON skins(is_active);

-- updated_at 자동 업데이트 트리거
CREATE TRIGGER update_skins_timestamp 
AFTER UPDATE ON skins
BEGIN
    UPDATE skins SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

---

#### 3.7 세션 테이블 (member_sessions)

##### 3.7.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | VARCHAR(128) | PRIMARY KEY | 세션 ID |
| member_id | BIGINT UNSIGNED | NULL, FOREIGN KEY | 회원 ID |
| ip_address | VARCHAR(45) | NULL | IP 주소 |
| user_agent | VARCHAR(255) | NULL | User Agent |
| data | TEXT | NULL | 세션 데이터 |
| last_activity | INT | NOT NULL | 마지막 활동 시간 (Unix Timestamp) |

##### 3.7.2 인덱스

```sql
CREATE INDEX idx_sessions_member_id ON member_sessions(member_id);
CREATE INDEX idx_sessions_last_activity ON member_sessions(last_activity);
```

##### 3.7.3 테이블 생성 SQL

```sql
CREATE TABLE member_sessions (
    id TEXT PRIMARY KEY,
    member_id INTEGER,
    ip_address TEXT,
    user_agent TEXT,
    data TEXT,
    last_activity INTEGER NOT NULL,
    FOREIGN KEY (member_id) REFERENCES members(id) ON DELETE CASCADE
);

-- 인덱스 생성
CREATE INDEX idx_sessions_member_id ON member_sessions(member_id);
CREATE INDEX idx_sessions_last_activity ON member_sessions(last_activity);
```

---

#### 3.8 시스템 설정 테이블 (system_config)

##### 3.8.1 테이블 구조

| 컬럼명 | 타입 | 제약조건 | 설명 |
|--------|------|----------|------|
| id | BIGINT UNSIGNED | PRIMARY KEY, AUTO_INCREMENT | 설정 고유 ID |
| config_key | VARCHAR(100) | UNIQUE, NOT NULL | 설정 키 |
| config_value | TEXT | NULL | 설정 값 |
| config_type | VARCHAR(20) | NOT NULL, DEFAULT 'string' | 설정 타입 (string, int, bool, json) |
| description | VARCHAR(255) | NULL | 설명 |
| updated_at | DATETIME | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정일 |

##### 3.8.2 테이블 생성 SQL

```sql
CREATE TABLE system_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    config_key TEXT NOT NULL UNIQUE,
    config_value TEXT,
    config_type TEXT NOT NULL DEFAULT 'string',
    description TEXT,
    updated_at TEXT NOT NULL DEFAULT (datetime('now'))
);

-- updated_at 자동 업데이트 트리거
CREATE TRIGGER update_config_timestamp 
AFTER UPDATE ON system_config
BEGIN
    UPDATE system_config SET updated_at = datetime('now') WHERE id = NEW.id;
END;
```

---

### 4. 초기 데이터

#### 4.1 기본 관리자 계정 생성 (SQLite)

```sql
INSERT INTO members (email, password_hash, nickname, name, role, email_verified, status)
VALUES ('admin@oneboard.com', '$2y$10$...', '관리자', '시스템 관리자', 2, 1, 1);
-- 비밀번호는 bcryptjs로 해시 생성 필요
```

#### 4.2 기본 스킨 데이터 (SQLite)

```sql
INSERT INTO skins (name, skin_key, description, version, author, is_system, is_active)
VALUES 
    ('Basic Skin', 'basic', '기본 스킨', '1.0.0', 'One Board', 1, 1),
    ('Modern Skin', 'modern', '모던 스킨', '1.0.0', 'One Board', 1, 1),
    ('Classic Skin', 'classic', '클래식 스킨', '1.0.0', 'One Board', 1, 1);
```

#### 4.3 기본 게시판 생성 (SQLite)

```sql
INSERT INTO boards (name, description, board_key, read_permission, write_permission, comment_permission, display_order, is_active)
VALUES ('공지사항', '공지사항 게시판', 'notice', 0, 2, 1, 1, 1);
```

#### 4.4 기본 시스템 설정 (SQLite)

```sql
INSERT INTO system_config (config_key, config_value, config_type, description)
VALUES 
    ('site_name', 'One Board', 'string', '사이트 이름'),
    ('site_description', 'One Board 게시판 시스템', 'string', '사이트 설명'),
    ('default_skin', 'basic', 'string', '기본 스킨'),
    ('max_upload_size', '5242880', 'int', '최대 업로드 크기 (바이트)'),
    ('allowed_image_types', 'jpg,jpeg,png,gif,webp', 'string', '허용 이미지 타입');
```

---

### 5. 데이터베이스 마이그레이션 전략

#### 5.1 마이그레이션 파일 구조

```
/migrations/
├── 001_create_members_table.php
├── 002_create_boards_table.php
├── 003_create_posts_table.php
├── 004_create_comments_table.php
├── 005_create_files_table.php
├── 006_create_skins_table.php
├── 007_create_sessions_table.php
├── 008_create_config_table.php
└── 009_insert_initial_data.php
```

#### 5.2 마이그레이션 실행 순서
1. 기본 테이블 생성 (members, skins, boards)
2. 참조 테이블 생성 (board_posts, post_comments, post_files)
3. 세션 및 설정 테이블 생성
4. 초기 데이터 삽입

---

### 6. 데이터베이스 최적화

#### 6.1 파티셔닝 (선택사항)
- 대용량 데이터의 경우 `board_posts`, `post_comments` 테이블을 날짜별 파티셔닝 고려

#### 6.2 아카이빙 전략
- 삭제된 게시글/댓글은 `deleted_at`이 1년 이상 지난 경우 별도 아카이브 테이블로 이동

#### 6.3 백업 전략
- 일일 자동 백업 (전체 DB)
- 주간 증분 백업
- 백업 파일 암호화 저장

---

### 7. 데이터 무결성 규칙

#### 7.1 외래키 제약조건
- 모든 외래키는 CASCADE 또는 SET NULL 정책 적용
- 게시판 삭제 시 게시글 CASCADE 삭제
- 회원 삭제 시 게시글/댓글은 SET NULL (작성자 정보 보존)

#### 7.2 트리거 (선택사항)
- 게시글 작성 시 `boards.post_count` 자동 증가
- 댓글 작성 시 `board_posts.comment_count` 자동 증가
- 게시글 삭제 시 관련 파일 자동 삭제

---

### 8. 성능 고려사항

#### 8.1 쿼리 최적화 (SQLite)
- 자주 조회되는 컬럼에 인덱스 생성
- FTS5 가상 테이블 활용 (FULLTEXT 검색)
- JOIN 최소화, 필요한 경우만 사용
- WAL 모드 활성화로 읽기 성능 향상
- EXPLAIN QUERY PLAN으로 쿼리 최적화 확인

#### 8.2 캐싱 전략
- 게시판 목록 캐싱
- 인기 게시글 캐싱
- 스킨 설정 캐싱

---

### 9. 보안 고려사항

#### 9.1 데이터 암호화
- 비밀번호: bcrypt 해시
- 비회원 게시글 비밀번호: bcrypt 해시
- 민감 정보 암호화 저장 (선택)

#### 9.2 SQL Injection 방지
- 모든 쿼리는 Prepared Statements 사용
- 사용자 입력 값은 반드시 바인딩

#### 9.3 데이터 접근 제어
- SQLite는 파일 기반이므로 파일 시스템 권한으로 제어
- 읽기 전용 모드로 열기 (선택)
- 데이터베이스 파일 암호화 (SQLCipher 사용, 선택)

---

### 10. SQLite 특성 및 주의사항

#### 10.1 SQLite 특징
- 파일 기반 데이터베이스 (단일 .db 파일)
- 서버 없이 동작
- 트랜잭션 지원 (ACID 준수)
- 외래키 제약조건 지원 (PRAGMA foreign_keys = ON 필요)

#### 10.2 MySQL/MariaDB와의 주요 차이점

##### 10.2.1 데이터 타입
- MySQL의 `BIGINT UNSIGNED`, `TINYINT` → SQLite의 `INTEGER`
- MySQL의 `BOOLEAN` → SQLite의 `INTEGER` (0 또는 1)
- MySQL의 `DATETIME` → SQLite의 `TEXT` (ISO 8601) 또는 `INTEGER` (Unix timestamp)
- MySQL의 `VARCHAR(n)` → SQLite의 `TEXT` (길이 제한 없음, 애플리케이션 레벨에서 검증)

##### 10.2.2 자동 증가
- MySQL: `AUTO_INCREMENT`
- SQLite: `INTEGER PRIMARY KEY AUTOINCREMENT` 또는 `INTEGER PRIMARY KEY`

##### 10.2.3 타임스탬프 자동 업데이트
- MySQL: `ON UPDATE CURRENT_TIMESTAMP` (자동 지원)
- SQLite: 트리거로 구현 필요

##### 10.2.4 FULLTEXT 검색
- MySQL: `FULLTEXT INDEX`
- SQLite: FTS5 가상 테이블 사용

##### 10.2.5 JSON 타입
- MySQL: `JSON` 타입 (네이티브 지원)
- SQLite: `TEXT` 타입에 JSON 문자열 저장, 애플리케이션에서 파싱

#### 10.3 SQLite 설정 권장사항

```sql
-- 외래키 제약조건 활성화
PRAGMA foreign_keys = ON;

-- WAL 모드 활성화 (성능 향상)
PRAGMA journal_mode = WAL;

-- 동기화 모드 설정 (성능 vs 안정성 트레이드오프)
PRAGMA synchronous = NORMAL; -- 또는 FULL (더 안전)

-- 캐시 크기 설정 (메모리 허용 범위 내에서)
PRAGMA cache_size = -64000; -- 64MB (음수는 KB 단위)
```

#### 10.4 성능 최적화 팁
- WAL 모드 사용으로 읽기 성능 향상
- 적절한 인덱스 생성
- 트랜잭션 사용 (여러 INSERT/UPDATE를 하나의 트랜잭션으로)
- VACUUM 주기적 실행 (파편화 제거)

#### 10.5 백업 전략 (SQLite)
- 파일 복사 방식 백업 (단순하지만 안전)
- `.backup` 명령 사용
- 주기적 백업 스크립트 작성
- 백업 파일 암호화 (선택)


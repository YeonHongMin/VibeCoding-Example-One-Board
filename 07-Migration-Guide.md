# Migration Guide (One Board 구현 가이드)
## One Board 구현 가이드

### 1. 개요

#### 1.1 목적
이 문서는 Next.js + SQLite 기반의 One Board 구현을 위한 상세 가이드를 제공합니다.

#### 1.2 주요 기능 분석

One Board는 다음과 같은 핵심 기능들을 제공합니다:

1. **게시판 시스템**
   - 다중 게시판 생성 및 관리
   - 게시판별 권한 설정
   - 게시판 그룹 관리
   - 게시판 스킨 설정

2. **회원 시스템**
   - 회원 가입/로그인/로그아웃
   - 회원 등급 관리
   - 포인트 시스템
   - 회원 정보 관리

3. **게시글 시스템**
   - 게시글 작성/수정/삭제
   - 파일 첨부
   - 이미지 업로드
   - 게시글 검색
   - 게시글 추천/비추천

4. **댓글 시스템**
   - 댓글 작성/수정/삭제
   - 대댓글 (댓글의 댓글)
   - 댓글 추천

5. **파일 관리**
   - 파일 업로드/다운로드
   - 이미지 리사이징
   - 썸네일 생성

6. **권한 관리**
   - 게시판별 읽기/쓰기 권한
   - 회원 등급별 권한
   - 관리자 권한

7. **스킨 시스템**
   - 게시판별 스킨 적용
   - 테마 관리

8. **관리자 기능**
   - 게시판 관리
   - 회원 관리
   - 환경 설정
   - 데이터 백업

---

### 2. 데이터베이스 구조 분석

#### 2.1 주요 테이블

One Board는 다음과 같은 주요 테이블을 사용합니다:

```
members            # 회원 테이블
boards             # 게시판 테이블
board_posts        # 게시글 테이블
post_files         # 파일 테이블
post_comments      # 댓글 테이블
system_config      # 환경 설정 테이블
skins              # 스킨 테이블
```

#### 2.2 테이블 구조

| 테이블명 | 설명 |
|---------|------|
| members | 회원 정보 |
| boards | 게시판 정보 |
| board_posts | 게시글 |
| post_files | 파일 정보 |
| post_comments | 댓글 |
| system_config | 시스템 설정 |
| skins | 스킨 정보 |

---

### 3. 기능별 포팅 계획

#### 3.1 게시판 시스템 포팅

##### 3.1.1 게시판 구조 분석

One Board의 게시판은 다음과 같은 특징이 있습니다:
- 게시판별 동적 테이블 생성 (`g5_write_{board_id}`)
- 게시판별 설정 파일 (`board/{board_id}/config.php`)
- 게시판별 스킨 적용

##### 3.1.2 One Board 포팅 전략

**변경 사항:**
- 동적 테이블 대신 단일 `board_posts` 테이블 사용
- `board_id`로 게시판 구분
- 게시판 설정은 `boards` 테이블에 JSON으로 저장

**구현 파일:**
```
lib/db/boards.ts          # 게시판 데이터베이스 함수
app/api/boards/route.ts   # 게시판 API
components/board/         # 게시판 컴포넌트
```

##### 3.1.3 포팅 체크리스트

- [ ] 게시판 생성 로직 포팅
- [ ] 게시판 설정 관리 포팅
- [ ] 게시판 목록 조회 포팅
- [ ] 게시판 권한 체크 로직 포팅
- [ ] 게시판 그룹 관리 포팅

---

#### 3.2 회원 시스템 포팅

##### 3.2.1 회원 시스템 분석

One Board의 회원 시스템 특징:
- 회원 등급 시스템 (일반회원, 정회원, 우수회원 등)
- 포인트 시스템
- 회원 그룹 관리
- 회원 정보 확장 필드

##### 3.2.2 One Board 포팅 전략

**변경 사항:**
- 회원 등급을 `role` 필드로 단순화 (0:비회원, 1:일반회원, 2:관리자)
- 포인트 시스템은 Phase 2에서 추가
- 회원 정보 확장은 JSON 필드로 저장

**구현 파일:**
```
lib/db/members.ts         # 회원 데이터베이스 함수
app/api/auth/             # 인증 API
lib/auth/                 # 인증 유틸리티
components/auth/          # 인증 컴포넌트
```

##### 3.2.3 포팅 체크리스트

- [ ] 회원 가입 로직 포팅
- [ ] 로그인/로그아웃 로직 포팅
- [ ] 비밀번호 암호화 로직 포팅 (bcrypt)
- [ ] 회원 정보 수정 로직 포팅
- [ ] 회원 등급 관리 포팅
- [ ] 세션 관리 포팅

---

#### 3.3 게시글 시스템 포팅

##### 3.3.1 게시글 시스템 분석

One Board의 게시글 시스템 특징:
- 게시판별 동적 테이블 사용
- 파일 첨부 기능
- 이미지 업로드 및 리사이징
- 게시글 추천/비추천
- 게시글 스크랩
- 게시글 검색 (FULLTEXT)

##### 3.3.2 One Board 포팅 전략

**변경 사항:**
- 단일 `board_posts` 테이블 사용
- 이미지 Copy & Paste 기능 추가
- 추천 기능은 `like_count` 필드로 단순화
- 검색은 SQLite FTS5 사용

**구현 파일:**
```
lib/db/posts.ts           # 게시글 데이터베이스 함수
app/api/posts/            # 게시글 API
components/post/          # 게시글 컴포넌트
lib/upload/               # 파일 업로드 유틸리티
```

##### 3.3.3 포팅 체크리스트

- [ ] 게시글 작성 로직 포팅
- [ ] 게시글 수정/삭제 로직 포팅
- [ ] 파일 첨부 로직 포팅
- [ ] 이미지 업로드 및 리사이징 포팅
- [ ] 게시글 검색 로직 포팅 (FTS5)
- [ ] 게시글 추천 로직 포팅
- [ ] 조회수 증가 로직 포팅

---

#### 3.4 댓글 시스템 포팅

##### 3.4.1 댓글 시스템 분석

One Board의 댓글 시스템 특징:
- 단일 댓글 테이블 (`g5_comment`)
- 게시글 ID로 연결
- 댓글 추천 기능
- 댓글 신고 기능

##### 3.4.2 One Board 포팅 전략

**변경 사항:**
- 대댓글 지원 추가 (`parent_id` 필드)
- 댓글 추천은 `like_count` 필드로 단순화
- 댓글 신고 기능은 Phase 2에서 추가

**구현 파일:**
```
lib/db/comments.ts        # 댓글 데이터베이스 함수
app/api/comments/         # 댓글 API
components/comment/       # 댓글 컴포넌트
```

##### 3.4.3 포팅 체크리스트

- [ ] 댓글 작성 로직 포팅
- [ ] 댓글 수정/삭제 로직 포팅
- [ ] 대댓글 로직 구현
- [ ] 댓글 목록 조회 로직 포팅
- [ ] 댓글 추천 로직 포팅

---

#### 3.5 파일 관리 시스템 포팅

##### 3.5.1 파일 관리 분석

One Board의 파일 관리 특징:
- 파일 업로드 디렉토리 구조 (`data/file/{board_id}/`)
- 파일명 난수화
- 이미지 리사이징 (썸네일 생성)
- 파일 다운로드 권한 체크

##### 3.5.2 One Board 포팅 전략

**변경 사항:**
- 파일 저장 경로: `/uploads/images/YYYY/MM/DD/` 형식
- Sharp 라이브러리로 이미지 처리
- Next.js API Route로 파일 서빙

**구현 파일:**
```
lib/upload/fileUploader.ts   # 파일 업로드 유틸리티
lib/image/imageProcessor.ts   # 이미지 처리 유틸리티
app/api/upload/               # 파일 업로드 API
app/api/files/[id]/route.ts   # 파일 다운로드 API
```

##### 3.5.3 포팅 체크리스트

- [ ] 파일 업로드 로직 포팅
- [ ] 파일 검증 로직 포팅
- [ ] 이미지 리사이징 로직 포팅 (Sharp)
- [ ] 썸네일 생성 로직 포팅
- [ ] 파일 다운로드 로직 포팅
- [ ] 파일 삭제 로직 포팅

---

#### 3.6 권한 관리 시스템 포팅

##### 3.6.1 권한 관리 분석

One Board의 권한 관리 특징:
- 게시판별 읽기/쓰기 권한 설정
- 회원 등급별 권한
- 관리자 권한 분리 (최고관리자, 관리자 등)

##### 3.6.2 One Board 포팅 전략

**변경 사항:**
- 권한 레벨 단순화 (0:비회원, 1:회원, 2:관리자)
- 게시판별 권한 설정 유지
- Next.js Middleware로 라우트 보호

**구현 파일:**
```
lib/auth/permissions.ts       # 권한 체크 함수
middleware.ts                  # Next.js 미들웨어
lib/db/boards.ts              # 게시판 권한 체크
```

##### 3.6.3 포팅 체크리스트

- [ ] 게시판 권한 체크 로직 포팅
- [ ] 회원 권한 체크 로직 포팅
- [ ] 관리자 권한 체크 로직 포팅
- [ ] 미들웨어 구현
- [ ] API Route 권한 체크 포팅

---

#### 3.7 스킨 시스템 포팅

##### 3.7.1 스킨 시스템 분석

One Board의 스킨 시스템 특징:
- 게시판별 스킨 적용
- 스킨 디렉토리 구조 (`skin/{skin_name}/`)
- 스킨 설정 파일

##### 3.7.2 One Board 포팅 전략

**변경 사항:**
- React 컴포넌트 기반 스킨 시스템
- 스킨 디렉토리: `skins/{skin_name}/components/`
- 스킨 설정은 데이터베이스에 저장

**구현 파일:**
```
skins/basic/                  # 기본 스킨
lib/skin/skinLoader.ts        # 스킨 로더
app/api/skins/                # 스킨 API
```

##### 3.7.3 포팅 체크리스트

- [ ] 스킨 로더 시스템 구현
- [ ] 기본 스킨 포팅
- [ ] 게시판별 스킨 적용 로직 포팅
- [ ] 스킨 설정 관리 포팅

---

#### 3.8 관리자 기능 포팅

##### 3.8.1 관리자 기능 분석

One Board의 관리자 기능:
- 게시판 관리
- 회원 관리
- 환경 설정
- 데이터 백업/복원
- 통계 조회

##### 3.8.2 One Board 포팅 전략

**변경 사항:**
- Next.js App Router로 관리자 페이지 구성
- SQLite 백업 기능 구현
- 통계는 실시간 조회

**구현 파일:**
```
app/admin/                    # 관리자 페이지
app/api/admin/                # 관리자 API
lib/admin/                    # 관리자 유틸리티
lib/backup/                   # 백업 기능
```

##### 3.8.3 포팅 체크리스트

- [ ] 관리자 대시보드 포팅
- [ ] 게시판 관리 페이지 포팅
- [ ] 회원 관리 페이지 포팅
- [ ] 환경 설정 페이지 포팅
- [ ] 데이터 백업 기능 포팅
- [ ] 통계 조회 기능 포팅

---

### 4. 핵심 함수 구현

#### 4.1 공통 함수 포팅

One Board의 주요 공통 함수들:

##### 4.1.1 문자열 처리 함수

```php
// 참고 예시
function get_text($str) { ... }
function cut_str($str, $len) { ... }
function clean_xss_tags($str) { ... }
```

```typescript
// One Board
// lib/utils/string.ts
export function getText(str: string): string { ... }
export function cutStr(str: string, len: number): string { ... }
export function cleanXssTags(str: string): string { ... }
```

##### 4.1.2 날짜 처리 함수

```php
// 참고 예시
function date($format, $timestamp) { ... }
function datetime($datetime) { ... }
```

```typescript
// One Board
// lib/utils/date.ts
export function formatDate(date: Date, format: string): string { ... }
export function formatDateTime(date: Date): string { ... }
```

##### 4.1.3 파일 처리 함수

```php
// 참고 예시
function get_file_icon($filename) { ... }
function get_file_size($filename) { ... }
```

```typescript
// One Board
// lib/utils/file.ts
export function getFileIcon(filename: string): string { ... }
export function getFileSize(bytes: number): string { ... }
```

---

#### 4.2 데이터베이스 함수 포팅

##### 4.2.1 DB 함수

```php
// 참고 예시
function sql_query($sql) { ... }
function sql_fetch_array($result) { ... }
function sql_num_rows($result) { ... }
```

##### 4.2.2 One Board로 포팅

```typescript
// One Board
// Drizzle ORM 사용
import { db } from '@/lib/db';
import { posts } from '@/lib/db/schema';

// 쿼리 예시
const result = await db.select().from(posts).where(eq(posts.id, postId));
```

---

#### 4.3 세션/인증 함수 포팅

##### 4.3.1 세션 함수

```php
// 참고 예시
function set_session($key, $value) { ... }
function get_session($key) { ... }
function is_member() { ... }
function is_admin() { ... }
```

##### 4.3.2 One Board로 포팅

```typescript
// One Board
// lib/auth/session.ts
import { cookies } from 'next/headers';

export async function setSession(key: string, value: string) { ... }
export async function getSession(key: string) { ... }
export async function isMember() { ... }
export async function isAdmin() { ... }
```

---

### 5. 단계별 포팅 계획

#### Phase 1: 핵심 기능 포팅 (MVP)

**우선순위:**
1. 회원 시스템 (가입, 로그인, 로그아웃)
2. 게시판 기본 기능 (생성, 목록 조회)
3. 게시글 기본 기능 (작성, 조회, 수정, 삭제)
4. 파일 업로드 (이미지 Copy & Paste 포함)
5. 기본 관리자 페이지

**예상 기간:** 4-6주

---

#### Phase 2: 확장 기능 포팅

**우선순위:**
1. 댓글 시스템 (대댓글 포함)
2. 게시판 권한 관리
3. 게시글 검색 (FTS5)
4. 스킨 시스템
5. 회원 관리 페이지

**예상 기간:** 3-4주

---

#### Phase 3: 고급 기능 포팅

**우선순위:**
1. 포인트 시스템
2. 게시글 추천/스크랩
3. 알림 시스템
4. 데이터 백업/복원
5. 통계 및 분석

**예상 기간:** 2-3주

---

### 6. 포팅 시 주의사항

#### 6.1 보안
- Next.js/React의 보안 모범 사례 적용
- SQL Injection 방지: Drizzle ORM 사용
- XSS 방지: React의 자동 이스케이프 + DOMPurify
- CSRF 방지: Next.js 내장 보호

#### 6.2 성능
- JWT + HTTP-only Cookie 사용
- SQLite는 파일 기반이므로 동시 접속자 수 제한 고려
- 이미지 최적화: Next.js Image 컴포넌트 사용
- 서버 컴포넌트 적극 활용

#### 6.3 호환성
- 데이터 마이그레이션 기능은 별도 구현 필요
- 플러그인 시스템은 재설계 필요

---

### 7. 코드 분석 가이드

#### 7.1 분석해야 할 주요 파일

**게시판 관련:**
- `bbs/board.php` - 게시판 목록
- `bbs/write.php` - 게시글 작성
- `bbs/view.php` - 게시글 조회
- `bbs/list.php` - 게시글 목록

**회원 관련:**
- `member/register.php` - 회원 가입
- `member/login.php` - 로그인
- `lib/common.lib.php` - 공통 함수

**관리자 관련:**
- `adm/board_list.php` - 게시판 관리
- `adm/member_list.php` - 회원 관리
- `adm/config_form.php` - 환경 설정

#### 7.2 분석 방법

1. **기능별로 파일 그룹화**
   - 게시판 기능 관련 파일들
   - 회원 기능 관련 파일들
   - 관리자 기능 관련 파일들

2. **로직 추출**
   - 각 파일에서 핵심 로직 파악
   - 데이터베이스 쿼리 분석
   - 비즈니스 로직 분석

3. **Next.js로 재구현**
   - API Route로 서버 로직 구현
   - React 컴포넌트로 UI 구현
   - Drizzle ORM으로 데이터베이스 접근

---

### 8. 테스트 전략

#### 8.1 기능 테스트
- 각 기능이 One Board에서 올바르게 동작하는지 테스트
- 회원 가입/로그인 테스트
- 게시글 작성/조회 테스트
- 파일 업로드 테스트

#### 8.2 통합 테스트
- 전체 플로우 테스트
- 권한 체크 테스트
- 에러 처리 테스트

#### 8.3 성능 테스트
- 페이지 로딩 속도 측정
- 동시 접속자 테스트
- 데이터베이스 쿼리 성능 테스트

---

### 9. 참고 자료

- Next.js 공식 문서
- React 공식 문서
- Next.js 공식 문서: https://nextjs.org/docs
- Drizzle ORM 문서: https://orm.drizzle.team/
- SQLite 문서: https://www.sqlite.org/docs.html

---

### 10. 다음 단계

이 가이드를 바탕으로 다음 작업을 진행하세요:

1. One Board 코드베이스 분석
2. Phase 1 기능부터 순차적으로 포팅
3. 각 기능 포팅 후 테스트 수행
4. 문서화 및 코드 리뷰

각 기능 포팅 시 해당 섹션의 체크리스트를 참고하여 진행하세요.


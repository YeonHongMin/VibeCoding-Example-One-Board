# TASKS (AI 개발 파트너용 프롬프트 설계서)
## One Board AI 개발 파트너 작업 가이드

### 1. 문서 개요

#### 1.1 목적
이 문서는 AI 개발 파트너(예: Cursor, GitHub Copilot 등)가 One Board 프로젝트를 효율적으로 개발할 수 있도록 작업 단위와 프롬프트를 체계적으로 정리한 가이드입니다.

#### 1.2 사용 방법
- 각 작업은 독립적으로 실행 가능하도록 설계
- 작업 시작 전 관련 문서(PRD, TRD, Database Design 등) 참고 필수
- 작업 완료 후 코드 리뷰 및 테스트 수행

---

### 2. 프로젝트 초기 설정 작업

#### 2.1 작업: Next.js 프로젝트 초기 설정

**프롬프트:**
```
One Board 프로젝트를 Next.js 14 (App Router)로 초기 설정해주세요.

요구사항:
1. Next.js 14 프로젝트 생성 (TypeScript 사용)
2. TRD 문서의 디렉토리 구조를 참고하여 생성
3. 다음 디렉토리 구조 포함:
   - app/ (Next.js App Router)
     - (auth)/ (인증 관련 라우트)
     - (main)/ (메인 콘텐츠 라우트)
     - admin/ (관리자 페이지)
     - api/ (API Routes)
   - components/ (React 컴포넌트)
   - lib/ (유틸리티 및 라이브러리)
     - db/ (데이터베이스 설정)
   - hooks/ (Custom React Hooks)
   - types/ (TypeScript 타입 정의)
   - styles/ (전역 스타일)
   - public/ (정적 파일)
   - uploads/ (업로드 파일, public 외부)
   - skins/ (스킨 디렉토리)

4. 필수 패키지 설치:
   - drizzle-orm, drizzle-kit (또는 prisma)
   - better-sqlite3 (SQLite 드라이버)
   - bcryptjs (비밀번호 해시)
   - zod (입력 검증)
   - react-hook-form (폼 관리)

5. 기본 설정 파일 생성:
   - package.json
   - tsconfig.json
   - next.config.js
   - .env.local (템플릿)
   - .gitignore
```

**예상 산출물:**
- Next.js 프로젝트 구조
- package.json 및 기본 설정 파일들

---

#### 2.2 작업: SQLite 데이터베이스 설정 및 스키마 생성

**프롬프트:**
```
SQLite 데이터베이스 설정 및 Drizzle ORM 스키마를 생성해주세요.

요구사항:
1. lib/db/index.ts 파일 생성 (데이터베이스 연결)
2. lib/db/schema.ts 파일 생성 (Drizzle 스키마 정의)
3. Database Design 문서의 모든 테이블을 Drizzle 스키마로 정의:
   - members
   - boards
   - board_posts
   - post_comments
   - post_files
   - skins
   - member_sessions
   - system_config
4. drizzle.config.ts 파일 생성 (마이그레이션 설정)
5. 환경 변수에서 데이터베이스 경로 읽기 (.env.local)
6. WAL 모드 활성화
7. 외래키 제약조건 활성화

참고: Database Design 문서의 테이블 구조 참고
```

**예상 산출물:**
- `lib/db/index.ts` (데이터베이스 연결)
- `lib/db/schema.ts` (Drizzle 스키마)
- `drizzle.config.ts` (마이그레이션 설정)
- 초기 마이그레이션 파일

---

#### 2.3 작업: 환경 변수 및 설정 파일 생성

**프롬프트:**
```
One Board의 환경 변수 및 설정 파일을 생성해주세요.

요구사항:
1. .env.local 파일 생성 (템플릿)
2. .env.example 파일 생성 (예시)
3. 다음 환경 변수 포함:
   - DATABASE_URL (SQLite 파일 경로)
   - NEXTAUTH_SECRET (인증 시크릿)
   - NEXTAUTH_URL (사이트 URL)
   - UPLOAD_MAX_SIZE (최대 업로드 크기)
   - ALLOWED_FILE_TYPES (허용 파일 타입)
4. lib/config.ts 파일 생성 (설정 값 타입 안전하게 관리)
5. 환경 변수 검증 로직 (Zod 사용)
6. 개발/프로덕션 환경 구분
```

**예상 산출물:**
- `.env.local` (템플릿)
- `.env.example`
- `lib/config.ts` (설정 관리)

---

### 3. 인증 및 회원 관리 작업

#### 3.1 작업: 회원 관련 데이터베이스 함수 생성

**프롬프트:**
```
회원(Member) 관련 데이터베이스 함수를 생성해주세요.

요구사항:
1. lib/db/members.ts 파일 생성
2. Database Design 문서의 members 테이블 구조 참고
3. Drizzle ORM을 사용하여 다음 함수 구현:
   - createMember(): 회원 가입
   - findMemberByEmail(): 이메일로 회원 조회
   - findMemberByNickname(): 닉네임으로 회원 조회
   - updateMember(): 회원 정보 수정
   - updateMemberPassword(): 비밀번호 변경
   - deleteMember(): 회원 탈퇴 (Soft Delete)
   - verifyPassword(): 비밀번호 확인
   - incrementLoginFailCount(): 로그인 실패 횟수 증가
   - lockAccount(): 계정 잠금
   - unlockAccount(): 계정 잠금 해제
4. 비밀번호는 bcryptjs 사용하여 해시 저장
5. 입력 검증 포함 (Zod 스키마)
6. TypeScript 타입 정의 포함

참고: Drizzle ORM 문서 및 Database Design 문서 참고
```

**예상 산출물:**
- `lib/db/members.ts`
- 회원 관련 모든 CRUD 함수
- TypeScript 타입 정의

---

#### 3.2 작업: 인증 API Routes 및 Server Actions 생성

**프롬프트:**
```
회원 가입, 로그인, 로그아웃을 처리하는 API Routes와 Server Actions를 생성해주세요.

요구사항:
1. app/api/auth/register/route.ts (회원 가입 API)
2. app/api/auth/login/route.ts (로그인 API)
3. app/api/auth/logout/route.ts (로그아웃 API)
4. app/api/auth/check-email/route.ts (이메일 중복 체크 API)
5. app/api/auth/check-nickname/route.ts (닉네임 중복 체크 API)
6. app/actions/auth.ts (Server Actions, 선택사항)
7. 다음 기능 구현:
   - 입력 검증 (Zod 스키마)
   - 비밀번호 해시 (bcryptjs)
   - JWT 토큰 생성 또는 세션 관리
   - HTTP-only Cookie 설정
   - 로그인 실패 시 계정 잠금 처리
8. 에러 처리 및 응답 형식 통일
9. User Flow 문서의 로그인/가입 플로우 참고

참고: Next.js Route Handlers 및 Server Actions 문서 참고
```

**예상 산출물:**
- 인증 관련 API Routes
- Server Actions (선택)
- 인증 관련 유틸리티 함수

---

#### 3.3 작업: 회원 가입/로그인 페이지 생성

**프롬프트:**
```
회원 가입 및 로그인 페이지를 Next.js로 생성해주세요.

요구사항:
1. app/(auth)/register/page.tsx (회원가입 페이지)
2. app/(auth)/login/page.tsx (로그인 페이지)
3. components/auth/RegisterForm.tsx (회원가입 폼 컴포넌트)
4. components/auth/LoginForm.tsx (로그인 폼 컴포넌트)
5. Design System 문서의 디자인 가이드 준수
6. Tailwind CSS 또는 CSS Modules 사용
7. React Hook Form 사용 (폼 관리)
8. Zod를 사용한 클라이언트 사이드 검증
9. 실시간 중복 체크 (useEffect + fetch API)
10. 접근성 고려 (ARIA 레이블, 키보드 네비게이션)
11. 에러 메시지 표시 (react-hot-toast 또는 유사 라이브러리)
12. 반응형 디자인 적용

참고: Next.js App Router 및 React Hook Form 문서 참고
```

**예상 산출물:**
- 회원가입 페이지 (Server Component + Client Component)
- 로그인 페이지 (Server Component + Client Component)
- 폼 컴포넌트들
- 관련 스타일 파일

---

### 4. 게시판 관리 작업

#### 4.1 작업: 게시판 모델 생성

**프롬프트:**
```
게시판(Board) 모델 클래스를 생성해주세요.

요구사항:
1. models/Board.php 파일 생성
2. Database Design 문서의 boards 테이블 구조 참고
3. 다음 메서드 구현:
   - create(): 게시판 생성
   - findAll(): 모든 게시판 조회 (활성화된 것만)
   - findById(): ID로 게시판 조회
   - findByKey(): board_key로 게시판 조회
   - update(): 게시판 정보 수정
   - delete(): 게시판 삭제
   - updatePostCount(): 게시글 수 업데이트
   - checkPermission(): 권한 체크 (읽기/쓰기/댓글)
4. 게시판 순서 관리 메서드
5. 카테고리별 조회 메서드
6. PDO Prepared Statements 사용
```

**예상 산출물:**
- `models/Board.php`
- 게시판 관련 모든 CRUD 메서드

---

#### 4.2 작업: 게시판 컨트롤러 생성

**프롬프트:**
```
게시판 목록 조회를 처리하는 BoardController를 생성해주세요.

요구사항:
1. controllers/BoardController.php 파일 생성
2. 다음 메서드 구현:
   - index(): 게시판 목록 표시
   - show(): 특정 게시판의 게시글 목록 표시
3. 권한 체크 포함
4. 페이징 처리
5. 정렬 옵션 (최신순, 조회수순, 댓글순)
6. 검색 기능
7. User Flow 문서의 게시판 조회 플로우 참고
```

**예상 산출물:**
- `controllers/BoardController.php`
- 게시판 목록 로직

---

### 5. 게시글 관리 작업

#### 5.1 작업: 게시글 모델 생성

**프롬프트:**
```
게시글(Post) 모델 클래스를 생성해주세요.

요구사항:
1. models/Post.php 파일 생성
2. Database Design 문서의 board_posts 테이블 구조 참고
3. 다음 메서드 구현:
   - create(): 게시글 작성
   - findById(): ID로 게시글 조회
   - findByBoardId(): 게시판별 게시글 목록 조회
   - update(): 게시글 수정
   - delete(): 게시글 삭제 (Soft Delete)
   - incrementViewCount(): 조회수 증가
   - incrementLikeCount(): 좋아요 수 증가
   - updateCommentCount(): 댓글 수 업데이트
   - search(): 검색 기능 (FULLTEXT 인덱스 활용)
4. 페이징 지원
5. 정렬 옵션 지원
6. 권한 체크 메서드
7. PDO Prepared Statements 사용
```

**예상 산출물:**
- `models/Post.php`
- 게시글 관련 모든 CRUD 메서드

---

#### 5.2 작업: 게시글 작성/수정 컨트롤러 생성

**프롬프트:**
```
게시글 작성, 수정, 삭제를 처리하는 PostController를 생성해주세요.

요구사항:
1. controllers/PostController.php 파일 생성
2. 다음 메서드 구현:
   - create(): 게시글 작성 폼 표시
   - store(): 게시글 저장 처리
   - show(): 게시글 상세 조회
   - edit(): 게시글 수정 폼 표시
   - update(): 게시글 수정 처리
   - delete(): 게시글 삭제 처리
3. 권한 체크 (작성자 본인 또는 관리자만 수정/삭제 가능)
4. 입력 검증 및 보안 처리
5. 조회수 증가 처리
6. 이전글/다음글 조회
7. User Flow 문서의 게시글 작성/수정 플로우 참고
```

**예상 산출물:**
- `controllers/PostController.php`
- 게시글 관련 모든 로직

---

#### 5.3 작업: WYSIWYG 에디터 통합 및 이미지 Copy & Paste 기능

**프롬프트:**
```
게시글 작성 페이지에 WYSIWYG 에디터를 통합하고, 이미지 Copy & Paste 기능을 구현해주세요.

요구사항:
1. TinyMCE, Quill, 또는 Tiptap 에디터 통합 (React 컴포넌트)
2. 이미지 Copy & Paste 기능 구현:
   - 클립보드에서 이미지 붙여넣기 시 자동 업로드
   - 드래그 앤 드롭으로 이미지 업로드
   - 파일 선택으로 이미지 업로드
3. 클라이언트 사이드 (React/TypeScript):
   - components/editor/Editor.tsx (에디터 컴포넌트)
   - paste 이벤트 리스너 (useEffect)
   - drop 이벤트 리스너
   - 이미지 미리보기 표시
   - 업로드 진행률 표시 (로딩 상태)
4. 서버 사이드 (Next.js API Route):
   - app/api/upload/image/route.ts 생성
   - 이미지 검증 (Zod 스키마, MIME 타입, 확장자, 크기)
   - Sharp 라이브러리로 이미지 리사이징 (최대 너비 1920px)
   - 파일 저장 (/uploads/images/YYYY/MM/DD/ 형식)
   - 파일명 난수화
   - DB 기록 (post_files 테이블, Drizzle ORM 사용)
5. fetch API를 사용한 비동기 업로드
6. 에러 처리 및 사용자 피드백 (toast 메시지)
7. TRD 문서의 이미지 Copy & Paste 구현 섹션 참고

참고: Next.js API Routes, Sharp 라이브러리 문서 참고
```

**예상 산출물:**
- 에디터 React 컴포넌트
- 이미지 업로드 API Route
- 이미지 처리 유틸리티 함수 (lib/upload/)
- 클라이언트 사이드 훅 (hooks/useImageUpload.ts)

---

#### 5.4 작업: 게시글 작성/수정/상세 뷰 생성

**프롬프트:**
```
게시글 작성, 수정, 상세 조회 페이지를 생성해주세요.

요구사항:
1. views/post/create.php (작성 페이지)
2. views/post/edit.php (수정 페이지)
3. views/post/show.php (상세 페이지)
4. Design System 문서의 디자인 가이드 준수
5. WYSIWYG 에디터 포함
6. 이미지 Copy & Paste 기능 포함
7. 파일 첨부 기능 (게시판 설정에 따라)
8. 태그 입력 기능
9. 반응형 디자인
10. 접근성 고려
11. 이전글/다음글 네비게이션 (상세 페이지)
```

**예상 산출물:**
- 게시글 작성 페이지
- 게시글 수정 페이지
- 게시글 상세 페이지
- 관련 CSS/JS 파일

---

### 6. 파일 관리 작업

#### 6.1 작업: 파일 모델 생성

**프롬프트:**
```
파일(File) 모델 클래스를 생성해주세요.

요구사항:
1. models/File.php 파일 생성
2. Database Design 문서의 post_files 테이블 구조 참고
3. 다음 메서드 구현:
   - create(): 파일 정보 DB 저장
   - findById(): ID로 파일 조회
   - findByPostId(): 게시글별 파일 조회
   - delete(): 파일 삭제 (파일 시스템 + DB)
   - incrementDownloadCount(): 다운로드 횟수 증가
4. 임시 파일 관리 메서드
5. 파일 타입별 처리 (이미지/일반 파일)
```

**예상 산출물:**
- `models/File.php`
- 파일 관련 모든 메서드

---

#### 6.2 작업: 파일 업로드 라이브러리 생성

**프롬프트:**
```
파일 업로드를 처리하는 라이브러리를 생성해주세요.

요구사항:
1. lib/upload/FileUploader.php 파일 생성
2. 다음 기능 구현:
   - 이미지 업로드 및 리사이징
   - 일반 파일 업로드
   - 파일 검증 (타입, 크기, 확장자)
   - 파일명 난수화
   - 저장 경로 생성 (/uploads/images/YYYY/MM/DD/)
   - 썸네일 생성 (이미지인 경우)
3. 보안 처리:
   - MIME 타입 검증
   - 파일 확장자 화이트리스트
   - 실행 파일 업로드 방지
   - 파일 크기 제한
4. 에러 처리 및 예외 처리
5. TRD 문서의 파일 업로드 시스템 섹션 참고
```

**예상 산출물:**
- `lib/upload/FileUploader.php`
- 이미지 처리 클래스
- 파일 검증 로직

---

### 7. 댓글 시스템 작업

#### 7.1 작업: 댓글 모델 생성

**프롬프트:**
```
댓글(Comment) 모델 클래스를 생성해주세요.

요구사항:
1. models/Comment.php 파일 생성
2. Database Design 문서의 post_comments 테이블 구조 참고
3. 다음 메서드 구현:
   - create(): 댓글 작성
   - findByPostId(): 게시글별 댓글 조회 (계층 구조)
   - findById(): ID로 댓글 조회
   - update(): 댓글 수정
   - delete(): 댓글 삭제 (Soft Delete)
   - incrementLikeCount(): 좋아요 수 증가
4. 대댓글 지원 (parent_id 활용)
5. 계층 구조 조회 메서드
6. PDO Prepared Statements 사용
```

**예상 산출물:**
- `models/Comment.php`
- 댓글 관련 모든 CRUD 메서드

---

#### 7.2 작업: 댓글 컨트롤러 생성

**프롬프트:**
```
댓글 작성, 수정, 삭제를 처리하는 CommentController를 생성해주세요.

요구사항:
1. controllers/CommentController.php 파일 생성
2. 다음 메서드 구현:
   - store(): 댓글 작성 처리
   - update(): 댓글 수정 처리
   - delete(): 댓글 삭제 처리
   - like(): 댓글 좋아요 처리
3. 권한 체크
4. 입력 검증 및 보안 처리
5. AJAX 지원
6. 게시글의 댓글 수 업데이트
```

**예상 산출물:**
- `controllers/CommentController.php`
- 댓글 관련 모든 로직

---

### 8. 관리자 페이지 작업

#### 8.1 작업: 관리자 대시보드 생성

**프롬프트:**
```
관리자 대시보드를 생성해주세요.

요구사항:
1. admin/dashboard.php 파일 생성
2. 다음 통계 정보 표시:
   - 총 회원 수
   - 총 게시판 수
   - 총 게시글 수
   - 총 댓글 수
   - 오늘 방문자 수
   - 최근 활동 내역
3. 차트 또는 그래프로 시각화 (선택)
4. 관리자 권한 체크
5. Design System 문서의 디자인 가이드 준수
```

**예상 산출물:**
- 관리자 대시보드 페이지
- 통계 조회 로직

---

#### 8.2 작업: 게시판 관리 페이지 생성

**프롬프트:**
```
관리자 페이지에서 게시판을 관리하는 기능을 구현해주세요.

요구사항:
1. admin/boards/index.php (게시판 목록)
2. admin/boards/create.php (게시판 생성)
3. admin/boards/edit.php (게시판 수정)
4. 다음 기능 구현:
   - 게시판 생성/수정/삭제
   - 게시판 순서 변경 (드래그 앤 드롭)
   - 게시판 활성화/비활성화
   - 게시판 설정 변경
5. 권한 설정 UI
6. 파일 업로드 설정 UI
7. 관리자 권한 체크
8. User Flow 문서의 게시판 관리 플로우 참고
```

**예상 산출물:**
- 게시판 관리 페이지들
- 게시판 관리 컨트롤러

---

#### 8.3 작업: 스킨 관리 페이지 생성

**프롬프트:**
```
관리자 페이지에서 스킨을 관리하는 기능을 구현해주세요.

요구사항:
1. admin/skins/index.php (스킨 목록)
2. admin/skins/apply.php (스킨 적용)
3. 다음 기능 구현:
   - 스킨 목록 조회
   - 스킨 미리보기
   - 전체 게시판에 기본 스킨 적용
   - 게시판별 개별 스킨 적용
   - 스킨 설정 변경 (색상, 레이아웃 등)
4. 스킨 모델 생성 (models/Skin.php)
5. 관리자 권한 체크
6. User Flow 문서의 스킨 관리 플로우 참고
7. TRD 문서의 스킨 시스템 섹션 참고
```

**예상 산출물:**
- 스킨 관리 페이지들
- 스킨 모델
- 스킨 관리 컨트롤러

---

#### 8.4 작업: 회원 관리 페이지 생성

**프롬프트:**
```
관리자 페이지에서 회원을 관리하는 기능을 구현해주세요.

요구사항:
1. admin/members/index.php (회원 목록)
2. admin/members/edit.php (회원 정보 수정)
3. 다음 기능 구현:
   - 회원 목록 조회 (페이징, 검색)
   - 회원 정보 수정
   - 회원 권한 변경 (일반 회원 ↔ 관리자)
   - 회원 정지/해제
   - 회원 탈퇴 처리
4. 관리자 권한 체크
5. Design System 문서의 디자인 가이드 준수
```

**예상 산출물:**
- 회원 관리 페이지들
- 회원 관리 컨트롤러

---

### 9. 스킨 시스템 작업

#### 9.1 작업: 기본 스킨 생성

**프롬프트:**
```
One Board의 기본 스킨(Basic Skin)을 생성해주세요.

요구사항:
1. skins/basic/ 디렉토리 생성
2. 다음 파일 생성:
   - index.php (게시판 목록 템플릿)
   - view.php (게시글 상세 템플릿)
   - write.php (게시글 작성 템플릿)
   - css/style.css (스킨 스타일)
   - js/script.js (스킨 스크립트)
   - config.php (스킨 설정)
3. Design System 문서의 디자인 가이드 준수
4. 반응형 디자인 적용
5. 접근성 고려
6. TRD 문서의 스킨 시스템 섹션 참고
```

**예상 산출물:**
- 기본 스킨 파일들
- 스킨 CSS/JS

---

#### 9.2 작업: 스킨 로더 시스템 생성

**프롬프트:**
```
스킨을 로드하고 적용하는 시스템을 생성해주세요.

요구사항:
1. lib/skin/SkinLoader.php 파일 생성
2. 다음 기능 구현:
   - 게시판별 스킨 조회
   - 스킨 템플릿 파일 로드
   - 스킨 CSS/JS 자동 로드
   - 스킨 변수 주입 (게시판 정보, 게시글 정보 등)
   - 스킨 설정 읽기
3. 스킨 캐싱 (선택)
4. 에러 처리 (스킨이 없을 경우 기본 스킨 사용)
5. TRD 문서의 스킨 시스템 섹션 참고
```

**예상 산출물:**
- 스킨 로더 클래스
- 스킨 헬퍼 함수

---

### 10. 보안 및 유틸리티 작업

#### 10.1 작업: 보안 유틸리티 생성

**프롬프트:**
```
보안 관련 유틸리티 함수를 생성해주세요.

요구사항:
1. includes/security.php 파일 생성
2. 다음 함수 구현:
   - csrf_token(): CSRF 토큰 생성
   - csrf_verify(): CSRF 토큰 검증
   - sanitize_input(): 입력값 정제 (XSS 방지)
   - escape_html(): HTML 이스케이프
   - validate_email(): 이메일 검증
   - validate_password(): 비밀번호 강도 검증
   - generate_random_string(): 난수 문자열 생성
3. 세션 보안 설정
4. TRD 문서의 보안 구현 섹션 참고
```

**예상 산출물:**
- `includes/security.php`
- 보안 관련 모든 함수

---

#### 10.2 작업: 공통 함수 라이브러리 생성

**프롬프트:**
```
공통으로 사용할 함수 라이브러리를 생성해주세요.

요구사항:
1. includes/functions.php 파일 생성
2. 다음 함수 구현:
   - redirect(): 리다이렉트 함수
   - view(): 뷰 렌더링 함수
   - asset(): 정적 자원 URL 생성
   - url(): URL 생성 함수
   - old(): 이전 입력값 가져오기
   - flash(): 플래시 메시지 설정/가져오기
   - format_date(): 날짜 포맷팅
   - truncate(): 텍스트 자르기
3. 에러 처리 함수
4. 디버깅 함수 (개발 환경에서만)
```

**예상 산출물:**
- `includes/functions.php`
- 공통 함수들

---

### 11. 데이터베이스 마이그레이션 작업

#### 11.1 작업: 데이터베이스 마이그레이션 스크립트 생성

**프롬프트:**
```
데이터베이스 마이그레이션 스크립트를 생성해주세요.

요구사항:
1. install/migrate.php 파일 생성
2. Database Design 문서의 모든 테이블 생성 SQL 포함
3. 초기 데이터 삽입 (기본 관리자, 기본 스킨, 기본 게시판)
4. 마이그레이션 실행 순서 관리
5. 이미 실행된 마이그레이션 체크
6. 롤백 기능 (선택)
7. 에러 처리 및 로깅
```

**예상 산출물:**
- 마이그레이션 스크립트
- 모든 테이블 생성 SQL
- 초기 데이터 삽입 SQL

---

### 12. 설치 스크립트 작업

#### 12.1 작업: 설치 마법사 생성

**프롬프트:**
```
One Board 설치 마법사를 생성해주세요.

요구사항:
1. install/index.php 파일 생성
2. 설치 단계:
   - 시스템 요구사항 체크 (PHP 버전, 확장 모듈)
   - 데이터베이스 연결 설정
   - 데이터베이스 마이그레이션 실행
   - 관리자 계정 생성
   - 기본 설정
3. 설치 완료 후 install 디렉토리 삭제 또는 보호
4. 사용자 친화적인 UI
5. 에러 처리 및 피드백
```

**예상 산출물:**
- 설치 마법사 페이지
- 설치 로직

---

### 13. 테스트 작업

#### 13.1 작업: 단위 테스트 작성

**프롬프트:**
```
주요 모델 클래스에 대한 단위 테스트를 작성해주세요.

요구사항:
1. PHPUnit 또는 간단한 테스트 프레임워크 사용
2. 다음 모델 테스트:
   - Member 모델 테스트
   - Board 모델 테스트
   - Post 모델 테스트
   - Comment 모델 테스트
3. 각 CRUD 메서드 테스트
4. 에러 케이스 테스트
5. 테스트 데이터베이스 사용
```

**예상 산출물:**
- 테스트 파일들
- 테스트 실행 스크립트

---

### 14. 문서화 작업

#### 14.1 작업: API 문서 생성

**프롬프트:**
```
One Board의 주요 API 엔드포인트 문서를 생성해주세요.

요구사항:
1. API 엔드포인트 목록
2. 각 엔드포인트의:
   - HTTP 메서드
   - URL
   - 요청 파라미터
   - 응답 형식
   - 에러 코드
3. 예시 요청/응답 포함
4. Markdown 형식
```

**예상 산출물:**
- API 문서 파일

---

### 15. 작업 우선순위

#### Phase 1 (MVP)
1. 프로젝트 기본 구조 생성
2. 데이터베이스 설정 및 마이그레이션
3. 회원 모델 및 인증 시스템
4. 게시판 모델 및 기본 기능
5. 게시글 모델 및 기본 기능
6. 이미지 Copy & Paste 기능
7. 기본 관리자 페이지
8. 기본 스킨 1개

#### Phase 2
1. 댓글 시스템
2. 파일 첨부 기능
3. 게시판별 스킨 적용
4. 고급 검색 기능
5. 회원 관리 페이지

#### Phase 3
1. 다중 스킨 제공
2. 스킨 커스터마이징
3. 통계 및 분석 기능
4. 알림 시스템

---

### 16. 작업 시 주의사항

#### 16.1 코드 품질
- TypeScript 엄격 모드 사용
- ESLint 및 Prettier 설정
- 의미 있는 변수명 및 함수명 사용
- 주석 작성 (복잡한 로직)
- 에러 처리 필수 (try-catch, 에러 바운더리)

#### 16.2 보안
- 모든 사용자 입력 검증 (Zod 스키마)
- SQL Injection 방지 (Drizzle ORM/Prisma의 Prepared Statements)
- XSS 방지 (React의 자동 이스케이프, DOMPurify 필요시)
- CSRF 방지 (Next.js 내장 보호)
- 파일 업로드 보안 (타입 검증, 크기 제한)
- 환경 변수 보안 (.env.local은 .gitignore에 포함)

#### 16.3 성능
- Server Components 적극 활용 (클라이언트 번들 크기 감소)
- 불필요한 쿼리 최소화
- 인덱스 활용 (SQLite)
- 이미지 최적화 (Next.js Image 컴포넌트)
- 정적 생성 (Static Generation) 활용
- React.memo 및 useMemo 적절히 사용
- SQLite WAL 모드 활성화

#### 16.4 접근성
- 시맨틱 HTML 사용
- ARIA 레이블 사용
- 키보드 네비게이션 지원
- 색상 대비 준수

---

### 17. 작업 완료 체크리스트

각 작업 완료 시 확인:
- [ ] 코드 작성 완료
- [ ] 관련 문서 참고 완료
- [ ] 보안 검토 완료
- [ ] 기본 테스트 완료
- [ ] 에러 처리 구현 완료
- [ ] 주석 작성 완료
- [ ] 코드 리뷰 완료

---

### 18. 참고 문서
- PRD (01-PRD.md)
- TRD (02-TRD.md)
- User Flow (03-User-Flow.md)
- Database Design (04-Database-Design.md)
- Design System (05-Design-System.md)

### 19. Next.js + SQLite 개발 참고 자료

#### 19.1 공식 문서
- Next.js 공식 문서: https://nextjs.org/docs
- React 공식 문서: https://react.dev/
- TypeScript 공식 문서: https://www.typescriptlang.org/docs/
- SQLite 공식 문서: https://www.sqlite.org/docs.html

#### 19.2 ORM/쿼리 빌더
- Drizzle ORM: https://orm.drizzle.team/
- Prisma: https://www.prisma.io/docs/
- better-sqlite3: https://github.com/WiseLibs/better-sqlite3

#### 19.3 유용한 라이브러리
- bcryptjs: 비밀번호 해시
- Zod: 입력 검증
- React Hook Form: 폼 관리
- Sharp: 이미지 처리
- react-hot-toast: 토스트 메시지

#### 19.4 개발 팁
- SQLite는 파일 기반이므로 개발/프로덕션 환경에서 다른 파일 사용 권장
- WAL 모드 활성화로 성능 향상
- Drizzle ORM 사용 시 타입 안전성 보장
- Next.js Server Components 적극 활용
- 클라이언트 컴포넌트는 필요한 경우만 사용 ('use client')


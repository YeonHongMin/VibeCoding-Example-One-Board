# Implementation Plan (One Board 구현 계획서)
## One Board 상세 구현 계획

### 1. 프로젝트 초기화

#### 1.1 Next.js 프로젝트 생성

```bash
npx create-next-app@latest oneboard --typescript --tailwind --app
cd oneboard
```

#### 1.2 필수 패키지 설치

```bash
# 데이터베이스
npm install drizzle-orm drizzle-kit better-sqlite3
npm install -D @types/better-sqlite3

# 인증 및 보안
npm install bcryptjs jsonwebtoken
npm install -D @types/bcryptjs @types/jsonwebtoken

# 유틸리티
npm install zod date-fns
npm install sharp  # 이미지 처리

# 폼 관리
npm install react-hook-form @hookform/resolvers

# UI 라이브러리 (선택)
npm install react-hot-toast
npm install lucide-react  # 아이콘
```

#### 1.3 프로젝트 구조 생성

```
oneboard/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (main)/
│   │   ├── board/
│   │   │   └── [boardKey]/
│   │   │       ├── page.tsx
│   │   │       └── [postId]/
│   │   │           └── page.tsx
│   │   └── write/
│   ├── admin/
│   │   ├── dashboard/
│   │   ├── boards/
│   │   ├── members/
│   │   └── skins/
│   └── api/
│       ├── auth/
│       ├── boards/
│       ├── posts/
│       ├── comments/
│       └── upload/
├── components/
│   ├── ui/
│   ├── auth/
│   ├── board/
│   ├── post/
│   ├── comment/
│   └── admin/
├── lib/
│   ├── db/
│   │   ├── index.ts
│   │   ├── schema.ts
│   │   ├── members.ts
│   │   ├── boards.ts
│   │   ├── posts.ts
│   │   └── comments.ts
│   ├── auth/
│   ├── upload/
│   ├── image/
│   └── utils/
├── hooks/
├── types/
├── skins/
│   └── basic/
└── uploads/
```

---

### 2. 데이터베이스 설정

#### 2.1 Drizzle 스키마 생성

**파일: `lib/db/schema.ts`**

Drizzle 스키마 작성 (04-Database-Design.md 참고)

#### 2.2 마이그레이션 실행

```bash
npx drizzle-kit generate
npx drizzle-kit migrate
```

#### 2.3 초기 데이터 삽입

- 기본 관리자 계정
- 기본 스킨 데이터
- 기본 게시판 데이터

---

### 3. 단계별 구현 계획

#### Week 1-2: 인증 시스템 구현

**목표:** 회원 가입, 로그인, 로그아웃 기능 구현

**작업 내용:**
1. 데이터베이스 스키마 설정 (members 테이블)
2. 회원 가입 API 구현
3. 로그인 API 구현 (JWT + HTTP-only Cookie)
4. 회원 가입/로그인 페이지 구현
5. 인증 미들웨어 구현

**체크리스트:**
- [ ] `lib/db/members.ts` - 회원 DB 함수 구현
- [ ] `app/api/auth/register/route.ts` - 회원가입 API
- [ ] `app/api/auth/login/route.ts` - 로그인 API
- [ ] `app/(auth)/register/page.tsx` - 회원가입 페이지
- [ ] `app/(auth)/login/page.tsx` - 로그인 페이지
- [ ] `middleware.ts` - 인증 미들웨어
- [ ] `lib/auth/session.ts` - 세션 관리 함수

**참고 파일:**
- `member/register.php`
- `member/login.php`
- `lib/common.lib.php` (인증 관련 함수)

---

#### Week 3-4: 게시판 시스템 구현

**목표:** 게시판 생성, 목록 조회, 게시판별 게시글 목록

**작업 내용:**
1. 게시판 데이터베이스 스키마 설정
2. 게시판 생성/수정/삭제 API
3. 게시판 목록 페이지
4. 게시판별 게시글 목록 페이지

**체크리스트:**
- [ ] `lib/db/boards.ts` - 게시판 DB 함수 구현
- [ ] `app/api/boards/route.ts` - 게시판 CRUD API
- [ ] `app/(main)/board/[boardKey]/page.tsx` - 게시판 목록 페이지
- [ ] `components/board/BoardList.tsx` - 게시판 목록 컴포넌트
- [ ] `components/board/PostList.tsx` - 게시글 목록 컴포넌트

**참고 파일:**
- `bbs/board.php`
- `bbs/list.php`
- `adm/board_list.php`

---

#### Week 5-6: 게시글 시스템 구현

**목표:** 게시글 작성, 조회, 수정, 삭제 기능

**작업 내용:**
1. 게시글 데이터베이스 스키마 설정
2. 게시글 CRUD API
3. 게시글 작성 페이지 (WYSIWYG 에디터)
4. 게시글 상세 페이지
5. 이미지 Copy & Paste 기능

**체크리스트:**
- [x] `lib/db/posts.ts` - 게시글 DB 함수 구현
- [x] `app/api/posts/route.ts` - 게시글 CRUD API
- [x] `app/api/posts/[id]/route.ts` - 게시글 수정/삭제 API
- [x] `app/(main)/write/page.tsx` - 게시글 작성 페이지
- [x] `app/(main)/board/[boardKey]/[postId]/page.tsx` - 게시글 상세 페이지
- [x] `components/editor/Editor.tsx` - WYSIWYG 에디터 컴포넌트
- [x] `app/api/upload/image/route.ts` - 이미지 업로드 API
- [x] `app/api/upload/file/route.ts` - 파일 업로드 API
- [x] 이미지 Copy & Paste 기능 구현
- [x] 파일 첨부 기능 (Editor 통합)
- [x] 이미지/파일 업로드 시 fileId 추출 및 게시글 연결

**참고 파일:**
- `bbs/write.php`
- `bbs/view.php`
- `lib/editor.lib.php`

---

#### Week 7-8: 파일 관리 시스템 구현

**목표:** 파일 업로드, 다운로드, 이미지 리사이징

**작업 내용:**
1. 파일 데이터베이스 스키마 설정
2. 파일 업로드 API
3. 파일 다운로드 API
4. 이미지 리사이징 (Sharp)
5. 썸네일 생성

**체크리스트:**
- [x] `lib/db/files.ts` - 파일 DB 함수 구현
- [x] `lib/upload/fileUploader.ts` - 파일 업로드 유틸리티
- [x] `app/api/upload/file/route.ts` - 파일 업로드 API
- [x] `app/api/files/[id]/route.ts` - 파일 다운로드 API
- [x] `app/api/files/[id]/download/route.ts` - 파일 다운로드 API
- [x] Editor에 파일 첨부 기능 통합

**참고 파일:**
- `lib/file.lib.php`
- `bbs/file_download.php`

---

#### Week 9-10: 댓글 시스템 구현

**목표:** 댓글 작성, 수정, 삭제, 대댓글 지원

**작업 내용:**
1. 댓글 데이터베이스 스키마 설정
2. 댓글 CRUD API
3. 댓글 컴포넌트 (계층 구조)
4. 댓글 목록 표시

**체크리스트:**
- [x] `lib/db/comments.ts` - 댓글 DB 함수 구현
- [x] `app/api/comments/route.ts` - 댓글 CRUD API
- [x] `app/api/comments/[id]/route.ts` - 댓글 수정/삭제 API
- [x] `components/comment/CommentSection.tsx` - 댓글 섹션 컴포넌트
- [x] 대댓글 기능 구현
- [x] 댓글 수정/삭제 권한 체크 (작성자 본인 또는 관리자만)

**참고 파일:**
- `bbs/comment.php`
- `bbs/comment_update.php`

---

#### Week 11-12: 관리자 페이지 구현

**목표:** 관리자 대시보드, 게시판 관리, 회원 관리

**작업 내용:**
1. 관리자 대시보드
2. 게시판 관리 페이지
3. 회원 관리 페이지
4. 권한 체크

**체크리스트:**
- [x] `app/admin/dashboard/page.tsx` - 관리자 대시보드
- [x] `app/admin/boards/page.tsx` - 게시판 관리 페이지
- [x] `app/admin/members/page.tsx` - 회원 관리 페이지
- [x] `components/admin/BoardManagement.tsx` - 게시판 관리 컴포넌트
- [x] `components/admin/MemberManagement.tsx` - 회원 관리 컴포넌트
- [x] `lib/auth/permissions.ts` - 권한 체크 함수
- [x] 관리자 미들웨어

**참고 파일:**
- `adm/index.php`
- `adm/board_list.php`
- `adm/member_list.php`

---

#### Week 13-14: 스킨 시스템 구현

**목표:** 스킨 로더, 기본 스킨, 게시판별 스킨 적용

**작업 내용:**
1. 스킨 로더 시스템
2. 기본 스킨 구현
3. 게시판별 스킨 적용
4. 스킨 관리 페이지

**체크리스트:**
- [ ] `lib/skin/skinLoader.ts` - 스킨 로더 구현
- [ ] `skins/basic/` - 기본 스킨 구현
- [ ] `app/admin/skins/page.tsx` - 스킨 관리 페이지
- [ ] 게시판별 스킨 적용 로직

**참고 파일:**
- `skin/` 디렉토리
- `adm/skin_list.php`

---

### 4. 코드 분석 및 구현 가이드

#### 4.1 코드 분석 방법

**1단계: 파일 구조 파악**
```bash
# One Board 프로젝트 구조 확인
cd oneboard

# 주요 디렉토리 분석
- app/          # Next.js App Router
  - (main)/     # 메인 콘텐츠
  - (auth)/     # 인증 관련
  - admin/      # 관리자
  - api/        # API Routes
- components/   # React 컴포넌트
- lib/          # 라이브러리 및 유틸리티
```

**2단계: 핵심 로직 추출**

각 파일에서:
1. 데이터베이스 쿼리 파악
2. 비즈니스 로직 파악
3. 입력/출력 파악
4. 보안 로직 파악

**3단계: 구현**

- TypeScript 함수 구현
- Drizzle ORM 쿼리 작성
- JWT + Cookie 세션 관리
- React 컴포넌트 구현

---

#### 4.2 주요 파일 구조

| 파일 경로 | 설명 |
|---------|------|
| `app/(main)/board/[boardKey]/page.tsx` | 게시판 목록 |
| `components/board/PostList.tsx` | 게시글 목록 |
| `app/(main)/write/page.tsx` | 게시글 작성 |
| `app/(main)/board/[boardKey]/[postId]/page.tsx` | 게시글 상세 |
| `app/(auth)/register/page.tsx` | 회원 가입 |
| `member/login.php` | `app/(auth)/login/page.tsx` | 로그인 |
| `adm/board_list.php` | `app/admin/boards/page.tsx` | 게시판 관리 |
| `adm/member_list.php` | `app/admin/members/page.tsx` | 회원 관리 |
| `lib/common.lib.php` | `lib/utils/` | 공통 함수 |
| `lib/file.lib.php` | `lib/upload/fileUploader.ts` | 파일 처리 |

---

### 5. 구현 우선순위 및 마일스톤

#### Milestone 1: MVP (최소 기능 제품)
**기간:** 6주
**기능:**
- 회원 가입/로그인
- 게시판 목록/생성
- 게시글 작성/조회
- 이미지 업로드 (Copy & Paste)
- 기본 관리자 페이지

#### Milestone 2: 핵심 기능 완성
**기간:** 4주
**기능:**
- 댓글 시스템 (수정/삭제 포함)
- 파일 첨부 (Editor 통합)
- 게시글 수정/삭제
- 게시판 권한 관리
- 고급 검색 기능 (작성자, 날짜 범위, 정렬 옵션)

#### Milestone 3: 고급 기능
**기간:** 4주
**기능:**
- 스킨 시스템
- 게시판별 스킨 적용
- 통계 및 분석
- 데이터 백업
- 포인트 시스템 (선택)

---

### 6. 개발 환경 설정

#### 6.1 환경 변수 설정

**.env.local:**
```env
# 데이터베이스
DATABASE_URL=./data/oneboard.db

# 인증
NEXTAUTH_SECRET=your-secret-key-here
NEXTAUTH_URL=http://localhost:3000

# 파일 업로드
UPLOAD_MAX_SIZE=5242880
UPLOAD_DIR=./uploads
ALLOWED_IMAGE_TYPES=jpg,jpeg,png,gif,webp
ALLOWED_FILE_TYPES=pdf,doc,docx,xls,xlsx

# 사이트 설정
SITE_NAME=One Board
SITE_URL=http://localhost:3000
```

#### 6.2 개발 스크립트

**package.json:**
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:studio": "drizzle-kit studio"
  }
}
```

---

### 7. 테스트 계획

#### 7.1 단위 테스트
- 각 함수/컴포넌트별 테스트
- Jest + React Testing Library 사용

#### 7.2 통합 테스트
- API 엔드포인트 테스트
- 전체 플로우 테스트

#### 7.3 E2E 테스트
- Playwright 사용
- 주요 사용자 시나리오 테스트

---

### 8. 배포 계획

#### 8.1 개발 환경
- 로컬 개발: SQLite 파일
- 개발 서버: Next.js dev 서버

#### 8.2 프로덕션 환경
- 호스팅: Vercel (권장) 또는 Node.js 서버
- 데이터베이스: SQLite 파일 (백업 필수)
- 파일 저장: 로컬 또는 S3 호환 스토리지

---

### 9. 다음 단계

1. **프로젝트 초기화**
   ```bash
   # 이 가이드의 1.1, 1.2, 1.3 단계 실행
   ```

2. **데이터베이스 설정**
   ```bash
   # 2단계 실행
   ```

3. **단계별 구현 시작**
   ```bash
   # Week 1-2부터 순차적으로 진행
   ```

4. **코드 분석**
   ```bash
   # 4단계 가이드 참고하여 코드 분석 및 포팅
   ```

각 단계 완료 후 체크리스트를 확인하고, 다음 단계로 진행하세요.

---

### 10. 참고 문서

- **07-Migration-Guide.md** - 구현 가이드
- **06-TASKS.md** - AI 개발 파트너용 작업 가이드
- **04-Database-Design.md** - 데이터베이스 설계
- **02-TRD.md** - 기술 요구사항
- Next.js 공식 문서: https://nextjs.org/docs


# TRD (Technical Requirements Document)
## One Board 기술 요구사항 정의서

### 1. 기술 스택

#### 1.1 프레임워크
- **프레임워크**: Next.js 14 이상 (App Router 사용)
- **언어**: TypeScript 5.0 이상
- **패턴**: Server Components + Client Components 하이브리드
- **참고**: One Board는 Next.js 기반의 현대적인 웹 애플리케이션

#### 1.2 프론트엔드
- **UI 라이브러리**: React 18 이상
- **스타일링**: Tailwind CSS 또는 CSS Modules
- **상태 관리**: React Context API 또는 Zustand (필요시)
- **폼 관리**: React Hook Form
- **에디터**: TinyMCE, Quill, 또는 Tiptap
- **이미지 처리**: Canvas API, File API, Sharp (서버 사이드)

#### 1.3 백엔드
- **API Routes**: Next.js API Routes (App Router의 Route Handlers)
- **서버 액션**: Next.js Server Actions (폼 처리 등)
- **인증**: NextAuth.js 또는 자체 구현 (JWT + HTTP-only Cookie)
- **파일 업로드**: FormData 처리, Multer 대신 Next.js 내장 기능

#### 1.4 데이터베이스
- **DBMS**: SQLite 3
- **ORM/쿼리 빌더**: Drizzle ORM 또는 Prisma
- **마이그레이션**: Drizzle Kit 또는 Prisma Migrate
- **문자 인코딩**: UTF-8
- **트랜잭션**: SQLite 트랜잭션 지원

#### 1.5 개발 환경
- **Node.js**: 18.0 이상
- **패키지 관리**: npm, yarn, 또는 pnpm
- **빌드 도구**: Next.js 내장 (Turbopack)
- **이미지 최적화**: Next.js Image 컴포넌트 + Sharp

#### 1.6 보안
- **인증**: JWT + HTTP-only Cookie 또는 NextAuth.js
- **비밀번호 암호화**: bcryptjs
- **CSRF 방지**: Next.js 내장 CSRF 보호
- **XSS 방지**: React의 자동 이스케이프, DOMPurify (필요시)
- **SQL Injection 방지**: ORM/쿼리 빌더의 Prepared Statements

---

### 2. 시스템 아키텍처

#### 2.1 디렉토리 구조 (Next.js App Router)

```
/oneboard/
├── app/                        # Next.js App Router
│   ├── (auth)/                 # 인증 관련 (그룹 라우트)
│   │   ├── login/
│   │   └── register/
│   ├── (main)/                 # 메인 콘텐츠 (그룹 라우트)
│   │   ├── board/
│   │   │   └── [boardKey]/
│   │   │       ├── page.tsx    # 게시판 목록
│   │   │       └── [postId]/
│   │   │           └── page.tsx # 게시글 상세
│   │   └── write/
│   ├── admin/                  # 관리자 페이지
│   │   ├── dashboard/
│   │   ├── boards/
│   │   ├── members/
│   │   └── skins/
│   ├── api/                    # API Routes
│   │   ├── auth/
│   │   ├── boards/
│   │   ├── posts/
│   │   ├── comments/
│   │   └── upload/
│   ├── layout.tsx              # 루트 레이아웃
│   └── page.tsx                # 홈 페이지
├── components/                 # React 컴포넌트
│   ├── ui/                     # 기본 UI 컴포넌트
│   ├── board/                  # 게시판 관련 컴포넌트
│   ├── post/                   # 게시글 관련 컴포넌트
│   ├── admin/                  # 관리자 컴포넌트
│   └── editor/                 # 에디터 컴포넌트
├── lib/                        # 유틸리티 및 라이브러리
│   ├── db/                     # 데이터베이스 설정
│   │   ├── index.ts
│   │   └── schema.ts           # Drizzle 스키마
│   ├── auth/                   # 인증 관련
│   ├── utils/                  # 유틸리티 함수
│   ├── upload/                 # 파일 업로드
│   └── image/                  # 이미지 처리
├── hooks/                      # Custom React Hooks
├── types/                      # TypeScript 타입 정의
├── styles/                     # 전역 스타일
│   └── globals.css
├── public/                     # 정적 파일
│   ├── images/
│   └── icons/
├── uploads/                    # 업로드 파일 저장 (public 외부)
│   ├── images/
│   └── files/
├── skins/                      # 스킨 디렉토리
│   ├── basic/
│   ├── modern/
│   └── classic/
├── prisma/                     # Prisma 사용 시 (선택)
│   └── schema.prisma
├── drizzle/                    # Drizzle 사용 시 (선택)
│   └── migrations/
├── .env.local                  # 환경 변수
├── next.config.js              # Next.js 설정
├── tsconfig.json               # TypeScript 설정
├── tailwind.config.js          # Tailwind 설정 (사용 시)
└── package.json
```

#### 2.2 Next.js 아키텍처 패턴

##### 2.2.1 Server Components
- 서버에서 렌더링되는 React 컴포넌트
- 데이터베이스 직접 접근 가능
- SEO 최적화
- 초기 로딩 속도 향상

##### 2.2.2 Client Components
- 클라이언트에서 인터랙티브 기능 제공
- 'use client' 디렉티브 사용
- 브라우저 API 사용 (localStorage, window 등)
- 상태 관리 및 이벤트 처리

##### 2.2.3 API Routes / Route Handlers
- RESTful API 엔드포인트
- Server Actions (폼 처리 등)
- 인증/권한 체크
- 데이터 검증 및 처리

##### 2.2.4 데이터 레이어
- ORM/쿼리 빌더를 통한 데이터베이스 접근
- 타입 안전성 보장 (TypeScript)
- 마이그레이션 관리

---

### 3. 핵심 기술 구현

#### 3.1 이미지 Copy & Paste 업로드

##### 3.1.1 클라이언트 사이드 구현
```javascript
// 에디터에 붙여넣기 이벤트 리스너
editor.addEventListener('paste', function(e) {
    const items = e.clipboardData.items;
    
    for (let i = 0; i < items.length; i++) {
        if (items[i].type.indexOf('image') !== -1) {
            const blob = items[i].getAsFile();
            uploadImage(blob);
        }
    }
});

// 드래그 앤 드롭
editor.addEventListener('drop', function(e) {
    e.preventDefault();
    const files = e.dataTransfer.files;
    handleImageFiles(files);
});
```

##### 3.1.2 서버 사이드 구현 (Next.js API Route)
- **이미지 검증**
  - MIME 타입 검증 (image/jpeg, image/png, image/gif, image/webp)
  - 파일 확장자 검증
  - 파일 크기 제한 (기본 5MB)
  - 이미지 실제 검증 (Sharp 라이브러리 사용)
  
- **이미지 처리**
  - Sharp 라이브러리로 자동 리사이징 (최대 너비 1920px)
  - 썸네일 생성 (필요시)
  - WebP 변환 (선택)
  - 파일명 난수화 (보안)

- **저장 경로**
  - `/uploads/images/YYYY/MM/DD/` 형식
  - 파일명: `{timestamp}_{random}.{ext}`
  - Next.js public 폴더 외부에 저장 (보안)

#### 3.2 스킨 시스템

##### 3.2.1 스킨 구조 (React 컴포넌트 기반)
```
/skins/{skin_name}/
├── components/        # 스킨 전용 컴포넌트
│   ├── BoardList.tsx
│   ├── PostView.tsx
│   └── PostWrite.tsx
├── styles/           # 스킨 스타일
│   └── skin.css
├── config.ts         # 스킨 설정 파일
└── layout.tsx        # 스킨 레이아웃
```

##### 3.2.2 스킨 로딩 메커니즘
- 게시판별 스킨 설정 조회 (SQLite)
- 동적 import로 스킨 컴포넌트 로드
- 스킨 CSS 자동 로드
- Context API로 스킨 설정 전달

##### 3.2.3 스킨 API
- 스킨에서 사용할 수 있는 헬퍼 함수 제공
- 공통 레이아웃 컴포넌트
- 위젯 시스템 (향후 확장)

#### 3.3 권한 관리 시스템

##### 3.3.1 권한 레벨
- **0**: 비회원
- **1**: 일반 회원
- **2**: 관리자

##### 3.3.2 권한 체크
- 게시판별 읽기/쓰기 권한 설정
- Server Component에서 권한 체크
- API Route에서 권한 체크
- Client Component에서 UI 표시 제어
- 미들웨어로 라우트 보호 (Next.js Middleware)

#### 3.4 파일 업로드 시스템

##### 3.4.1 업로드 프로세스 (Next.js)
1. 클라이언트에서 파일 선택/드래그
2. 파일 검증 (크기, 타입) - 클라이언트 사이드
3. FormData로 서버로 전송 (fetch API)
4. Next.js API Route에서 재검증
5. 파일 저장 (public 폴더 외부) 및 DB 기록 (SQLite)
6. JSON 응답 반환 (URL 또는 파일 정보)

##### 3.4.2 보안 처리
- 파일명 난수화
- 업로드 디렉토리 웹 접근 제한 (next.config.js rewrites)
- 실행 파일 업로드 방지
- 파일 타입 화이트리스트 방식
- 파일 크기 제한 (서버 사이드)

---

### 4. 데이터베이스 설계

#### 4.1 주요 테이블 (상세는 Database Design 문서 참조)

##### 4.1.1 회원 테이블 (members)
- 회원 기본 정보
- 권한 정보
- 가입일, 최종 접속일

##### 4.1.2 게시판 테이블 (boards)
- 게시판 정보
- 게시판 설정
- 스킨 정보

##### 4.1.3 게시글 테이블 (posts)
- 게시글 내용
- 작성자 정보
- 조회수, 좋아요 등

##### 4.1.4 댓글 테이블 (comments)
- 댓글 내용
- 부모 댓글 정보 (대댓글)

##### 4.1.5 파일 테이블 (files)
- 업로드된 파일 정보
- 게시글/댓글과의 연관

#### 4.2 인덱스 전략 (SQLite)
- 자주 조회되는 컬럼에 인덱스 생성
- 복합 인덱스 활용
- 외래키 인덱스 (SQLite는 외래키 제약조건 지원)
- FULLTEXT 인덱스 (FTS5 모듈 사용)

---

### 5. API 설계

#### 5.1 API 설계 (Next.js Route Handlers)

##### 5.1.1 엔드포인트 예시
```
POST   /api/auth/login          # 로그인
POST   /api/auth/logout         # 로그아웃
GET    /api/boards              # 게시판 목록
GET    /api/boards/[boardKey]/posts   # 게시글 목록
POST   /api/posts               # 게시글 작성
POST   /api/upload/image        # 이미지 업로드
```

##### 5.1.2 Route Handler 구조
```typescript
// app/api/posts/route.ts
export async function GET(request: Request) {
  // GET 처리
}

export async function POST(request: Request) {
  // POST 처리
}
```

##### 5.1.3 Server Actions (폼 처리)
```typescript
// app/actions/post.ts
'use server'

export async function createPost(formData: FormData) {
  // 서버 액션 처리
}
```

##### 5.1.4 응답 형식
```json
{
    "success": true,
    "data": {},
    "message": "",
    "error": null
}
```

---

### 6. 보안 구현

#### 6.1 입력 검증
- 모든 사용자 입력 검증 (Zod 또는 Yup 사용)
- 화이트리스트 방식 우선
- 서버 사이드 검증 필수 (API Route, Server Actions)

#### 6.2 출력 이스케이프
- React의 자동 이스케이프 활용
- 위험한 HTML은 DOMPurify 사용
- SQL 쿼리 시 ORM/쿼리 빌더의 Prepared Statements

#### 6.3 인증 보안
- JWT + HTTP-only Cookie 사용
- 토큰 만료 시간 설정
- 리프레시 토큰 구현 (선택)
- CSRF 토큰 검증

#### 6.4 파일 업로드 보안
- 파일 타입 검증 (MIME 타입 + 확장자)
- 파일 크기 제한
- Sharp로 이미지 재처리 (악성 코드 제거)
- 업로드 디렉토리 웹 접근 차단 (next.config.js)

---

### 7. 성능 최적화

#### 7.1 데이터베이스 최적화 (SQLite)
- 쿼리 최적화
- 인덱스 활용
- 페이지네이션
- WAL 모드 활성화 (성능 향상)
- 캐싱 (선택, Redis 등)

#### 7.2 프론트엔드 최적화 (Next.js)
- 자동 코드 스플리팅
- 이미지 최적화 (Next.js Image 컴포넌트)
- 레이지 로딩 (자동)
- 정적 생성 (Static Generation) 활용
- 서버 컴포넌트로 번들 크기 감소

#### 7.3 서버 최적화
- Next.js 빌드 최적화
- Gzip 압축 (자동)
- 정적 파일 캐싱 (자동)
- Edge Runtime 활용 (선택)

---

### 8. 에러 처리

#### 8.1 에러 로깅
- 에러 로그 파일 기록
- 민감 정보 제외
- 로그 로테이션

#### 8.2 사용자 친화적 에러 메시지
- 기술적 에러는 로그에만 기록
- 사용자에게는 일반적인 메시지 표시
- 에러 코드 시스템

---

### 9. 테스트 전략

#### 9.1 단위 테스트
- React 컴포넌트 테스트 (Jest + React Testing Library)
- 유틸리티 함수 테스트 (Jest)
- Server Actions 테스트

#### 9.2 통합 테스트
- API Route 테스트
- 데이터베이스 연동 테스트 (테스트용 SQLite DB)

#### 9.3 E2E 테스트
- Playwright 또는 Cypress 사용
- 주요 기능 시나리오 테스트
- 브라우저 호환성 테스트

---

### 10. 배포 전략

#### 10.1 환경 분리
- 개발 환경 (Development): `npm run dev`
- 스테이징 환경 (Staging)
- 프로덕션 환경 (Production): `npm run build && npm start`

#### 10.2 배포 프로세스 (Next.js)
1. 코드 빌드 (`npm run build`)
2. 데이터베이스 마이그레이션 (Drizzle/Prisma migrate)
3. 환경 변수 설정 (.env.production)
4. 프로덕션 서버 실행
5. 건강 상태 확인

#### 10.3 배포 옵션
- Vercel (권장, Next.js 최적화)
- Node.js 서버 (PM2 사용)
- Docker 컨테이너

---

### 11. 참고 기술 문서
- Next.js 공식 문서: https://nextjs.org/docs
- React 공식 문서: https://react.dev/
- TypeScript 공식 문서: https://www.typescriptlang.org/docs/
- SQLite 공식 문서: https://www.sqlite.org/docs.html
- Drizzle ORM 문서: https://orm.drizzle.team/
- Prisma 문서: https://www.prisma.io/docs/
- W3C 웹 표준: https://www.w3.org/standards/


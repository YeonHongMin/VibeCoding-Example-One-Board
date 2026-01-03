# One Board 설치 가이드

## 빠른 시작

### 1. 패키지 설치

```bash
npm install
```

**주의:** `better-sqlite3` 설치에 문제가 있는 경우:
- 네트워크 문제일 수 있습니다. 다시 시도하거나
- `npm install better-sqlite3 --build-from-source` 시도

### 2. 환경 변수 설정

`.env.local` 파일을 생성하고 다음 내용을 추가하세요:

```env
DATABASE_URL=./data/oneboard.db
NEXTAUTH_SECRET=your-random-secret-key-at-least-32-characters-long
NEXTAUTH_URL=http://localhost:3000
UPLOAD_MAX_SIZE=5242880
UPLOAD_DIR=./uploads
ALLOWED_IMAGE_TYPES=jpg,jpeg,png,gif,webp
ALLOWED_FILE_TYPES=pdf,doc,docx,xls,xlsx
SITE_NAME=One Board
SITE_URL=http://localhost:3000
```

### 3. 데이터베이스 초기화

```bash
# 마이그레이션 생성
npm run db:generate

# 마이그레이션 실행
npm run db:migrate

# 초기 데이터 삽입 (관리자 계정, 기본 게시판 등)
npm run db:init
```

### 4. 개발 서버 실행

```bash
npm run dev
```

브라우저에서 [http://localhost:3000](http://localhost:3000)을 열어 확인하세요.

## 기본 관리자 계정

초기화 스크립트 실행 후 다음 계정으로 로그인할 수 있습니다:

- **이메일:** admin@oneboard.com
- **비밀번호:** admin123!

**보안을 위해 프로덕션 환경에서는 반드시 비밀번호를 변경하세요!**

## 디렉토리 구조

프로젝트 실행 전 다음 디렉토리가 자동 생성됩니다:

- `data/` - SQLite 데이터베이스 파일
- `uploads/` - 업로드된 파일 저장

## 문제 해결

### 데이터베이스 오류

데이터베이스 파일이 없거나 오류가 발생하는 경우:

```bash
# 데이터베이스 디렉토리 생성
mkdir -p data

# 마이그레이션 재실행
npm run db:migrate

# 초기 데이터 재삽입
npm run db:init
```

### 포트 충돌

3000번 포트가 이미 사용 중인 경우:

```bash
# 다른 포트로 실행
PORT=3001 npm run dev
```

## 다음 단계

설치가 완료되면 다음 문서를 참고하세요:

- `README.md` - 프로젝트 개요
- `01-PRD.md` - 제품 요구사항
- `02-TRD.md` - 기술 요구사항
- `06-TASKS.md` - 개발 작업 가이드


# 화면 UI 및 구조 문서

One Board 프로젝트의 현재 화면 UI 구조와 디자인 시스템에 대한 문서입니다.

## 1. 메인 레이아웃 구조 (`app/(main)/layout.tsx`)

사용자 페이지의 공통 레이아웃입니다.

- **Header (GNB)**
    - **Logo**: 'One Board' 텍스트 로고 (메인으로 이동)
    - **Navigation**: 
        - 공지사항 (/board/notice)
        - 자유게시판 (/board/free)
    - **User Menu**: 
        - 로그인 전: [로그인] [회원가입] 버튼
        - 로그인 후: `[닉네임]님` 표시, `관리자 페이지` 링크(관리자만), [로그아웃] 버튼

- **Body**: 페이지별 콘텐츠 영역 (`min-h-screen`, `flex-grow`)
- **Footer**: 카피라이트 정보 표시

---

## 2. 관리자 레이아웃 구조 (`app/admin/layout.tsx`)

관리자 전용 페이지의 레이아웃입니다. 좌측 고정 사이드바를 사용하는 대시보드 형태입니다.

- **Sidebar (Left, Fixed)**
    - **Header**: One Board Admin 로고
    - **Navigation Menu**:
        - 대시보드 (/admin/dashboard): 전체 현황 요약
        - 게시판 관리 (/admin/boards): 게시판 생성/수정/삭제
        - 회원 관리 (/admin/members): 회원 목록 및 상태 관리
        - 스킨 관리 (/admin/skins): 게시판 스킨 활성화/비활성화
    - **Footer**: 사용자 모드로 돌아가기 링크

- **Main Content (Right)**
    - 각 관리 페이지의 콘텐츠가 표시되는 영역

---

## 3. 주요 페이지 상세

### 3.1 게시판 상세 (`/board/[boardKey]`)
- **Header**: 게시판 이름, 설명, 검색바, 글쓰기 버튼
- **Start**: `BoardHeader`, `SearchBar` 컴포넌트 사용
- **List**: 게시글 목록 (`PostList`)
    - 제목, 작성자, 날짜, 조회수, 댓글수 표시
    - 공지사항 상단 고정 스타일링
    - 검색 결과 필터링 지원

### 3.2 게시글 상세 (`/board/[boardKey]/[postId]`)
- **Navigation**: 목록으로 돌아가기 버튼
- **Post Header**: 제목, 작성자 정보, 작성일, 조회수
- **Post Content**: HTML 렌더링 된 본문 내용
- **Interaction**: 수정/삭제 버튼 (권한 있는 경우)
- **Comments**: 댓글 섹션 (`CommentSection`)
    - 댓글 작성 폼
    - 댓글 목록 (계층형 대댓글 구조)

### 3.3 글쓰기/수정 (`/board/[boardKey]/write`, `.../edit`)
- **Form**: 제목 입력, 옵션(공지사항, 비밀글) 체크박스
- **Editor**: WYSIWYG 에디터 (`TipTap` 기반)
    - 이미지 업로드 지원 (드래그 앤 드롭 또는 버튼)
    - 서식 도구 (볼드, 이탤릭, 리스트 등)

### 3.4 관리자 페이지
- **대시보드**: 통계 카드 (회원수, 게시글수 등)
- **게시판 관리**: 게시판 목록 테이블, 상태 토글, 편집 기능
- **회원 관리**: 회원 검색, 목록, 차단/해제 토글
- **스킨 관리**: 스킨 목록 및 사용 여부 설정

---

## 4. 디자인 시스템 (TailwindCSS)
- **Colors**:
    - Primary: `--color-primary` 변수를 사용하여 스킨별 색상 테마 적용
    - Slate/Gray 계열: 중립적인 배경 및 텍스트 색상
- **Components**:
    - `btn-primary`, `btn-secondary`: 공통 버튼 스타일 (globals.css 정의)
    - `input-field`: 공통 입력 필드 스타일
- **Feedback**:
    - `react-hot-toast`: 작업 성공/실패 시 우측 하단 토스트 메시지
    - Loading State: 버튼 및 데이터 로딩 시 `Loader2` 스피너 아이콘 사용

## 5. 반응형 구조
- **Mobile First**: 기본적으로 모바일 스타일을 먼저 정의
- **Breakpoints**: 
    - `md`: 태블릿/데스크탑 분기점 (사이드바 표시, 그리드 레이아웃 변경 등)
    - `lg`: 대형 화면 최적화

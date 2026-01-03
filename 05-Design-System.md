# Design System (기초 디자인 시스템)
## One Board 디자인 시스템 가이드

### 1. 디자인 철학

#### 1.1 디자인 원칙
- **단순성 (Simplicity)**: 불필요한 요소 제거, 직관적인 인터페이스
- **일관성 (Consistency)**: 전체 시스템에서 일관된 디자인 언어 사용
- **접근성 (Accessibility)**: 모든 사용자가 쉽게 사용할 수 있는 디자인
- **반응형 (Responsive)**: 다양한 디바이스에서 최적화된 경험 제공
- **현대성 (Modernity)**: 최신 웹 디자인 트렌드 반영

#### 1.2 디자인 목표
- 사용자가 3초 이내에 원하는 기능을 찾을 수 있도록
- 모바일과 데스크톱 모두에서 일관된 경험 제공
- 시각적 피로도 최소화

---

### 2. 색상 시스템

#### 2.1 주요 색상 (Primary Colors)

##### 2.1.1 브랜드 컬러
```
Primary Blue: #2563EB
- RGB: rgb(37, 99, 235)
- 사용: 주요 버튼, 링크, 강조 요소

Primary Blue Dark: #1E40AF
- RGB: rgb(30, 64, 175)
- 사용: 호버 상태, 활성 상태

Primary Blue Light: #3B82F6
- RGB: rgb(59, 130, 246)
- 사용: 배경, 하이라이트
```

##### 2.1.2 보조 색상 (Secondary Colors)
```
Success Green: #10B981
- RGB: rgb(16, 185, 129)
- 사용: 성공 메시지, 완료 상태

Warning Yellow: #F59E0B
- RGB: rgb(245, 158, 11)
- 사용: 경고 메시지, 주의 사항

Error Red: #EF4444
- RGB: rgb(239, 68, 68)
- 사용: 에러 메시지, 삭제 버튼

Info Blue: #3B82F6
- RGB: rgb(59, 130, 246)
- 사용: 정보 메시지, 알림
```

#### 2.2 중성 색상 (Neutral Colors)

##### 2.2.1 그레이 스케일
```
Gray 50:  #F9FAFB  (가장 밝은 배경)
Gray 100: #F3F4F6  (배경)
Gray 200: #E5E7EB  (경계선)
Gray 300: #D1D5DB  (비활성 요소)
Gray 400: #9CA3AF  (플레이스홀더)
Gray 500: #6B7280  (보조 텍스트)
Gray 600: #4B5563  (본문 텍스트)
Gray 700: #374151  (강조 텍스트)
Gray 800: #1F2937  (제목 텍스트)
Gray 900: #111827  (가장 어두운 텍스트)
```

##### 2.2.2 흑백
```
White: #FFFFFF
Black: #000000
```

#### 2.3 색상 사용 가이드

##### 2.3.1 텍스트 색상
- **제목**: Gray 900 (#111827)
- **본문**: Gray 700 (#374151)
- **보조 텍스트**: Gray 500 (#6B7280)
- **링크**: Primary Blue (#2563EB)
- **링크 호버**: Primary Blue Dark (#1E40AF)

##### 2.3.2 배경 색상
- **메인 배경**: White (#FFFFFF)
- **섹션 배경**: Gray 50 (#F9FAFB)
- **카드 배경**: White (#FFFFFF)
- **호버 배경**: Gray 100 (#F3F4F6)

##### 2.3.3 경계선 색상
- **기본 경계선**: Gray 200 (#E5E7EB)
- **강조 경계선**: Gray 300 (#D1D5DB)
- **포커스 경계선**: Primary Blue (#2563EB)

---

### 3. 타이포그래피

#### 3.1 폰트 패밀리

##### 3.1.1 한글 폰트
```
Primary: 'Noto Sans KR', sans-serif
Fallback: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
```

##### 3.1.2 영문 폰트
```
Primary: 'Inter', sans-serif
Fallback: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
```

##### 3.1.3 코드 폰트
```
Primary: 'Fira Code', 'Consolas', monospace
```

#### 3.2 폰트 크기 스케일

```
xs:   0.75rem  (12px)  - 작은 라벨, 캡션
sm:   0.875rem (14px)  - 보조 텍스트
base: 1rem     (16px)  - 본문 텍스트 (기본)
lg:   1.125rem (18px)  - 강조 텍스트
xl:   1.25rem  (20px)  - 소제목
2xl:  1.5rem   (24px)  - 제목
3xl:  1.875rem (30px)  - 큰 제목
4xl:  2.25rem  (36px)  - 헤더
```

#### 3.3 폰트 두께

```
Light:    300
Regular:  400 (기본)
Medium:   500
Semibold: 600
Bold:     700
```

#### 3.4 라인 높이 (Line Height)

```
Tight:  1.25  (제목)
Normal: 1.5   (본문)
Relaxed: 1.75 (긴 텍스트)
```

#### 3.5 타이포그래피 사용 예시

```css
/* 제목 (H1) */
h1 {
    font-size: 2.25rem;    /* 4xl */
    font-weight: 700;      /* Bold */
    line-height: 1.25;
    color: #111827;        /* Gray 900 */
}

/* 제목 (H2) */
h2 {
    font-size: 1.875rem;   /* 3xl */
    font-weight: 600;      /* Semibold */
    line-height: 1.25;
    color: #1F2937;        /* Gray 800 */
}

/* 본문 */
p {
    font-size: 1rem;       /* base */
    font-weight: 400;      /* Regular */
    line-height: 1.75;
    color: #374151;        /* Gray 700 */
}

/* 링크 */
a {
    font-size: 1rem;
    color: #2563EB;        /* Primary Blue */
    text-decoration: none;
}

a:hover {
    color: #1E40AF;        /* Primary Blue Dark */
    text-decoration: underline;
}
```

---

### 4. 간격 시스템 (Spacing)

#### 4.1 간격 스케일 (8px 기준)

```
0:   0px
1:   0.25rem  (4px)
2:   0.5rem   (8px)
3:   0.75rem  (12px)
4:   1rem     (16px)
5:   1.25rem  (20px)
6:   1.5rem   (24px)
8:   2rem     (32px)
10:  2.5rem   (40px)
12:  3rem     (48px)
16:  4rem     (64px)
20:  5rem     (80px)
24:  6rem     (96px)
```

#### 4.2 간격 사용 가이드

- **컴포넌트 내부 간격**: 4 (16px), 6 (24px)
- **컴포넌트 간 간격**: 6 (24px), 8 (32px)
- **섹션 간 간격**: 12 (48px), 16 (64px)
- **페이지 여백**: 4 (16px) ~ 8 (32px)

---

### 5. 레이아웃 시스템

#### 5.1 그리드 시스템

##### 5.1.1 컨테이너 너비
```
Mobile:  100% (패딩 16px)
Tablet:  768px (패딩 24px)
Desktop: 1200px (패딩 32px)
Large:   1400px (패딩 32px)
```

##### 5.1.2 그리드 컬럼
```
Mobile:   1 컬럼
Tablet:   2-3 컬럼
Desktop:  3-4 컬럼
Large:    4-6 컬럼
```

#### 5.2 브레이크포인트

```css
/* Mobile First 접근 */
/* Mobile: 기본 (0px ~) */
/* Tablet: 768px 이상 */
@media (min-width: 768px) { }

/* Desktop: 1024px 이상 */
@media (min-width: 1024px) { }

/* Large: 1280px 이상 */
@media (min-width: 1280px) { }
```

#### 5.3 레이아웃 구조

```
┌─────────────────────────────────┐
│         Header (고정)           │
├─────────────────────────────────┤
│                                 │
│    Main Content Area            │
│    (최대 너비: 1200px)          │
│                                 │
│    ┌──────────┬──────────┐     │
│    │  Sidebar │  Content │     │
│    │  (선택)  │          │     │
│    └──────────┴──────────┘     │
│                                 │
├─────────────────────────────────┤
│         Footer                  │
└─────────────────────────────────┘
```

---

### 6. 컴포넌트 디자인

#### 6.1 버튼 (Button)

##### 6.1.1 버튼 스타일

**Primary Button**
```css
background: #2563EB;        /* Primary Blue */
color: #FFFFFF;
padding: 0.75rem 1.5rem;
border-radius: 0.5rem;
font-weight: 500;
border: none;
cursor: pointer;
```

**Secondary Button**
```css
background: #FFFFFF;
color: #2563EB;
padding: 0.75rem 1.5rem;
border-radius: 0.5rem;
font-weight: 500;
border: 2px solid #2563EB;
cursor: pointer;
```

**Danger Button**
```css
background: #EF4444;        /* Error Red */
color: #FFFFFF;
padding: 0.75rem 1.5rem;
border-radius: 0.5rem;
font-weight: 500;
border: none;
cursor: pointer;
```

##### 6.1.2 버튼 크기

```
Small:   padding: 0.5rem 1rem;    font-size: 0.875rem;
Medium:  padding: 0.75rem 1.5rem; font-size: 1rem;      (기본)
Large:   padding: 1rem 2rem;      font-size: 1.125rem;
```

##### 6.1.3 버튼 상태

- **기본**: 위 스타일 적용
- **호버**: 배경색 10% 어둡게, transform: translateY(-1px)
- **활성**: 배경색 20% 어둡게
- **비활성**: opacity: 0.5, cursor: not-allowed

#### 6.2 입력 필드 (Input)

##### 6.2.1 기본 스타일
```css
width: 100%;
padding: 0.75rem 1rem;
border: 1px solid #E5E7EB;        /* Gray 200 */
border-radius: 0.5rem;
font-size: 1rem;
color: #374151;                    /* Gray 700 */
background: #FFFFFF;
```

##### 6.2.2 입력 필드 상태

- **포커스**: border-color: #2563EB, box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1)
- **에러**: border-color: #EF4444
- **비활성**: background: #F3F4F6, cursor: not-allowed

#### 6.3 카드 (Card)

##### 6.3.1 기본 스타일
```css
background: #FFFFFF;
border: 1px solid #E5E7EB;        /* Gray 200 */
border-radius: 0.75rem;
padding: 1.5rem;
box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
```

##### 6.3.2 카드 호버
```css
box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
transform: translateY(-2px);
transition: all 0.2s;
```

#### 6.4 배지 (Badge)

##### 6.4.1 기본 스타일
```css
display: inline-block;
padding: 0.25rem 0.75rem;
border-radius: 9999px;             /* 완전히 둥근 모서리 */
font-size: 0.875rem;
font-weight: 500;
```

##### 6.4.2 배지 색상 변형
- **Primary**: background: #DBEAFE, color: #1E40AF
- **Success**: background: #D1FAE5, color: #065F46
- **Warning**: background: #FEF3C7, color: #92400E
- **Error**: background: #FEE2E2, color: #991B1B

#### 6.5 알림 (Alert)

##### 6.5.1 기본 스타일
```css
padding: 1rem 1.5rem;
border-radius: 0.5rem;
border-left: 4px solid;
margin-bottom: 1rem;
```

##### 6.5.2 알림 타입
- **Success**: background: #D1FAE5, border-color: #10B981, color: #065F46
- **Warning**: background: #FEF3C7, border-color: #F59E0B, color: #92400E
- **Error**: background: #FEE2E2, border-color: #EF4444, color: #991B1B
- **Info**: background: #DBEAFE, border-color: #3B82F6, color: #1E40AF

---

### 7. 아이콘 시스템

#### 7.1 아이콘 라이브러리
- **추천**: Heroicons, Feather Icons, 또는 Font Awesome
- **크기**: 16px, 20px, 24px, 32px
- **스타일**: Outline (기본), Solid (강조)

#### 7.2 아이콘 사용 가이드
- 텍스트와 함께 사용 시 아이콘 크기는 텍스트 크기와 동일하거나 약간 작게
- 버튼 내 아이콘은 텍스트와 8px 간격 유지
- 색상은 텍스트 색상과 동일하게

---

### 8. 애니메이션 및 전환 효과

#### 8.1 전환 시간 (Transition Duration)

```
Fast:   150ms  (호버 효과)
Normal: 200ms  (기본)
Slow:   300ms  (복잡한 애니메이션)
```

#### 8.2 전환 함수 (Easing)

```css
/* 기본 */
transition: all 0.2s ease-in-out;

/* 부드러운 */
transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
```

#### 8.3 주요 애니메이션

##### 8.3.1 페이드 인/아웃
```css
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

@keyframes fadeOut {
    from { opacity: 1; }
    to { opacity: 0; }
}
```

##### 8.3.2 슬라이드
```css
@keyframes slideUp {
    from { transform: translateY(20px); opacity: 0; }
    to { transform: translateY(0); opacity: 1; }
}
```

##### 8.3.3 로딩 스피너
```css
@keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
}
```

---

### 9. 그림자 시스템 (Shadows)

```css
/* 작은 그림자 */
box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);

/* 기본 그림자 */
box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);

/* 중간 그림자 */
box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);

/* 큰 그림자 */
box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);

/* 매우 큰 그림자 */
box-shadow: 0 20px 25px rgba(0, 0, 0, 0.1);
```

---

### 10. 반응형 디자인 가이드

#### 10.1 모바일 우선 설계
- 기본 스타일은 모바일용으로 작성
- 미디어 쿼리로 데스크톱 스타일 추가

#### 10.2 터치 타겟 크기
- 최소 44px × 44px (iOS 가이드라인)
- 버튼, 링크 등 인터랙티브 요소에 적용

#### 10.3 모바일 최적화
- 네비게이션: 햄버거 메뉴
- 테이블: 스크롤 또는 카드 형태로 변환
- 이미지: 자동 리사이징

---

### 11. 접근성 (Accessibility)

#### 11.1 색상 대비
- 본문 텍스트: 최소 4.5:1 대비율
- 큰 텍스트: 최소 3:1 대비율
- WCAG 2.1 Level AA 준수

#### 11.2 키보드 네비게이션
- 모든 인터랙티브 요소는 키보드로 접근 가능
- 포커스 인디케이터 명확히 표시
- Tab 순서 논리적으로 구성

#### 11.3 스크린 리더 지원
- 의미 있는 alt 텍스트 제공
- ARIA 레이블 적절히 사용
- 시맨틱 HTML 사용

---

### 12. 다크 모드 (선택사항, 향후 확장)

#### 12.1 다크 모드 색상 팔레트
```
Background: #111827        /* Gray 900 */
Surface: #1F2937           /* Gray 800 */
Text: #F9FAFB              /* Gray 50 */
Border: #374151            /* Gray 700 */
```

#### 12.2 구현 방법
- CSS 변수 사용
- `prefers-color-scheme` 미디어 쿼리 활용
- 사용자 설정 저장

---

### 13. 스킨 시스템 연동

#### 13.1 스킨별 커스터마이징
- 색상 변수 오버라이드 가능
- 폰트 설정 변경 가능
- 레이아웃 옵션 제공

#### 13.2 스킨 개발 가이드
- 기본 CSS 변수 사용
- 스킨별 CSS 파일 분리
- 설정 파일로 테마 변경

---

### 14. 디자인 토큰 (Design Tokens)

#### 14.1 CSS 변수 정의

```css
:root {
    /* 색상 */
    --color-primary: #2563EB;
    --color-primary-dark: #1E40AF;
    --color-success: #10B981;
    --color-warning: #F59E0B;
    --color-error: #EF4444;
    
    /* 그레이 */
    --color-gray-50: #F9FAFB;
    --color-gray-100: #F3F4F6;
    --color-gray-200: #E5E7EB;
    --color-gray-500: #6B7280;
    --color-gray-700: #374151;
    --color-gray-900: #111827;
    
    /* 간격 */
    --spacing-1: 0.25rem;
    --spacing-2: 0.5rem;
    --spacing-4: 1rem;
    --spacing-6: 1.5rem;
    --spacing-8: 2rem;
    
    /* 폰트 */
    --font-family-base: 'Noto Sans KR', sans-serif;
    --font-size-base: 1rem;
    --font-weight-normal: 400;
    --font-weight-medium: 500;
    --font-weight-bold: 700;
    
    /* 테두리 */
    --border-radius-sm: 0.25rem;
    --border-radius-md: 0.5rem;
    --border-radius-lg: 0.75rem;
    
    /* 그림자 */
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
    --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
    --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
    
    /* 전환 */
    --transition-fast: 150ms;
    --transition-normal: 200ms;
    --transition-slow: 300ms;
}
```

---

### 15. 디자인 시스템 문서화

#### 15.1 스타일 가이드 문서
- 컴포넌트 사용 예시
- 코드 스니펫 제공
- Do's and Don'ts 가이드

#### 15.2 디자인 파일
- Figma 또는 Sketch 디자인 파일
- 컴포넌트 라이브러리
- 아이콘 세트

---

### 16. 참고 자료
- Tailwind CSS: https://tailwindcss.com/
- Material Design: https://material.io/design
- Apple Human Interface Guidelines: https://developer.apple.com/design/
- WCAG 2.1: https://www.w3.org/WAI/WCAG21/quickref/


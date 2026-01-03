# 문제 해결 가이드 (Troubleshooting Guide)

이 문서는 One Board 프로젝트에서 발생한 문제점들과 해결 방법을 정리한 것입니다. 다른 프로젝트에 적용할 수 있도록 일반화된 형태로 작성되었습니다.

## 목차

1. [게시글 수정 후 목록 이동 문제](#1-게시글-수정-후-목록-이동-문제)
2. [사용자 프로필 수정 기능 구현](#2-사용자-프로필-수정-기능-구현)
3. [회원 게시글 작성자 표시 문제](#3-회원-게시글-작성자-표시-문제)
4. [다크 스킨 시스템 구현](#4-다크-스킨-시스템-구현)
5. [회원 가입 에러 수정](#5-회원-가입-에러-수정)
6. [로그인 시스템 개선 (username 기반)](#6-로그인-시스템-개선-username-기반)
7. [게시글 수정 권한 체크 및 버튼 비활성화](#7-게시글-수정-권한-체크-및-버튼-비활성화)
8. [게시글 목록 정렬 순서 개선](#8-게시글-목록-정렬-순서-개선)
9. [게시글 내용 줄바꿈 무시 문제](#9-게시글-내용-줄바꿈-무시-문제)
10. [Better-SQLite3 트랜잭션과 비동기 처리 문제](#10-better-sqlite3-트랜잭션과-비동기-처리-문제)
11. [스킨 관리 페이지 404 에러 및 구현](#11-스킨-관리-페이지-404-에러-및-구현)

---

## 1. 게시글 수정 후 목록 이동 문제

### 문제 상황
게시글을 수정한 후 상세 페이지로 이동하는데, 사용자는 목록으로 돌아가길 원함.

### 원인
게시글 작성과 수정이 동일한 로직을 사용하여, 수정 후에도 작성과 동일하게 상세 페이지로 이동함.

### 해결 방법

**파일**: `app/(main)/write/page.tsx`

```typescript
// 수정 전
if (result.success) {
  toast.success(postId ? '게시글이 수정되었습니다' : '게시글이 작성되었습니다');
  
  // 게시글 상세 페이지로 이동
  if (result.data.id && result.data.boardKey) {
    router.push(`/board/${result.data.boardKey}/${result.data.id}`);
  } else if (result.data.boardKey) {
    router.push(`/board/${result.data.boardKey}`);
  } else {
    router.push('/');
  }
}

// 수정 후
if (result.success) {
  toast.success(postId ? '게시글이 수정되었습니다' : '게시글이 작성되었습니다');
  
  // 수정 모드일 때는 목록으로 이동, 작성 모드일 때는 상세 페이지로 이동
  if (postId) {
    // 수정 후 목록으로 이동
    if (result.data.boardKey) {
      router.push(`/board/${result.data.boardKey}`);
    } else {
      router.push('/');
    }
  } else {
    // 작성 후 상세 페이지로 이동
    if (result.data.id && result.data.boardKey) {
      router.push(`/board/${result.data.boardKey}/${result.data.id}`);
    } else if (result.data.boardKey) {
      router.push(`/board/${result.data.boardKey}`);
    } else {
      router.push('/');
    }
  }
}
```

### 핵심 포인트
- **작업 유형에 따른 분기 처리**: 수정과 작성의 사용자 경험(UX)이 다를 수 있으므로, 작업 유형에 따라 다른 동작을 수행해야 함
- **조건부 리다이렉션**: `postId` 존재 여부로 수정/작성 모드를 구분

### 다른 프로젝트 적용 시
- CRUD 작업 후 리다이렉션 로직을 작업 유형별로 분리
- 사용자 피드백을 수집하여 적절한 이동 경로 결정

---

## 2. 사용자 프로필 수정 기능 구현

### 문제 상황
사용자가 자신의 정보를 수정할 수 있는 기능이 없음.

### 해결 방법

#### 2.1 프로필 수정 API 구현

**파일**: `app/api/auth/profile/route.ts`

```typescript
// GET: 현재 사용자 정보 조회
export async function GET() {
  const session = await getSession();
  if (!session) {
    return NextResponse.json({ success: false, message: '로그인이 필요합니다' }, { status: 401 });
  }
  
  const member = await findMemberById(session.memberId as number);
  // ... 회원 정보 반환
}

// PATCH: 프로필 정보 수정
export async function PATCH(request: NextRequest) {
  const session = await getSession();
  // ... 권한 체크
  
  // 비밀번호 변경 시 현재 비밀번호 확인
  if (validatedData.newPassword) {
    if (!validatedData.currentPassword) {
      return NextResponse.json({ success: false, message: '현재 비밀번호를 입력해주세요' }, { status: 400 });
    }
    const isPasswordValid = await verifyPassword(validatedData.currentPassword, member.passwordHash);
    // ... 비밀번호 검증
  }
  
  // 이메일/닉네임 중복 체크 (현재 사용자 제외)
  if (validatedData.email && validatedData.email !== member.email) {
    const existingMember = await findMemberByEmail(validatedData.email);
    if (existingMember && existingMember.id !== memberId) {
      return NextResponse.json({ success: false, message: '이미 사용 중인 이메일입니다' }, { status: 400 });
    }
  }
  // ... 프로필 업데이트
}
```

#### 2.2 프로필 수정 페이지 구현

**파일**: `app/(main)/profile/page.tsx`

- React Hook Form을 사용한 폼 관리
- 실시간 유효성 검사
- 비밀번호 변경은 선택 사항으로 처리

#### 2.3 사용자 메뉴에 프로필 링크 추가

**파일**: `components/auth/UserMenu.tsx`

```typescript
<Link 
  href="/profile" 
  className="text-gray-700 hover:text-gray-900 font-medium cursor-pointer"
>
  {user.nickname}님
</Link>
```

### 핵심 포인트
- **보안**: 현재 비밀번호 확인 후에만 비밀번호 변경 허용
- **중복 체크**: 현재 사용자를 제외한 중복 검사
- **세션 업데이트**: 이메일 변경 시 세션 갱신 필요
- **사용자 경험**: 이름 클릭으로 프로필 수정 페이지 접근

### 다른 프로젝트 적용 시
- 프로필 수정 API는 인증된 사용자만 접근 가능하도록 구현
- 중복 체크 시 현재 사용자 ID를 제외하는 로직 필수
- 비밀번호 변경은 별도 검증 프로세스 필요

---

## 3. 회원 게시글 작성자 표시 문제

### 문제 상황
회원이 게시글을 작성했는데도 "비회원"으로 표시됨. `authorName` 필드가 비어있어서 발생.

### 원인
게시글 작성 시 `memberId`만 저장하고 `authorName`을 저장하지 않음. 조회 시 `authorName`이 없으면 "비회원"으로 표시됨.

### 해결 방법

#### 3.1 게시글 작성 시 닉네임 저장

**파일**: `app/api/posts/route.ts`

```typescript
const memberId = await getCurrentMemberId();

// 회원인 경우 닉네임 가져오기
let authorName: string | undefined;
if (memberId) {
  const member = await findMemberById(memberId);
  if (member) {
    authorName = member.nickname;
  }
}

const post = await createPost({
  boardId: validatedData.boardId,
  memberId: memberId || undefined,
  authorName, // 닉네임 저장
  // ... 기타 필드
});
```

#### 3.2 댓글 작성 시 닉네임 저장

**파일**: `app/api/comments/route.ts`

동일한 로직 적용

#### 3.3 기존 데이터 조회 시 닉네임 표시

**파일**: `lib/db/posts.ts`

```typescript
export async function findPostById(id: number): Promise<BoardPost | null> {
  const [post] = await db.select().from(boardPosts)
    .where(and(eq(boardPosts.id, id), eq(boardPosts.status, 1)))
    .limit(1);
  
  if (!post) return null;
  
  // authorName이 없고 memberId가 있으면 회원 닉네임 가져오기
  if (!post.authorName && post.memberId) {
    const [member] = await db.select({ nickname: members.nickname })
      .from(members)
      .where(eq(members.id, post.memberId))
      .limit(1);
    
    if (member) {
      post.authorName = member.nickname;
    }
  }
  
  return post;
}

// 목록 조회 시 배치 처리로 성능 최적화
export async function findPostsByBoard(...): Promise<{ posts: BoardPost[]; total: number }> {
  const posts = await finalQuery.limit(limit).offset(offset);

  // authorName이 없고 memberId가 있는 게시글들의 memberId 수집
  const memberIdsToFetch = posts
    .filter(post => !post.authorName && post.memberId)
    .map(post => post.memberId!)
    .filter((id, index, arr) => arr.indexOf(id) === index); // 중복 제거

  // 배치로 회원 정보 가져오기
  let memberNicknames: Record<number, string> = {};
  if (memberIdsToFetch.length > 0) {
    const membersData = await db.select({ id: members.id, nickname: members.nickname })
      .from(members)
      .where(inArray(members.id, memberIdsToFetch));
    
    membersData.forEach(member => {
      memberNicknames[member.id] = member.nickname;
    });
  }

  // authorName이 없는 게시글에 회원 닉네임 설정
  posts.forEach(post => {
    if (!post.authorName && post.memberId && memberNicknames[post.memberId]) {
      post.authorName = memberNicknames[post.memberId];
    }
  });

  return { posts, total: count };
}
```

### 핵심 포인트
- **데이터 일관성**: 작성 시점에 `authorName` 저장으로 데이터 일관성 유지
- **하위 호환성**: 기존 데이터를 위한 조회 시점 보완 로직
- **성능 최적화**: 목록 조회 시 N+1 문제 방지를 위한 배치 처리
- **데이터 정규화 vs 비정규화**: 조회 성능을 위해 `authorName` 필드 유지 (닉네임 변경 시에도 작성 당시 닉네임 유지)

### 다른 프로젝트 적용 시
- **작성 시점 저장**: 외래키 관계가 있어도 조회 성능을 위해 필요한 데이터는 저장
- **배치 조회**: 목록 조회 시 N+1 문제 방지를 위한 배치 처리 필수
- **마이그레이션 전략**: 기존 데이터 호환성을 위한 조회 시점 보완 로직

---

## 4. 다크 스킨 시스템 구현

### 문제 상황
다크 모드 스킨이 없어 사용자가 원하는 테마를 선택할 수 없음.

### 해결 방법

#### 4.1 다크 스킨 CSS 파일 생성

**파일**: `app/skins/dark.css`

```css
/* 다크 스킨 스타일 */
.skin-dark {
  --bg-primary: #111827;
  --bg-secondary: #1F2937;
  --bg-tertiary: #374151;
  --text-primary: #F9FAFB;
  --text-secondary: #D1D5DB;
  --text-tertiary: #9CA3AF;
  --border-color: #374151;
  --border-hover: #4B5563;
  --shadow: rgba(0, 0, 0, 0.3);
}

.skin-dark .bg-white {
  background-color: var(--bg-secondary) !important;
}

.skin-dark .text-gray-900 {
  color: var(--text-primary) !important;
}

/* ... 기타 스타일 오버라이드 */
```

#### 4.2 스킨 데이터베이스 함수 구현

**파일**: `lib/db/skins.ts`

```typescript
export async function findSkinById(id: number): Promise<Skin | null> {
  const [skin] = await db.select().from(skins).where(eq(skins.id, id)).limit(1);
  return skin || null;
}

export async function findSkinByKey(skinKey: string): Promise<Skin | null> {
  const [skin] = await db.select().from(skins).where(eq(skins.skinKey, skinKey)).limit(1);
  return skin || null;
}
```

#### 4.3 레이아웃에 스킨 CSS 로드

**파일**: `app/layout.tsx`

```typescript
import "./globals.css";
import "./skins/dark.css";
```

#### 4.4 페이지에서 스킨 클래스 적용

**파일**: `app/(main)/board/[boardKey]/page.tsx`

```typescript
const board = await findBoardByKey(params.boardKey);
const skin = board.skinId ? await findSkinById(board.skinId) : null;
const skinClass = skin ? `skin-${skin.skinKey}` : '';

return (
  <div className={`min-h-screen bg-gray-50 ${skinClass}`}>
    {/* ... */}
  </div>
);
```

#### 4.5 스킨 적용 API 구현

**파일**: `app/api/admin/skins/[id]/apply/route.ts`

```typescript
export async function POST(request: NextRequest, { params }: { params: { id: string } }) {
  // 관리자 권한 체크
  if (!(await isAdmin())) {
    return NextResponse.json({ success: false, message: '관리자만 적용할 수 있습니다' }, { status: 403 });
  }

  const skinId = parseInt(params.id, 10);
  const { boardId } = await request.json();

  if (boardId) {
    // 특정 게시판에 스킨 적용
    await updateBoard(boardId, { skinId });
  } else {
    // 모든 게시판에 스킨 적용
    const boards = await findAllBoards(false);
    for (const board of boards) {
      await updateBoard(board.id, { skinId });
    }
  }

  return NextResponse.json({ success: true, message: '스킨이 적용되었습니다' });
}
```

#### 4.6 데이터베이스 초기화 스크립트 수정

**파일**: `scripts/setup-db.ts`

```typescript
const [darkSkin] = await db.insert(skins).values({
  name: 'Dark Skin',
  skinKey: 'dark',
  description: '다크 모드 스킨',
  version: '1.0.0',
  author: 'One Board',
  isSystem: 1,
  isActive: 1,
}).returning();
```

### 핵심 포인트
- **CSS 변수 활용**: 테마 색상을 CSS 변수로 관리하여 유지보수 용이
- **클래스 기반 적용**: `.skin-{skinKey}` 클래스로 스킨 구분
- **!important 사용**: Tailwind 클래스 오버라이드를 위해 필요 (선택적)
- **게시판별 스킨**: 게시판마다 다른 스킨 적용 가능
- **관리자 인터페이스**: 관리자가 쉽게 스킨을 적용할 수 있는 UI 제공

### 다른 프로젝트 적용 시
- **테마 시스템 설계**: CSS 변수와 클래스 기반 접근 방식 고려
- **성능**: 필요한 스킨 CSS만 로드하는 동적 로딩 고려
- **확장성**: 새로운 스킨 추가가 쉬운 구조 설계
- **사용자 선택**: 사용자가 개인적으로 스킨을 선택할 수 있는 기능 고려

---

## 5. 회원 가입 에러 수정

### 문제 상황
회원 가입 시 "회원가입 중 오류가 발생했습니다" 에러가 발생하고, 이메일 필드가 선택사항인데도 validation 에러가 발생함.

### 원인
1. 이메일 필드에 unique 인덱스가 있는데 빈 문자열이 저장되면 unique 제약 위반 발생
2. 이메일 validation 스키마가 빈 문자열을 제대로 처리하지 못함
3. DB에 `username` 컬럼이 없어서 "no such column: username" 에러 발생

### 해결 방법

#### 5.1 DB에 username 컬럼 추가

```sql
ALTER TABLE members ADD COLUMN username text;
CREATE UNIQUE INDEX IF NOT EXISTS uk_members_username ON members (username);
```

#### 5.2 이메일 validation 개선

**파일**: `app/api/auth/register/route.ts`

```typescript
// 수정 전
email: z.string().email('올바른 이메일 주소를 입력해주세요').optional().or(z.literal('')),

// 수정 후
email: z.union([
  z.string().email('올바른 이메일 주소를 입력해주세요'),
  z.literal(''),
]).optional().transform((val) => val === '' ? undefined : val),
```

#### 5.3 빈 문자열을 null로 변환

**파일**: `lib/db/members.ts`

```typescript
export async function createMember(data: {
  username: string;
  email?: string;
  password: string;
  nickname: string;
  name?: string;
  phone?: string;
}): Promise<Member> {
  const passwordHash = await bcrypt.hash(data.password, 10);
  
  // 빈 문자열을 null로 변환 (unique 제약 위반 방지)
  const email = data.email && data.email.trim() !== '' ? data.email : null;
  const name = data.name && data.name.trim() !== '' ? data.name : null;
  const phone = data.phone && data.phone.trim() !== '' ? data.phone : null;
  
  const [member] = await db.insert(members).values({
    username: data.username,
    email: email,
    // ... 기타 필드
  }).returning();

  return member;
}
```

#### 5.4 에러 처리 개선

**파일**: `app/api/auth/register/route.ts`

```typescript
} catch (error) {
  if (error instanceof z.ZodError) {
    return NextResponse.json(
      { success: false, message: error.errors[0].message },
      { status: 400 }
    );
  }

  console.error('Register error:', error);
  
  // 에러 메시지 추출
  let errorMessage = '회원가입 중 오류가 발생했습니다';
  if (error instanceof Error) {
    errorMessage = error.message;
  }

  // DB 에러인 경우 더 구체적인 메시지 제공
  if (errorMessage.includes('UNIQUE constraint')) {
    if (errorMessage.includes('username')) {
      errorMessage = '이미 사용 중인 아이디입니다';
    } else if (errorMessage.includes('email')) {
      errorMessage = '이미 사용 중인 이메일입니다';
    } else if (errorMessage.includes('nickname')) {
      errorMessage = '이미 사용 중인 닉네임입니다';
    }
  }

  return NextResponse.json(
    { success: false, message: errorMessage },
    { status: 500 }
  );
}
```

### 핵심 포인트
- **NULL vs 빈 문자열**: SQLite에서 unique 인덱스는 NULL 값은 여러 개 허용하지만 빈 문자열은 중복 불가
- **Validation 개선**: `z.union`과 `transform`을 사용하여 빈 문자열을 `undefined`로 변환
- **에러 메시지**: 사용자에게 친화적인 에러 메시지 제공
- **DB 스키마**: 스키마 변경 시 마이그레이션 필요

### 다른 프로젝트 적용 시
- 선택적 필드에 unique 제약이 있을 때는 NULL을 사용
- 빈 문자열과 NULL을 구분하여 처리
- 에러 메시지를 구체적으로 제공하여 디버깅 용이

---

## 6. 로그인 시스템 개선 (username 기반)

### 문제 상황
로그인 페이지는 이메일 필드를 사용하지만, 로그인 API는 username을 기대하여 로그인이 실패함.

### 원인
프론트엔드와 백엔드의 필드명 불일치.

### 해결 방법

**파일**: `app/(auth)/login/page.tsx`

```typescript
// 수정 전
const loginSchema = z.object({
  email: z.string().email('올바른 이메일 주소를 입력해주세요'),
  password: z.string().min(1, '비밀번호를 입력해주세요'),
});

// 수정 후
const loginSchema = z.object({
  username: z.string().min(1, '아이디를 입력해주세요'),
  password: z.string().min(1, '비밀번호를 입력해주세요'),
});
```

```typescript
// 수정 전
<label htmlFor="email">이메일</label>
<input {...register('email')} type="email" />

// 수정 후
<label htmlFor="username">아이디</label>
<input {...register('username')} type="text" autoComplete="username" />
```

### 관리자 계정 정보
- **아이디**: `admin`
- **비밀번호**: `admin123!`
- **이메일**: `admin@oneboard.com`

### 핵심 포인트
- **일관성**: 프론트엔드와 백엔드의 필드명 일치 필수
- **사용자 경험**: 아이디 기반 로그인이 더 직관적
- **초기 계정**: 관리자 계정에 username 설정 필요

---

## 7. 게시글 수정 권한 체크 및 버튼 비활성화

### 문제 상황
수정 권한이 없는 사용자도 수정 버튼이 표시되어 클릭 시 에러가 발생함.

### 해결 방법

**파일**: `app/(main)/board/[boardKey]/[postId]/page.tsx`

```typescript
import { getCurrentMemberId, getCurrentMemberRole } from '@/lib/auth/permissions';

export default async function PostPage({ params }: { params: { boardKey: string; postId: string } }) {
  // ... 게시글 조회 로직

  // 수정 권한 체크 (작성자 본인 또는 관리자)
  const memberId = await getCurrentMemberId();
  const userRole = await getCurrentMemberRole();
  const canEdit = post.memberId === memberId || userRole === 2;

  return (
    <div>
      {/* ... 게시글 내용 */}
      
      <div className="flex gap-2">
        <Link href={`/board/${board.boardKey}`}>목록</Link>
        {canEdit ? (
          <>
            <Link href={`/write?board=${board.id}&post=${post.id}`}>수정</Link>
            <DeletePostButton postId={post.id} boardKey={board.boardKey} />
          </>
        ) : (
          <button
            disabled
            className="px-4 py-2 bg-gray-300 text-gray-500 rounded-md cursor-not-allowed"
            title="수정 권한이 없습니다"
          >
            수정
          </button>
        )}
      </div>
    </div>
  );
}
```

### 핵심 포인트
- **권한 체크**: 작성자 본인(`post.memberId === memberId`) 또는 관리자(`userRole === 2`)만 수정 가능
- **UI 개선**: 권한이 없으면 버튼을 비활성화하여 사용자 혼란 방지
- **관리자 권한**: 관리자는 모든 게시글 수정 가능

### 다른 프로젝트 적용 시
- 권한 체크를 서버 컴포넌트에서 수행
- 권한이 없을 때는 버튼을 숨기거나 비활성화
- 툴팁으로 권한이 없는 이유 표시

---

## 8. 게시글 목록 정렬 순서 개선

### 문제 상황
게시글 목록에서 최신 글이 위로 오지 않음.

### 원인
정렬 순서가 제대로 적용되지 않거나, 공지사항/고정글 정렬이 최신순 정렬을 덮어씀.

### 해결 방법

**파일**: `lib/db/posts.ts`

```typescript
// 수정 전
const orderedQuery = order === 'desc' 
  ? query.orderBy(desc(orderColumn))
  : query.orderBy(asc(orderColumn));

// 공지사항과 고정글을 먼저 표시
const finalQuery = orderedQuery.orderBy(
  desc(boardPosts.isNotice),
  desc(boardPosts.isPinned)
);

// 수정 후
// 공지사항과 고정글을 먼저 표시한 후, 선택한 기준으로 정렬
const finalQuery = query.orderBy(
  desc(boardPosts.isNotice),
  desc(boardPosts.isPinned),
  order === 'desc' ? desc(orderColumn) : asc(orderColumn)
);
```

**파일**: `app/(main)/board/[boardKey]/page.tsx`

```typescript
const { posts, total } = await findPostsByBoard(board.id, {
  page,
  limit: 20,
  orderBy: (searchParams.orderBy as 'createdAt' | 'viewCount' | 'commentCount') || 'createdAt',
  order: (searchParams.order as 'asc' | 'desc') || 'desc', // 기본값: 최신순
  // ... 기타 옵션
});
```

### 정렬 순서
1. 공지사항(`isNotice`) 우선 표시
2. 고정글(`isPinned`) 그 다음 표시
3. 일반 게시글은 선택한 기준(기본: 작성일)으로 정렬

### 핵심 포인트
- **다중 정렬**: `orderBy`를 여러 번 호출하여 우선순위 정렬
- **기본값**: 최신순(`desc`)이 기본값
- **공지/고정글**: 항상 최상단에 표시

### 다른 프로젝트 적용 시
- 정렬 우선순위를 명확히 정의
- 사용자가 정렬 기준을 선택할 수 있도록 UI 제공
- 중요한 항목(공지, 고정)은 항상 상단에 표시

---

## 9. 게시글 내용 줄바꿈 무시 문제

### 문제 상황
게시글 작성 시 에디터에서 줄바꿈(Enter)을 입력했음에도 불구하고, 실제 상세 페이지에서는 내용이 한 줄로 뭉쳐서 출력됨.

### 원인
1. **TipTap SSR Hydration 오류**: Next.js App Router(SSR) 환경에서 에디터가 서버와 클라이언트 간의 렌더링 차이로 인해 초기화가 제대로 되지 않아, HTML 구조(<p> 태그 등)를 생성하지 못하고 평문(Plain Text)으로 저장되는 경우가 발생함.
2. **CSS 공백 처리 규칙**: Tailwind CSS의 Typography 도구(`.prose`)나 브라우저 기본값이 연속된 공백이나 줄바꿈을 하나로 병합(collapse)하여 렌더링함.

### 해결 방법

#### 9.1 에디터 설정 수정 (SSR 대응)

**파일**: `components/board/Editor.tsx`

```typescript
const editor = useEditor({
  extensions: [...],
  content,
  immediatelyRender: false, // 클라이언트 측에서만 렌더링되도록 강제하여 하이드레이션 오류 방지
  onUpdate: ({ editor }) => {
    onChange(editor.getHTML());
  },
});
```

#### 9.2 CSS 공백 처리 방식 변경

**파일**: `app/globals.css`

```css
.prose {
  max-width: 100%;
  color: var(--color-gray-700);
  line-height: 1.8;
  font-size: 1.125rem;
  white-space: pre-wrap; /* 줄바꿈(\n)을 포함한 공백 문자를 그대로 유지하며 자동 줄바꿈 지원 */
}

/* 단락 요소에 대한 명확한 블록 처리 */
.prose p {
  display: block !important;
  margin-top: 1.25rem !important;
  margin-bottom: 1.25rem !important;
}
```

### 핵심 포인트
- **WYSIWYG 에디터와 SSR**: Next.js 환경에서 에디터 라이브러리 사용 시 `immediatelyRender`와 같은 SSR 방지 옵션 확인 필수.
- **`white-space: pre-wrap`**: HTML 구조가 깨지거나 평문으로 저장된 경우에도 사용자 입력 형식을 최대한 유지하기 위한 안전 장치.
- **CSS 우선순위**: Tailwind 스타일이 다른 전역 스타일과 충돌할 수 있으므로 명시적인 스타일 지정이 필요할 수 있음.

### 다른 프로젝트 적용 시
- SSR 환경에서 서드파티 UI 컴포넌트 사용 시 클라이언트 전용 초기화 로직 확인.
- 사용자 생성 콘텐츠(UGC) 출력 시 공백 및 줄바꿈 처리 정책 결정 (`pre-wrap` vs `p` 태그 래핑).

---

## 10. Better-SQLite3 트랜잭션과 비동기 처리 문제

### 문제 상황
댓글 작성 시 `Internal Server Error`가 발생하고, 서버 로그에 `TypeError: Transaction function cannot return a promise` 오류가 출력됨.

### 원인
`better-sqlite3`는 동기식 드라이버이므로 트랜잭션 함수 내부에서 `async/await`을 사용할 수 없음. Drizzle ORM과 함께 사용할 때도 트랜잭션 콜백은 동기식이어야 함. 또한, `returning()`의 결과를 배열 인덱스로 접근할 때 메서드 체이닝(`returning().get()`) 미사용으로 인해 값이 올바르게 반환되지 않음.

### 해결 방법

**파일**: `lib/db/comments.ts`

```typescript
// 수정 전 (오류 발생)
export async function createComment(data: CreateCommentInput) {
    return await db.transaction(async (tx) => {
        const result = await tx.insert(comments).values({...}).returning();
        await tx.update(posts)...;
        return result[0];
    });
}

// 수정 후 (정상 작동)
export async function createComment(data: CreateCommentInput) {
    return db.transaction((tx) => {
        // 동기식으로 실행 및 .get() 사용
        const result = tx.insert(comments).values({...}).returning().get();
        
        tx.update(posts)
            .set({ commentCount: sql`${posts.commentCount} + 1` })
            .where(eq(posts.id, data.postId))
            .run(); // .run() 명시적 호출
            
        return result; // 객체 직접 반환
    });
}
```

### 핵심 포인트
- **동기식 트랜잭션**: `better-sqlite3` 사용 시 트랜잭션 내부에서 `await` 금지.
- **메서드 체이닝**: `.returning()` 뒤에 `.get()` 또는 `.all()`을 사용하여 결과값을 명시적으로 가져와야 함.
- **명시적 실행**: `update`, `delete` 등의 쿼리는 `.run()`을 호출하여 실행해야 함.

---

## 11. 스킨 관리 페이지 404 에러 및 구현

### 문제 상황
`/admin/skins` 페이지 접속 시 404 페이지가 표시됨.

### 원인
해당 경로에 대한 페이지 컴포넌트(`page.tsx`)가 구현되지 않았음.

### 해결 방법
1. **페이지 생성**: `app/admin/skins/page.tsx` 생성.
2. **테이블 컴포넌트**: `components/admin/SkinListTable.tsx` 구현.
3. **API 라우트**: 스킨 상태 변경을 위한 `app/api/admin/skins/[id]/status/route.ts` 구현.
4. **메뉴 연결**: `app/admin/layout.tsx` 사이드바에 링크 추가.

---

## 12. Tiptap 에디터 이미지 붙여넣기 및 리사이즈 오류 해결

### 문제 상황
`tiptap-extension-resize-image` 라이브러리 적용 후, 클립보드 이미지 붙여넣기와 드래그 앤 드롭이 작동하지 않는 현상. "이미지가 업로드되었습니다" 메시지는 표시되나 에디터에 삽입되지 않음.

### 원인 분석
1.  **노드 이름 불일치**: 해당 확장은 기본 `image` 노드 대신 `imageResize`라는 노드 이름을 사용함. 기존 `type: 'image'` 코드는 무시됨.
2.  **`editorRef` 참조 불안정**: 비동기 핸들러(`handlePaste`) 내에서 `editorRef.current` 참조가 유실되거나 타이밍 문제로 실행되지 않음.

### 해결 방법

**파일**: `components/board/Editor.tsx`

**1. 노드 이름 변경 (`image` → `imageResize`)**
- 이미지 삽입 시 명시적으로 `imageResize` 타입을 사용해야 함.

**2. 이미지 삽입 방식 개선 (`editorRef` → `view.dispatch`)**
- `editorRef` 대신 ProseMirror의 `view` 객체를 직접 사용하여 트랜잭션을 실행.

```typescript
// [변경 전] 작동하지 않는 코드
editorRef.current.chain().focus().insertContent({
    type: 'image', // 잘못된 노드 이름
    attrs: { src: result.url, ... }
}).run();

// [변경 후] 해결된 코드
const { schema } = view.state;
const node = schema.nodes.imageResize.create({ // 올바른 노드 이름 (imageResize)
    src: result.url,
    alt: result.originalName,
});
const tr = view.state.tr.replaceSelectionWith(node); // view 객체를 사용해 직접 삽입
view.dispatch(tr);
```

### 핵심 포인트
- Tiptap 확장 라이브러리 사용 시, 해당 확장이 정의하는 정확한 **노드 이름(Node Name)**을 확인해야 함.
- 이벤트 핸들러 내부에서는 외부 Ref보다 내부 **`view` 객체**를 사용하는 것이 안정적임.

---

## 13. 관리자 회원 비밀번호 초기화 기능

### 문제 상황
관리자가 회원의 비밀번호를 분실했을 때 초기화해줄 수 있는 기능이 부재함.

### 해결 방법

**1. API 구현**: `app/api/admin/members/[id]/password/route.ts`
- 관리자 권한 확인
- `bcryptjs`를 사용한 새 비밀번호 해싱 및 DB 업데이트

**2. UI 구현**: `components/admin/MemberListTable.tsx`
- 회원 목록 테이블에 'P/W' 변경 버튼 추가
- 모달 창을 통해 새 비밀번호 입력 및 API 호출

### 핵심 포인트
- **보안**: 비밀번호 변경 API는 반드시 관리자 권한(`role === 2`)을 체크해야 함.
- **예외 처리**: 다른 관리자의 비밀번호는 변경할 수 없도록 UI 및 API에서 차단.

---

## 14. 관리자 페이지 리다이렉션 (404 해결)

### 문제 상황
`/admin` 경로로 접속 시 해당 페이지(`page.tsx`)가 존재하지 않아 404 에러가 발생함. 사용자는 `/admin/dashboard`로 자동 이동하기를 기대함.

### 해결 방법

**파일**: `app/admin/page.tsx`

```typescript
import { redirect } from "next/navigation";

export default function AdminPage() {
  redirect("/admin/dashboard");
}
```

### 핵심 포인트
- **루트 리다이렉트**: 하위 경로가 메인인 경우, 상위 경로 접속 시 `redirect()` 함수를 사용하여 자동으로 이동시킴.

---

## 일반적인 문제 해결 패턴

### 1. 데이터 일관성 문제
- **문제**: 작성 시점에 필요한 데이터를 저장하지 않음
- **해결**: 외래키 관계가 있어도 조회 성능을 위해 필요한 데이터를 저장
- **예시**: 게시글 작성 시 회원 닉네임 저장, 댓글 작성 시 작성자 정보 저장

### 2. 동기 vs 비동기 드라이버 차이
- **문제**: DB 드라이버 특성에 맞지 않는 비동기 처리
- **해결**: 사용 중인 드라이버(better-sqlite3 등)의 동기/비동기 지원 여부 확인 및 문법 준수
- **예시**: 트랜잭션 내부 `async/await` 제거 및 `.run()`, `.get()` 사용

### 3. 인증 및 권한 관리
- **문제**: 페이지/API 레벨에서의 접근 제어 누락
- **해결**: 미들웨어 및 컴포넌트 내부에서 이중 권한 체크
- **예시**: 관리자 전용 페이지 접근 시 role 확인 및 리다이렉션

---

## 체크리스트

다른 프로젝트에 적용할 때 확인할 사항:

- [ ] DB 드라이버(SQLite, PostgreSQL 등)의 동기/비동기 특성을 고려하여 코드를 작성했는가?
- [ ] 트랜잭션 처리 시 롤백 및 에러 핸들링이 올바르게 구현되었는가?
- [ ] 관리자 페이지 등 주요 기능에 대한 라우팅 및 컴포넌트가 모두 구현되었는가?
- [ ] API 에러 응답이 프론트엔드에서 적절히 처리되고 사용자에게 피드백을 주는가?

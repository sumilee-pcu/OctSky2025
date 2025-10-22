# Supabase 설정 가이드

이 프로젝트는 여러 기기에서 실시간으로 팀 점수를 공유하기 위해 Supabase를 사용합니다.

## 1. Supabase 프로젝트 생성

1. https://supabase.com 접속
2. "Start your project" 클릭
3. GitHub 계정으로 로그인
4. "New project" 클릭
5. 프로젝트 이름: `octsky2025`
6. Database Password 설정 (안전한 비밀번호 사용)
7. Region: Northeast Asia (Seoul) 선택
8. "Create new project" 클릭

## 2. 데이터베이스 테이블 생성

프로젝트가 생성되면:

1. 왼쪽 메뉴에서 "SQL Editor" 클릭
2. "New query" 클릭
3. 아래 SQL 코드를 붙여넣기:

```sql
-- team_scores 테이블 생성
CREATE TABLE team_scores (
  id BIGSERIAL PRIMARY KEY,
  session_id TEXT NOT NULL,
  team_index INTEGER NOT NULL,
  team_name TEXT NOT NULL,
  score INTEGER DEFAULT 0,
  combo INTEGER DEFAULT 0,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(session_id, team_index)
);

-- 인덱스 생성 (빠른 검색을 위해)
CREATE INDEX idx_team_scores_session ON team_scores(session_id);

-- Row Level Security (RLS) 활성화
ALTER TABLE team_scores ENABLE ROW LEVEL SECURITY;

-- 모든 사용자가 읽고 쓸 수 있도록 정책 설정 (데모용)
CREATE POLICY "Enable read access for all users" ON team_scores
  FOR SELECT USING (true);

CREATE POLICY "Enable insert access for all users" ON team_scores
  FOR INSERT WITH CHECK (true);

CREATE POLICY "Enable update access for all users" ON team_scores
  FOR UPDATE USING (true);

CREATE POLICY "Enable delete access for all users" ON team_scores
  FOR DELETE USING (true);
```

4. "Run" 버튼 클릭하여 실행

## 3. Supabase Realtime 활성화

1. 왼쪽 메뉴에서 "Database" 클릭
2. "Replication" 탭 선택
3. `team_scores` 테이블 옆의 스위치를 켜서 Realtime 활성화

## 4. API 키 가져오기

1. 왼쪽 메뉴에서 "Settings" 클릭
2. "API" 클릭
3. 다음 값을 복사:
   - **Project URL** (예: `https://abcdefghij.supabase.co`)
   - **anon public** key

## 5. index.html 파일 수정

`index.html` 파일에서 다음 부분을 찾아서 수정:

```javascript
// Supabase 초기화 (데모용 공개 프로젝트)
const SUPABASE_URL = 'https://xyzcompany.supabase.co';  // 여기를 Project URL로 변경
const SUPABASE_ANON_KEY = 'eyJhbG...';  // 여기를 anon public key로 변경
```

예시:
```javascript
const SUPABASE_URL = 'https://abcdefghij.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFiY2RlZmdoaWoiLCJyb2xlIjoiYW5vbiIsImlhdCI6MTY0MjQ4MDAwMCwiZXhwIjoxOTU4MDU2MDAwfQ.실제키값...';
```

## 6. 배포 및 테스트

1. 변경사항을 GitHub에 푸시
2. Vercel이 자동으로 배포
3. 여러 기기에서 같은 URL 접속
4. 가족 대항전에서 점수가 실시간으로 동기화되는지 확인

## 작동 원리

- **세션 코드**: 각 가족/그룹마다 고유한 세션 ID가 자동 생성됩니다
- **실시간 동기화**: 한 기기에서 점수를 기록하면 다른 모든 기기에 즉시 반영
- **localStorage**: 세션 ID가 브라우저에 저장되어 재접속 시에도 같은 세션 유지

## 참고사항

- Supabase 무료 플랜은 월 500MB 데이터베이스, 5GB 대역폭 제공
- 가족용으로는 충분하며, 많은 사용자가 접속해도 무료 플랜으로 가능
- 보안을 강화하려면 RLS 정책을 수정하여 세션별 접근 제한 가능

# 연자 사진 등록 페이지

로그인 없이, 페이지 주소를 아는 사람은 누구나 연자 사진을 등록/수정할 수 있는 정적 페이지입니다.
사진 파일은 Supabase Storage(무료)에 저장됩니다. GitHub Pages로 배포합니다.

## 1. Supabase 프로젝트 준비 (한 번만)

1. https://supabase.com 에서 무료 계정 생성 후 새 프로젝트 생성 (리전은 Northeast Asia 계열 추천)
2. 프로젝트 생성 후 왼쪽 메뉴 **Storage** → **New bucket**
   - 이름: `speaker-photos`
   - **Public bucket** 체크 (사진을 목록에서 바로 볼 수 있어야 하므로 필수)
3. 왼쪽 메뉴 **SQL Editor** → New query 에 아래 SQL 붙여넣고 실행 (로그인 없이 업로드를 허용하는 정책)

```sql
-- 누구나 읽기 가능 (버킷이 Public이면 사실상 자동이지만 명시적으로 추가)
create policy "Public read speaker-photos"
on storage.objects for select
using ( bucket_id = 'speaker-photos' );

-- 누구나(로그인 없이) 업로드 가능
create policy "Anon can upload speaker-photos"
on storage.objects for insert
to anon
with check ( bucket_id = 'speaker-photos' );

-- 누구나(로그인 없이) 기존 파일 덮어쓰기(수정) 가능
create policy "Anon can update speaker-photos"
on storage.objects for update
to anon
using ( bucket_id = 'speaker-photos' )
with check ( bucket_id = 'speaker-photos' );
```

4. 왼쪽 메뉴 **Project Settings → API** 에서 아래 두 값을 복사:
   - **Project URL** (예: `https://xxxxxxxx.supabase.co`)
   - **anon public** key (긴 문자열, 공개해도 되는 키입니다)

## 2. 페이지에 값 채워넣기

`index.html` 상단의 아래 부분을 실제 값으로 교체합니다.

```js
const SUPABASE_URL = "https://YOUR-PROJECT-REF.supabase.co";
const SUPABASE_ANON_KEY = "YOUR-ANON-PUBLIC-KEY";
```

> anon public 키는 클라이언트(브라우저)에 노출되는 것이 정상입니다. 실제 접근 제어는
> 위 SQL 정책(RLS)이 담당합니다. 절대로 **service_role** 키는 여기에 넣지 마세요.

## 3. GitHub Pages 배포

```bash
cd speaker-photos
git init
git add index.html README.md
git commit -m "Add speaker photo upload page"
git branch -M main
git remote add origin https://github.com/<계정>/<저장소명>.git
git push -u origin main
```

그 다음 GitHub 저장소 **Settings → Pages** 에서:
- Source: `Deploy from a branch`
- Branch: `main` / `(root)`

몇 분 후 `https://<계정>.github.io/<저장소명>/` 주소로 접속됩니다. 이 주소를 연자들에게 공유하세요.

## 4. 사용 방법

- **연자**: 페이지 접속 → 본인 이름 아래 "사진 등록" 버튼 클릭 → 사진 선택. 이미 등록되어 있으면 버튼이
  "사진 수정"으로 바뀌며, 다시 올리면 기존 사진이 새 사진으로 자동 교체됩니다.
- **전체 다운로드**: 우측 상단 "전체 사진 다운로드" 버튼 → 비밀번호 `design1234` 입력 → 등록된 사진을
  모두 zip으로 묶어 다운로드합니다. (비밀번호는 페이지 소스에 평문으로 저장하지 않고 SHA-256 해시로만
  비교합니다. 다만 정적 페이지 특성상 완전한 보안은 아니며, 가벼운 접근 제한 용도입니다.)

## 5. 연자 명단 수정이 필요할 때

`index.html` 안의 `SPEAKERS` 배열에서 이름/소속을 직접 수정하면 됩니다. 각 연자는 고유 `id`
(`spk01`, `spk02`, ...)로 사진 파일명이 정해지므로, 이미 등록된 사진과의 매칭을 유지하려면
기존 연자의 `id`는 바꾸지 마세요. 새 연자를 추가할 때는 새로운 `id`(예: `spk28`)를 사용하세요.

# 레슨 출석 (#03) — 배포·연결 순서

지금 상태: Firebase 설정이 비어 있어서 **데모 모드**(이 기기에만 저장)로 동작해요.
아래 순서대로 하면 실제로 원장쌤·레슨생이 같이 쓰는 앱이 돼요.

## 1. GitHub 저장소 만들고 올리기
1. GitHub에서 새 저장소 `lesson-attend` (Public) 만들기
2. 이 폴더의 파일 7개 올리기: `index.html`, `sw.js`, `manifest.webmanifest`, `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `README.md`
   (`firestore-rules.txt`는 올리지 않아도 돼요)
3. Settings → Pages → Branch `main` / `root` → Save
4. 주소: `https://leehidecember.github.io/lesson-attend/`

## 2. Firebase 연결 (#01 레슨 시간표와 같은 프로젝트 `lesson-schedule-16d8b`)
1. #01 `index.html`에서 `firebaseConfig = { ... }` 내용을 복사
2. 이 앱 `index.html` 위쪽 설정 두 줄을 채우기
   ```js
   const FIREBASE_CONFIG = { apiKey: "...", authDomain: "...", projectId: "lesson-schedule-16d8b", ... };
   const ADMIN_EMAILS = ["원장쌤 이메일", "내 이메일"];   // #01 보안 규칙에 있는 두 이메일
   ```
3. Firebase 콘솔 → Firestore → **규칙** 탭 → `firestore-rules.txt`의 블록을
   `match /databases/{database}/documents {` 안쪽에 **추가**(기존 #01 규칙은 그대로) → 이메일 2개 바꾸기 → 게시
4. Authentication → 로그인 방법: 이메일/비밀번호가 켜져 있는지 확인 (#01에서 이미 켬)
   - 승인된 도메인 `leehidecember.github.io`도 #01에서 이미 추가돼 있어요
5. `sw.js`의 `lesson-attend-v1` 숫자를 올리고 다시 올리기 (고칠 때마다)

> 같은 주소·같은 Firebase라서, 원장쌤이 #01에 로그인돼 있으면 이 앱도 자동으로 로그인돼요.

## 3. 처음 쓰는 법
- 원장: 이메일로 로그인 → 레슨생 탭 → **+ 추가** → 초대코드 생성 → **초대 메시지 보내기**(카톡 공유)
- 레슨생: 앱 열기 → **초대코드로 가입** → 코드 입력 → 비밀번호만 정하기 (아이디 = 이름)
- 다음부터 레슨생은 **이름 + 비밀번호**로 로그인
- 비밀번호 분실: 원장이 레슨생 화면에서 **새 초대코드 만들기** → 다시 가입 (기록 유지)

## 4. 데이터 구조 (Firestore `apps/attend/...`)
| 컬렉션 | 내용 | 누가 쓰나 |
|---|---|---|
| `students/{sid}` | name, memo, active, loginKey, code, email, uid | 원장 (레슨생은 가입 때 uid만) |
| `records/{id}` | sid, date, status, note, by, createdAt | 원장 전부 / 레슨생은 '출석' 새로 만들기만 |
| `passes/{id}` | sid, paidAt, total, amount, memo | 원장 |
| `meta/settings` | passSize, price, deductAbsent, deductLate | 원장 |
| `invites/{code}` | sid, name, email | 원장 (코드 아는 사람만 1건 조회) |
| `logins/{이름키}` | email, sid | 이름 → 로그인 이메일 연결표 |
| `users/{uid}` | sid, name | 계정 ↔ 레슨생 연결 |

- 남은 횟수는 저장하지 않고 매번 계산: 결제 순서대로 수강권을 놓고, 차감되는 기록을 날짜 순서대로 채움
- 상태: 출석·보강(항상 차감) / 당일취소·결석(설정에서 차감 여부) / 미리취소(차감 안 함)
- 레슨생 계정 이메일은 `a-초대코드@lesson-attend.app` 같은 가상 주소 (메일은 안 보냄)

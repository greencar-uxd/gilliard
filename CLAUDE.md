# 작업 지침

[출력 원칙]
- 결론부터. 서론·요약 반복·맺음말 인사 금지.
- 묻지 않은 내용 추가 설명 금지. 확장은 요청받을 때만.
- 만들기 전에 판단: 꼭 필요한가 → 이미 있는 걸 재사용 가능한가
  → 표준 기능으로 되는가 → 그래도 필요하면 최소한만.
- 예시는 1개면 충분. 같은 말 다르게 반복 금지.
- 검증·에러 처리·보안·접근성은 축약 대상에서 제외.

[검증 원칙]
- 팩트 / 추론 / 확인 필요를 구분해 표기.
- 외부 사실은 출처 명시. 근거 없으면 없다고 말할 것.
- 불확실하면 단정 대신 가능 시나리오로.

[진행 원칙]
- 매 단계 확인받지 말고 자율 진행. 막힐 때만 질문.

---

## 이 레포 고유 규칙 (gilliard)

**구성**
- 당구 운영관리 앱 하나. 회원 명단·3쿠션 경기 기록·순위·수지·회비를 다룬다.
- 순수 정적 파일 4개: `index.html`(앱 전체 — 마크업·스타일·로직이 한 파일), `gds.css`, `gds-theme.css`, `.nojekyll`. 빌드 단계 없음(바닐라 JS, Firebase compat SDK).
- Firebase 설정·회원 명단·상수는 전부 `index.html` 안에 있다. 외부 `config.js` 없음.

**데이터**
- Firebase Realtime Database(프로젝트 `srk-mt`, asia-southeast1). 쓰는 경로는 `members`, `clubmatches` 두 개.
- **이 DB는 다른 앱과 공유될 수 있다.** 한쪽만 보고 스키마를 바꾸지 말 것.
- `database.rules.json`이 이 DB 권한 규칙의 원본이다(`.read`/`.write` 전역 규칙). `firebase.json`·`.firebaserc`는 규칙 배포용.

**스타일**
- 3층 구조: `gds.css`(GDS 원시 토큰 `--gds-*`, 레포 안 복사본) → `gds-theme.css`(시맨틱 층, `--surface-*`/`--text-*`/`--space-*` 등을 `--gds-*`에 매핑) → dsds 컴포넌트 CSS(jsDelivr). `<head>`의 로드 순서를 지킬 것.
- `gds.css`는 `https://gds-e3y.pages.dev/tokens/gds.css`의 복사본. GDS 토큰이 바뀌면 손으로 다시 받아야 한다(자동 동기화 없음).

**배포**
- **GitHub Pages.** `main` 브랜치 루트 정적 파일이 그대로 `https://greencar-uxd.github.io/gilliard/`로 서빙. 커스텀 도메인·빌드·CI 배포 없음.
- **`main` 커밋/머지 = 곧 배포**(1~2분 뒤 반영). 커밋 전 배포돼도 되는 상태인지 확인할 것.
- 반영은 작업 브랜치 → PR → `main` 스쿼시 머지로. `main` 직접 push는 막혀 있음.

**절대 건드리지 말 것**
- `index.html`의 Firebase 설정(`FB` 상수 — projectId `srk-mt`, databaseURL 등). 바꾸면 데이터 연결이 끊긴다.
- RTDB 경로 구조(`members`, `clubmatches`).
- 회원 PIN(`members/<id>/pin`). 초기화·변경 금지 — 개개인이 설정한 값이다.
- 명단·설정 상수(`ROSTER_ID`/`ACTIVE`/`GHOST`/`DUES_EXTRA`/`GHOST_FROM`)는 실제 운영 데이터. 사용자 지시 없이 바꾸지 말 것.

**원칙**
- 이 레포가 source of truth. Firebase 콘솔에서 데이터 직접 편집은 지양(운영진이 앱에서 조작하는 게 기준).
- 라이브 Firebase DB는 에이전트가 코드로 직접 쓸 수 없다(권한 밖). 리셋·초기화 등은 인앱 버튼이나 코드 필터로 처리하고, 운영진이 앱에서 눌러야 반영된다.

**반복해서 틀렸던 것**
- **스타일시트를 외부 GitHub Pages에서 링크하다 세 번 깨졌다**(`design-system` → jsDelivr → `dsds` Pages). 그 사이트가 안 뜨면 페이지가 통째로 맨 HTML이 된다. 그래서 `gds.css`는 레포 안에 복사해 뒀다. 새 CSS를 외부 링크로 걸지 말 것.

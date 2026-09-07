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
- 동호회 운영 관리 앱 하나. 회장·총무가 쓴다. 탭 6개 - 대시보드 · 회원 체크 · 지출 · 월별 결산 · 설정·연동 · 운영 가이드.
- 순수 정적 파일 4개: `index.html`(앱 전체 — 마크업·스타일·로직이 한 파일), `gds.css`, `gds-theme.css`, `.nojekyll`. 빌드 단계 없음(바닐라 JS, Firebase compat SDK).
- Firebase 설정·회원 명단·상수는 전부 `index.html` 안에 있다. 외부 `config.js` 없음.

**데이터**
- Firebase Realtime Database(프로젝트 `srk-mt`, asia-southeast1). 쓰는 경로는 `gilead` 하나(`FB_PATH` 상수). 레포 이름과 다르지만 바꾸면 기존 데이터와 끊긴다.
- **이 DB는 다른 앱과 공유될 수 있다.** 한쪽만 보고 스키마를 바꾸지 말 것.
- `database.rules.json`이 이 DB 권한 규칙의 원본이다(`.read`/`.write` 전역 규칙). `firebase.json`·`.firebaserc`는 규칙 배포용.

**스타일**
- 3층 구조: `gds.css`(GDS 원시 토큰 `--gds-*`, 레포 안 복사본) → `gds-theme.css`(시맨틱 층, `--surface-*`/`--text-*`/`--space-*` 등을 `--gds-*`에 매핑) → dsds 컴포넌트 CSS(jsDelivr). `<head>`의 로드 순서를 지킬 것.
- `gds.css`는 `https://gds-e3y.pages.dev/tokens/gds.css`의 복사본. GDS 토큰이 바뀌면 손으로 다시 받아야 한다(자동 동기화 없음).
- **라이트 전용이다. 다크 모드를 만들지 말 것** — GDS 에 다크 팔레트가 없다. 예전엔 Navy 램프로 지어 썼지만 근거 없는 색이라 걷어냈다.

**배포**
- **GitHub Pages.** `main` 브랜치 루트 정적 파일이 그대로 `https://greencar-uxd.github.io/gilliard/`로 서빙. 커스텀 도메인·빌드·CI 배포 없음.
- **`main` 커밋/머지 = 곧 배포**(1~2분 뒤 반영). 커밋 전 배포돼도 되는 상태인지 확인할 것.
- 반영은 작업 브랜치 → PR → `main` 스쿼시 머지로. `main` 직접 push는 막혀 있음.

**절대 건드리지 말 것**
- `index.html`의 Firebase 설정(`FB` 상수 — projectId `srk-mt`, databaseURL 등). 바꾸면 데이터 연결이 끊긴다.
- RTDB 경로 `gilead`(`FB_PATH`).
- 인증번호 해시(`hashPin`, salt `srk!`)와 `ADMIN_ID`. 회장·총무 로그인이 이걸로 대조된다.
- 명단·설정 상수(`ACTIVE`/`GHOST`/`DUES_EXTRA`/`GHOST_FROM`/`JOIN_FROM`/`LEFT_FROM`/`NO_DUES_FROM`)는 실제 운영 데이터. 사용자 지시 없이 바꾸지 말 것.

**운영 규칙이 바뀌면 컷오프 상수로 넣는다**
- 과거 달의 결산은 그대로 보존해야 한다. 그래서 규칙 변경은 조건문이 아니라 「이 달부터」 상수로 넣는다 - `GHOST_FROM`(고스트 전환)·`JOIN_FROM`(합류)·`LEFT_FROM`(탈퇴)·`NO_DUES_FROM`(회비 폐지)이 그 패턴이다. 명단에 사람을 더할 때도 `ACTIVE`/`GHOST` 배열을 직접 늘리지 말고 `JOIN_FROM` 에 합류 달을 적는다.
- `NO_DUES_FROM='2026-09'` — 2026년 9월부터 회비를 걷지 않는다. 회비가 활동/고스트 구분의 근거였으므로 구분도 함께 없어졌다. 9월 이후 달에는 회비 카드·회비 열·활동/고스트 배지가 안 보이고, 8월 이하 달은 예전 그대로 계산된다.
- `fee()`(월 회비 금액)는 지웠으면 안 된다. 8월 이하 달의 결산이 이 값으로 계산된다. 설정 탭의 월 회비 입력칸은 고칠 일이 없어 뺐고, 값은 RTDB `settings/fee` 에 그대로 남아 있다.

**원칙**
- 이 레포가 source of truth. Firebase 콘솔에서 데이터 직접 편집은 지양(운영진이 앱에서 조작하는 게 기준).
- 라이브 Firebase DB는 에이전트가 코드로 직접 쓸 수 없다(권한 밖). 리셋·초기화 등은 인앱 버튼이나 코드 필터로 처리하고, 운영진이 앱에서 눌러야 반영된다.

**반복해서 틀렸던 것**
- **스타일시트를 외부 GitHub Pages에서 링크하다 세 번 깨졌다**(`design-system` → jsDelivr → `dsds` Pages). 그 사이트가 안 뜨면 페이지가 통째로 맨 HTML이 된다. 그래서 `gds.css`는 레포 안에 복사해 뒀다. 새 CSS를 외부 링크로 걸지 말 것.
- **당구 대전 기록 연동은 걷어냈다**(수지 관리 탭). `clubmatches`·`members` 구독과 `ROSTER_ID`도 함께 지웠다. 되살리려면 커밋 `4bd34de` 이전을 볼 것. 이미 저장된 `gilead/suji` 데이터는 DB에 남아 있다.

# Pa-Dack 코드 작업 규칙 (agent.md)

이 파일은 `index.html`(단일 파일 낚시 게임, 캔버스 + Firebase)을 수정할 때 따라야 할 규칙을 정리한 것이다.
다른 에이전트도 같은 스타일로 작업해야 한다. 사용자가 지적했던 항목을 우선 반영했다.

---

## 0. 가장 중요한 원칙

1. **같은 선택자를 두 번 정의하지 않는다.** 기존 규칙이 있으면 그 규칙 안에 속성을 추가/수정한다.
   - 파일 끝, 섹션 끝에 "새 블록"을 덧붙이지 않는다.
   - 쉼표 목록(`a, b`) 안의 선택자도 중복으로 본다.
2. **새 클래스/ID는 꼭 필요할 때만 만든다.** 기존 클래스(`.pill`, `.box`, `.btn`, `.ph`, `.seg`, `.sk` 등)를 재사용하고, 위치만 ID 규칙으로 덧붙인다.
3. **색·수치는 `:root` 토큰을 쓴다.** 하드코딩하지 않는다. 토큰이 없으면 `:root`에 먼저 추가한 뒤 쓴다.
4. **수정 후 검증한다.** 아래 6번 절차를 빠짐없이 수행한다.
5. **사용자가 확인하지 않은 동작을 임의로 바꾸지 않는다.** 애매하면 해석을 밝히고 확인을 요청한다.

---

## 1. CSS 규칙

### 1-1. 토큰 사용 (`:root`)
| 종류 | 토큰 | 용도 |
|---|---|---|
| 색 (RGB) | `rgb(var(--white-rgb) / a)`, `rgb(var(--black-rgb) / a)`, `rgb(var(--blue-950-rgb) / a)` | 투명도가 필요한 색 |
| 색 (HEX) | `var(--tx)`, `var(--tx2)`, `var(--ac)`, `var(--ac2)`, `var(--ln)`, `var(--cd)`, `var(--cd2)`, `var(--bg0)`, `var(--bg1)`, `var(--blue-900)` 등 | 불투명 색 |
| 블러 | `var(--blur-xs/sm/md/lg)` | `backdrop-filter`. 저사양 모드에서 0으로 바뀐다 |
| 시간 | `var(--dur-fast/base/slow)` | `transition` 길이 |
| 이징 | `var(--ease-std)`(좌우 이동), `var(--ease-out)`(등장/퇴장) | `transition` 곡선 |
| 간격 | `var(--gap-panel)`(패널 자식 간격), `var(--gap-tight)`(패널 내부 촘촘한 간격), `var(--card-min)`, `var(--slot-min)`, `var(--stat-min)` | `gap`, `minmax()` |
| 반경 | `var(--radius-panel)` | 다이얼로그 |
| 그림자 | `var(--shadow-float)` | 떠 있는 요소 |
| 투명도 | `var(--opacity-faint/dim/muted/soft/strong)`, `var(--opacity-scrim)` | `opacity` |
| 상단 pill | `var(--pill-top)`, `var(--pill-h)` | 상단 HUD 위치/높이 |
| z-index | `var(--z-scrim)`(딤·pill 컨테이너), `var(--z-dialog)`(패널), `var(--z-float)`(내비·HUD·토스트) | 계층 |

### 1-2. 레이아웃
- **간격은 부모의 `gap`으로 준다.** 자식의 `margin`으로 간격을 만들지 않는다.
- 정렬은 **flex 또는 grid**를 쓴다. 열 수가 화면에 따라 변해야 하면 `repeat(auto-fit, minmax(var(--토큰), 1fr))`.
- 패널 자식 중앙 정렬: `.panel > *`에서 `width: calc(100% - 32px); max-width: 560px; margin-inline: auto`.
  - `margin: auto`만 쓰면 flex 자식의 폭이 내용만큼 줄어든다. 항상 `width`를 함께 지정한다.
- 높이 고정 요소는 `min-height`/`height`를 토큰 또는 명확한 값으로 맞추고, 스켈레톤도 같은 값을 쓴다.

### 1-3. 상태 표현
- 상태는 클래스 토글로 표현한다: `.on`(선택), `.off`(숨김 상태의 pill), `.out`(페이드 아웃), `.open`(입력창 펼침).
- HTML 속성 상태: `hidden`(표시 여부), `data-lowspec`(저사양 모드, `<html>`에 붙음).
- `[hidden]`은 전역 `display: none !important`를 따른다. 같은 요소에 `display`를 지정해도 숨김은 유지된다.

### 1-4. 자주 쓰는 패턴
- **pill**: `.pill` 클래스를 재사용한다. 위치(`top/left/right`)만 ID로 지정한다.
  - 여러 pill은 컨테이너(`#hud`, `#qPills`)로 묶고, **z-index는 컨테이너에 준다.**
  - pill은 `pointer-events: auto`(클릭 인식), 컨테이너는 `none`.
- **패널/다이얼로그**: `.panel`(`role="dialog" aria-modal="true"`), 열림은 `.panel.on`.
  - 닫힌 상태: `visibility: hidden; opacity: 0; transform: translate(-50%, 5%) scale(0.9)`
  - 열린 상태: `visibility: visible; opacity: 1; transform: translate(-50%, 0) scale(1)`
  - 전환: `transition`에 `opacity`, `transform`, `visibility`를 `var(--dur-base) var(--ease-out)`로.
- **디밍**: `#dim`(`z-index: var(--z-scrim)`), `.on`으로 표시. 내비는 디밍 밖에 둔다.
- **헤더**: `.ph`(3열 grid: 빈칸 / 제목 가운데 / 닫기 버튼 오른쪽). 배경은 불투명 `rgb(var(--blue-950-rgb))`, 블러 없음.
  - 닫기 버튼은 `<button data-close>`. 클릭 시 `setTab(null)`.
- **세그먼트(탭 전환)**: `.seg` 안에 `<button data-v="…">`, 선택은 `.on`.
- **스켈레톤**: `.sk`(`skPulse` 키프레임, `--cd` ↔ `--cd2` 밝기 깜빡임). 크기 클래스 `.sk-row`, `.sk-card`(110px), `.sk-slot`.
- **반응형**: 고정 열 수 대신 `auto-fit + minmax`.

### 1-5. 금지
- 같은 선택자를 뒤쪽에 다시 쓰기
- 파일 끝에 스타일 블록 덧붙이기
- 하드코딩 색(`#fff`, `rgba(8,21,31,…)` 등), 하드코딩 블러, 하드코딩 z-index
- 자식 `margin`으로 간격 주기
- `transform`을 쓰는 요소에 `rotate`/`spin` 애니메이션 겹치기 (`.spin`은 `transform`을 덮어쓴다)

---

## 2. JavaScript 규칙

### 2-1. 형식
- 세미콜론 없음, 쌍따옴표, 들여쓰기 2칸, 한 줄 길이는 80자 안팎(prettier 스타일).
- 주석은 한국어. 이유가 드러나지 않는 코드에만 짧게 단다.
- DOM 선택은 `$("#id")` 헬퍼(또는 `document.getElementById`). 반복되는 요소는 상수로 한 번만 잡는다.

### 2-2. 상태와 저장
- 클라우드 저장 데이터는 `save` 객체. 바꾼 뒤 `persist()`를 호출한다.
- **기기별 설정은 클라우드에 올리지 않는다.** 예: 저사양 모드는 `localStorage`의 `pd-lowspec`. `save.settings`에 넣지 않는다.
- 로그인 후 `save`가 교체되므로, 설정 UI는 `applySaveUI()`에서 다시 맞춘다.

### 2-3. 탭과 패널
- 탭 전환은 `setTab(t)` 하나로 한다. `t`가 `null`이면 게임 화면만 보인다.
- 같은 탭을 다시 누르면 닫는다. 팝업이 열린 상태에서 다른 팝업으로 바꿀 때는 트랜지션을 끈다(`switching`).
- 열린 팝업이 있을 때는 키보드 게임 입력을 막는다(`if (tab) return`).

### 2-4. 무거운 렌더 (탭/도감/장비)
- `renderTab(taskFn, render, skel)` 패턴을 쓴다.
  1. `taskFn()`으로 작업 목록을 만든다. 각 작업은 `{ has, make }`.
  2. `has()`가 전부 true면 바로 `render()`. 스켈레톤도 띄우지 않는다.
  3. 아니면 `skel()`로 자리 표시자를 띄우고, 3개씩 `make()` 한 뒤 `yieldFrame()`으로 프레임을 넘긴다. 마지막에 `render()`.
- 스켈레톤은 탭별로 **최초 1회만** 쓰는 것이 기본이다(`skelOnce`). 카드 스켈레톤은 매번 쓸 수 있다.
- 슬롯 클릭처럼 일부만 바뀌는 경우에는 전체를 다시 그리지 말고 **그리드만** 다시 그린다.

### 2-5. 아이콘
- `iconURL(f, px, sil)`, `gearURL(id, px, sil)`는 `fitIcon(px, sil, draw, key)`를 쓴다.
- 완성된 dataURL은 `iconStore`(localStorage `ICON_STORE` 키)에 저장한다. 아이콘 그림을 바꾸면 **버전 문자열을 올린다.**
- 픽셀 읽기 캔버스는 `getContext("2d", { willReadFrequently: true })`.

### 2-6. 로딩
- 로딩 화면은 `#loader`, 메시지는 `setLoadMsg(t)`로 바꾼다.
- 로딩 종료는 `loaderHide()`로만 한다. `mpUI()` 시작에서 호출하고, 연결이 안 되면 15초 뒤에도 닫는다.
- 단계 메시지: "연결 중..." → "로그인 상태 확인 중..." → "데이터를 불러오는 중..." → (새 계정) "새 계정 준비 중...".

### 2-7. 난수와 맵 생성 (결정론)
- 맵 생성은 **고정 시드**(`mulberry(시드)`, `hash()`)만 쓴다. `Math.random()`을 맵 생성 경로에 쓰지 않는다.
- **정렬 비교 함수 안에서 난수를 호출하지 않는다.** 엔진(V8/JSC)마다 호출 순서가 달라져 기기별 맵이 갈라진다.
  - 올바른 방법: 정렬 전에 점수를 한 번씩 정한 뒤(`c.sc = …`), 비교는 그 값만 쓴다.
- 난수 호출 순서는 기기 상태(화면 크기, 저장 데이터)와 무관해야 한다.

### 2-8. 메시지
- 토스트 하나에 상태를 **한 줄로** 담는다. 예: `"꽝이에요 (남은 상자 3개)"`.
- `toast(msg, ms)`는 토스트와 시스템 로그를 함께 쓴다. 로그에만 남기려면 `addSys`/`addLine`을 쓴다.
- 공지 `[공지] …`는 보라색 로그(`.cl.notice`), 관리자 메시지는 빨간 이름(`.cl.adm`).

### 2-9. 입력과 포인터
- 캔버스 포인터는 UI 요소를 무시해야 한다: `closest("#act,#card,#mm,#qPills,#hud")`처럼 목록에 추가.
- UI 위에서 시작한 포인터는 `stopPropagation()`으로 캔버스 조작과 분리한다.
- 버튼의 포커스 탈취를 막으려면 `mousedown`에서 `preventDefault()`.

---

## 3. 화면 계층 (z-index 토큰 요약)
1. 게임 캔버스, 낚시 HUD 요소 (기본)
2. `#qPills` 컨테이너, `#dim` (`--z-scrim`)
3. 패널/다이얼로그 `.panel` (`--z-dialog`)
4. 내비, 상단 HUD `#hud`, 토스트 (`--z-float`)
5. 로딩 화면 (가장 위)

---

## 4. 저사양 모드
- `html[data-lowspec]`이 붙으면 블러 토큰을 0으로, `backdrop-filter`를 `none`으로 덮어쓴다.
- 캔버스 배율: 기본 `min(devicePixelRatio, 2.5)`, 저사양 `1`. 미니맵은 기본 `min(dpr, 3)`, 저사양 `1`.
- 저사양 값을 바꿀 때는 `resize()`를 다시 호출한다.

---

## 5. 맵/기기 일관성
- iOS와 안드로이드에서 맵이 같아야 한다. 같은 시드와 결정론적 계산만 쓴다.
- 화면 비율에 따라 맵 구조가 바뀌면 안 된다. 타일 크기는 화면 비율로 정하되, 지형 데이터에는 화면 값을 넣지 않는다.

---

## 6. 수정 후 검증 절차 (반드시 수행)
1. **문법 검사**: `<script>` 블록을 추출해 `node --check`로 확인.
2. **중복 선택자 검사**: 같은 미디어 조건에서 같은 선택자(쉼표 목록 포함)가 두 번 나오면 병합한다.
3. **병합 동등성 검사**: 병합 전후로 선택자별 최종 속성 값이 같은지 비교한다. 차이가 있으면 병합을 취소한다.
4. **잔여 참조 검사**: 제거한 ID/함수/변수(예: 옛 `#qSheet`, `#qClose`, `gearCnt`)가 JS/CSS에 남아 있지 않은지 `grep`.
5. **결과물 저장**: 수정 결과를 `/mnt/user-data/outputs/index.html`에 저장하고, 사용자에게 첨부.

---

## 7. 사용자 피드백에서 확정된 선호
- 정사각형 아이콘, 투명 배경, 캐릭터: 흰 외곽 원, 분홍 얼굴, 검은 눈, 웃는 입, 발 두 개(주황). 윙크 눈 포함.
- 로딩 메시지는 구체적으로(연결 중 / 데이터를 불러오는 중 / 새 계정 준비 중).
- 스피너보다 스켈레톤을 선호. 스켈레톤은 깜빡임(제자리 밝기 변화)으로.
- 팝업은 "창" 느낌: 타이틀바 가운데 제목, 오른쪽 X, 등장은 살짝 아래에서 올라오며 scale 0.9 → 1.
- 내비 선택 배경: 최초 등장은 scale 0.9 → 1 + 페이드, 탭 이동은 좌우 미끄러짐.
- 퀘스트 탭: 내부 탭은 "일일 퀘스트 / 보물상자" 두 개. 출석 보상은 일일 퀘스트 맨 위.
- 보물상자 기록은 최근 20개까지.

---

## 8. 작업 체크리스트 (요약)
- [ ] 기존 규칙에 추가했는가 (새 블록을 뒤에 붙이지 않았는가)
- [ ] 색/수치/간격/z-index를 토큰으로 썼는가
- [ ] 간격을 `gap`으로 줬는가 (자식 `margin` 없음)
- [ ] 기기별 설정은 `localStorage`에 있는가 (클라우드 X)
- [ ] 무거운 렌더는 `renderTab`을 거쳤는가
- [ ] 정렬 비교 함수 안에 난수가 없는가
- [ ] 검증 1~5를 수행했는가

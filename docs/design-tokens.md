# 디자인 토큰 & 제작 컨벤션

각 시각화는 **자기완결 단일 HTML**이므로 CSS를 런타임으로 공유하지 않습니다.
대신 아래 규칙을 모든 개념 파일이 동일하게 따라 사이트 전체의 일관된 룩을 유지합니다.

## 색 (테마 대응)

- 모든 색은 CSS 변수로 사용: `--background`, `--foreground`, `--card`, `--muted`, `--muted-foreground`, `--border`, `--primary`, `--viz-series-1` ~ `--viz-series-6`
- 리터럴 색상 금지 (`#fff`, `black`, Tailwind 팔레트 등) — 라이트/다크 테마 자동 대응을 위해
- 시리즈 색 규칙:
  - 첫 번째 계열/강조/활성 상태 → `--viz-series-1`
  - `--viz-series-2`~`6`은 카테고리 구분이 정말 필요할 때만
- 큰 면적 채움은 낮은 투명도(0.1~0.5), 테두리·구조선은 `--border` 1~2px
- 수치 막대/트랙에는 장식 테두리 금지 — 채움·투명도·간격으로 구분

## 타이포

- 본문·컨트롤: 호스트 기본 크기, 굵기는 400/500만 사용
- SVG 텍스트: 제목·값 12px(`--foreground`), 보조 11px(`--muted-foreground`) — **11px 미만 금지**
- 변하는 숫자·정렬 숫자는 `tabular-nums`

## 구성

- 개념당 `h1` 하나 + 필요시 정의용 캡션 한 줄
- 컨트롤은 `.viz-controls` 한 행에 모으기 (재생/정지, 단계 이동 등)
- 요약 수치는 `.viz-grid` + `.viz-stat` 카드, 최대 3개
- 카드 중첩 금지, 차트·도해는 프레임 없이 투명 배경에 직접 렌더

## 애니메이션

- `prefers-reduced-motion` 존중: 자동 재생 금지, 수동 재생 버튼 제공
- 상태 전환은 transition으로 처리, 등장 페이드 금지
- 시뮬레이션 흐름(물류 흐름 애니메이션)은 재생/정지 컨트롤 제공을 전제로 허용

## SVG / 반응형

- viewBox는 **컨테이너 실측 폭**으로 계산하고 `ResizeObserver`로 다시 그림 (고정폭 축소 금지)
- 좁은 폭(<620px)에서는 세로 배치로 전환, 텍스트는 어느 폭에서든 11px 이상 유지
- 320px에서도 내용이 겹치거나 잘리지 않아야 함

## 접근성

- 시뮬레이션 SVG에 `role="img"` + `aria-label`(한글 요약) 제공
- 동적 상태 안내는 `aria-live="polite"` 한 곳에서만
- 네이티브 `button`/`input` 사용, `tabindex` 추가 금지

## 상태 저장 (인라인 위젯)

- 사용자가 고른 모드 등 의미 있는 선택은 `window.zcode.setWidgetState({ modelContent })`에 저장
- 로드 시 `window.zcode.widgetState`에서 복원, **로드 시 저장 금지**
- 저장 용량 16KiB 이하

# 물류 개념 시각화 (Logistics Concepts, Visualized)

크로스독, LPN, SKU, 피킹 방식 같은 물류 핵심 개념을 **인터랙티브 애니메이션**으로 설명하는 정적 사이트입니다.
목표는 **물류를 처음 접하는 사람이 4PL 시점까지 이해하는 것**입니다 — 기본 개념(첫걸음 트랙)에서 시작해, WMS가 맡는 실행(창고 안의 일)과 OMS가 맡는 조율(화주·운영사·창고·운송사 사이의 일)로 이어집니다.

**현재 진행**: 첫걸음 트랙 6편 + 실행 층 16편 완료 — 총 22편 · 다음은 4PL 트랙 20편 — 전체 목록과 제작 순서는 `docs/roadmap.md`

## 핵심 설계 원칙

**개념 1개 = 자기완결 HTML 1개**

- 각 시각화는 외부 CSS/JS 없이 단일 파일로 완결됩니다 (빌드 도구 불필요).
- 라이브러리(D3 등)는 페이지에 임베드되므로 배포 후에도 외부 CDN 의존이 없습니다.
- 색·타이포 통일은 코드 공유가 아닌 컨벤션 공유로 해결 → `docs/design-tokens.md`
- 모든 페이지는 "질문 → 체험 → 손잡이 → 숫자 → 결론" 학습 패턴을 따름 (신규 페이지는 "OMS에서는 / WMS에서는" 한 줄 포함) → `docs/learning-pattern.md`
- 물류를 모르는 사람도 읽을 수 있게 씀 — 새 용어는 처음 나오는 자리에서 풀이하고, 풀이 문장은 용어집(`docs/glossary.md`)을 따름

## 학습 기능

- **학습 트랙**: 첫걸음 / 핵심 5개념 / 입고(Inbound) / 실전 챌린지 — `data/catalog.json`의 `tracks`에서 관리. 4PL 트랙은 묶음이 완료되면 추가
- **진도 저장**: 각 페이지 하단 `✓ 학습 완료` 버튼 → `localStorage['mv-done:<slug>']`, 홈에서 진도 바·완료 배지로 표시

## 다루는 영역

완료된 개념과 제작 예정 개념의 예시입니다. 보류한 개념(자동화 등)은 `docs/roadmap.md`의 "보류 · 합침"에 있습니다.

| 카테고리 | 예시 개념 |
|----------|----------|
| 창고 운영 | 창고의 하루, 크로스독, 피킹 방식 4종, 입고 생애주기, 적치 전략, 출고 플로우 |
| 식별·단위 | SKU vs LPN, 포장 단위와 UoM 변환 |
| 재고 | 재고의 기본, FIFO·LIFO·FEFO, 안전재고·재주문점·EOQ, 다중 창고 재고 배치, 재고 대사 |
| 수송·네트워크 | 허브 앤 스포크, TL vs LTL, 라스트마일, 운송사 선정, 네트워크 설계 |
| 주문·이행 | 주문 한 건의 여정, OMS→WMS→TMS 연계, 계약 창고 물량 배정, 주문 라우팅, 할당과 ATP |
| 시스템 | 물류 시스템 지도, 운영 구조(자사 운영·자체 운영), 상태 매핑, 마스터 데이터 표준화, 외부 시스템 연동, 컨트롤 타워 |
| 전략 | 물류의 큰 흐름, 물류의 등장인물, 3PL·4PL·포워더의 역할, 물류 KPI, SLA·스코어카드, 계약과 요율, 물류비 정산 |

## 프로젝트 구조

```
물류/
├── index.html            # 카탈로그 홈 (트랙 진도 + 개념 카드 + 카테고리 필터)
├── concepts/             # 배포되는 개별 시각화 페이지 (자기완결 HTML)
├── data/catalog.json     # 개념 메타데이터 + 학습 트랙 (index.html이 읽어 렌더링)
├── docs/roadmap.md       # 개념 로드맵 (첫걸음 트랙 · 실행 층 · 4PL 트랙) · 제작 순서 · 진행 상태
├── docs/glossary.md      # 용어집 — 모든 페이지의 풀이 문장의 기준
├── docs/design-tokens.md # 색·타이포·애니메이션 제작 컨벤션
├── docs/learning-pattern.md # 페이지 규격서 (구성 · 체험 형식 5종 · 쉽게 쓰기 · 기획서 템플릿)
└── docs/inbound-concepts.md # 입고 모듈 개념 지도
```

## 워크플로 (새 개념 추가하기)

0. **기획** — `docs/roadmap.md`에서 다음 순서의 개념을 고르고, `docs/learning-pattern.md`의 "페이지 기획서"를 채움 (slug · 질문 · 형식은 로드맵 그대로)
1. **인라인 제작** — ZCode 대화에서 visualize 스킬로 시각화를 만들고 검토
2. **Export** — 승인되면 standalone HTML로 내보내기:
   ```bash
   python3 "$HOME/.zcode/cli/plugins/cache/zcode-plugins-official/visualize/0.1.1/skills/visualize/scripts/render.py" \
     <fragment.html> concepts/<slug>.html
   ```
   - slug는 ASCII 소문자-하이픈 (예: `crossdock`, `fifo-fefo`, `hub-and-spoke`)
3. **등록** — `data/catalog.json`의 `concepts` 배열에 메타데이터 추가 (제목/카테고리/요약/파일 경로)하고 `planned`에서 같은 `id`를 제거, 필요시 `tracks`에 포함
4. **완료 버튼** — 페이지 하단에 `✓ 학습 완료` 버튼 추가 (`localStorage['mv-done:<slug>']` 저장, 골격은 기존 페이지 참고)
5. **로드맵 갱신** — `docs/roadmap.md`에서 상태를 ✅ 로 변경
6. **배포** — 커밋 & 푸시하면 끝

## 로컬 미리보기

`fetch()`로 카탈로그를 읽으므로 파일을 직접 열지 말고 로컬 서버로 확인하세요:

```bash
python3 -m http.server 8080
# → http://localhost:8080
```

## 배포

정적 사이트라 별도 빌드 설정이 필요 없습니다.

### GitHub Pages (추천)

```bash
git init
git add -A
git commit -m "init: logistics visualization catalog"
# GitHub에서 리포지토리 생성 후
git remote add origin git@github.com:<user>/<repo>.git
git push -u origin main
# → Settings → Pages → Source: main branch / root
```

### Vercel / Netlify

리포지토리 연결만 하면 됩니다. 빌드 명령 없음, 출력 디렉터리 = 루트.

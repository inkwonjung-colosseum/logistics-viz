# 물류 개념 시각화 (Logistics Concepts, Visualized)

크로스독, LPN, SKU, 피킹 방식 같은 물류 핵심 개념을 **인터랙티브 애니메이션**으로 설명하는 정적 사이트입니다.

**현재 진행**: 1 / 49편 공개 (크로스독 ✅) — 전체 목록은 `docs/roadmap.md`

## 핵심 설계 원칙

**개념 1개 = 자기완결 HTML 1개**

- 각 시각화는 외부 CSS/JS 없이 단일 파일로 완결됩니다 (빌드 도구 불필요).
- 라이브러리(D3 등)는 페이지에 임베드되므로 배포 후에도 외부 CDN 의존이 없습니다.
- 색·타이포 통일은 코드 공유가 아닌 컨벤션 공유로 해결 → `docs/design-tokens.md`

## 다루는 영역

| 카테고리 | 예시 개념 |
|----------|----------|
| 창고 운영 | 크로스독, 피킹 방식 4종, Wave Picking, 슬로팅, 인바운드/아웃바운드 플로우 |
| 식별·단위 | SKU vs LPN, Pack Hierarchy, UoM 변환, GTIN·SSCC |
| 재고 | FIFO·LIFO·FEFO, 안전재고·ROP·EOQ, ABC·XYZ 분석 |
| 수송·네트워크 | 허브 앤 스포크, TL vs LTL, 밀크런, 라스트마일, VRP |
| 주문·이행 | 주문 라이프사이클, 할당과 ATP, OMS→WMS→TMS 연계 |
| 자동화 | AS/RS, AGV vs AMR, Pick-to-Light, 컨베이어·소터 |
| 시스템 | WMS·TMS·OMS·ERP·YMS, EDI 흐름, 디지털 트윈 |
| 전략 | 채찍효과, Push vs Pull, 역물류, 콜드체인, 물류 KPI |

## 프로젝트 구조

```
물류/
├── index.html            # 카탈로그 홈 (개념 카드 + 카테고리 필터)
├── concepts/             # 배포되는 개별 시각화 페이지 (자기완결 HTML)
│   └── crossdock.html    #   예: 크로스독
├── data/catalog.json     # 개념 메타데이터 (index.html이 읽어 렌더링)
├── docs/roadmap.md       # 전체 개념 로드맵 + 진행 상태
└── docs/design-tokens.md # 색·타이포·애니메이션 제작 컨벤션
```

## 워크플로 (새 개념 추가하기)

1. **인라인 제작** — ZCode 대화에서 visualize 스킬로 시각화를 만들고 검토
2. **Export** — 승인되면 standalone HTML로 내보내기:
   ```bash
   python3 "$HOME/.zcode/cli/plugins/cache/zcode-plugins-official/visualize/0.1.1/skills/visualize/scripts/render.py" \
     <fragment.html> concepts/<slug>.html
   ```
   - slug는 ASCII 소문자-하이픈 (예: `crossdock`, `fifo-fefo`, `hub-and-spoke`)
3. **등록** — `data/catalog.json`의 `concepts` 배열에 메타데이터 추가 (제목/카테고리/요약/파일 경로)
4. **로드맵 갱신** — `docs/roadmap.md`에서 상태를 ✅ 로 변경
5. **배포** — 커밋 & 푸시하면 끝

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

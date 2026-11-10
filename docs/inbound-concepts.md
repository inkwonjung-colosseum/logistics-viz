# 입고(Inbound) 개념 전체 지도

입고 모듈을 개발/검증할 때 다뤄야 하는 개념을 생애주기 순서로 정리한 문서.
✅ = 이 프로젝트에 시각화가 이미 있음, ⬜ = 시각화 미제작(후보).

```
기준정보 → 발주(PO) → ASN → 예약/야적 → 도착·하역 → 접수·검수 → 검품(QC)
   → 가용 전환 → 적치(Put-away) → ERP 전기(GR) → 송장 · 3-way match
              ↑ 예외: 과다·부족·파손·지연 ↑        ↑ 수입입고: B/L·통관 ↑
```

## 1. 기준정보 — 입고가 성립하려면 먼저 정의되어야 하는 것

| 개념 | 설명 | 개발 포인트 |
|---|---|---|
| 품목 마스터 (SKU) | 입고 단위, 검사 대상 여부, 유효기간/lot 추적 여부, 보관 조건 | 품목 속성이 검수·QC·적치 로직을 결정 |
| UoM · Pack Hierarchy | EA/박스/카톤/팔레트 단위 변환 | 입고 단위 ≠ 보관 단위 변환 규칙 |
| 공급사 마스터 (Vendor) | 리드타임, ASN 준수율, 허용 과다입고 % | 공급사별 검수 정책 차이 |
| Routing Guide | 공급사 배송 규칙 (라벨, 팔레트 표준, ASN 필수) | 컴플라이언스 위반 페널티 |
| 검사 기준 (AQL) | 품목별 전수/샘플링 등급 | QC 필요 품목 판별 규칙 |
| 위치 마스터 | 도크·랙·스테이징, 도크별 하역 조건 | Put-away 제안의 기반 |
| 캘린더 · Capacity | 도크 예약 가능 시간, 일일 수용 한계 | 어포인트먼트 슬롯 관리 |

## 2. 발주 · 사전 단계 (Pre-Arrival)

| 개념 | 설명 | 시각화 |
|---|---|---|
| PO (발주서, EDI 850) | 입고의 근거 문서 — 검수 시 대조 기준 | ⬜ (생애주기에서 다룸 ✅) |
| ASN (EDI 856) | 사전 출하 통지 | ✅ asn |
| Shipment | 운송 단위 — 복수 PO 묶음, 분할 배송 | ⬜ |
| 도크 어포인트먼트 | 도착 시간 예약 | ✅ yms-dock-scheduling |
| VMI · Consignment | 위탁 보관 — 입고됐지만 소유권은 공급사 | ⬜ |
| 수입 준비 | B/L, Invoice, Packing List, Incoterms | ✅ import-inbound |

## 3. 도착 · 하역 (Arrival & Unloading)

| 개념 | 설명 | 시각화 |
|---|---|---|
| 게이트 체크인 · 봉인 확인 | 컨테이너 seal number 대조 | ⬜ |
| YMS (야적장 관리) | 게이트→야적→도크 트럭 흐름 | ✅ yms-dock-scheduling |
| Live vs Drop unload | 기다려 하역 vs 트레일러 통째 반입 | ⬜ |
| 데버닝 (Devanning) | 컨테이너 내부에서의 하역( unstuffing ) | ⬜ |
| 하역 장비 | 도크 레벨러, 지게차, 팔레트잭, 컨베이어 | ⬜ |
| 인력 계획 | 도착 예측 기반 레이버 스케줄링 | ⬜ |

## 4. 접수 · 검수 (Receiving) — 입고 모듈의 심장

### 4-1. 입고 유형 (시스템이 지원해야 하는 처리 경로)
| 유형 | 특징 | 시각화 |
|---|---|---|
| 표준 PO 입고 | PO 라인 대조 | ⬜ (생애주기 ✅) |
| ASN 스캔 입고 | 라벨 스캔 → 자동 대조 | ✅ blind-receiving(ASN 모드) |
| 블라인드 입고 | 사전 정보 없이 전표 대조 | ✅ blind-receiving |
| 무발주 입고 (No-PO) | 소매 보충 — PO 없이 수량만 등록 | ⬜ |
| 크로스독 접수 | 접수 즉시 출고 레인 할당 | ✅ crossdock |
| 반품 입고 (RMA) | 고객 반품 — 역방향 인바운드 | ⬜ |
| 이전 입고 (Transfer In) | 타 창고/타 지점 이전 | ⬜ |
| 위탁 입고 (Consignment) | 소유권 이전 없는 재고 | ⬜ |
| 키트 부품 입고 | 부품 입고 → 키트 구성 완료 | ⬜ |
| 무상 지급 자재 | 고객 제공 자재 (FOC) | ⬜ |

### 4-2. 접수 방식 (같은 검수, 다른 단위)
| 방식 | 설명 |
|---|---|
| 팔레트(LP) 입고 | 팔레트 라벨 1회 스캔 → 내부 품목 자동 확정 |
| 박스 단위 입고 | 박스 스캔 → 수량 확정 |
| 개품(EA) 입고 | 개별 계수 — 최느림, 분쟁 대응용 |
| RFID 게이트 | 통과 즉시 전 수량 인식 |

### 4-3. 검수 시 캡처해야 하는 데이터
수량 · 품목 · lot/유효기한 · 시리얼 · 단위중량(catch weight) · 팔레트 상태(CHEP 교환) · 손상 여부 · 온도(콜드체인 데이터로거)

## 5. 수량 · 품질 예외 — 실무에서 가장 많은 코딩 포인트

| 개념 | 설명 | 시각화 |
|---|---|---|
| 과다 입고 (Over) | PO 초과 — 허용 % 내 자동, 초과 시 상신 | ✅ osd-exceptions |
| 부족 입고 (Short) | 미도착 수량 — 백오더 유지 + 클레임 | ✅ osd-exceptions |
| 파손 · 불량 (Damaged) | 보류 → 사진/기록 → 클레임 or 폐기 | ✅ osd-exceptions |
| OS&D 리포트 | Over/Short/Damage 통합 보고서 | ✅ osd-exceptions |
| 출입 거부 (Refuse) | 서류·상태 불량으로 인수 거부 | ⬜ |
| 유효기간 초과 입고 | 잔여 기한 정책 미달 거부 | ⬜ |
| 도크 병목 · 디텐션 | 컨테이너 체류비 발생 | ⬜ |

## 6. 검품(QC) · 재고 상태 전환

| 개념 | 시각화 |
|---|---|
| 전수 vs 샘플링(AQL), 합격→가용 / 불합격→보류 | ✅ qc-stock-status |
| Inspection Lot (검사 로트 단위 관리) | ⬜ |
| 특채 처리 (Use-as-is), 재작업 (Rework) | ⬜ |

## 7. 적치 (Put-away)

| 개념 | 시각화 |
|---|---|
| 랜덤 · 고정 위치 · ABC 슬로팅 | ✅ putaway-strategies |
| Direct/flow-through 적치 (즉시 출고 지역) | ✅ crossdock |
| 이중 위치 적치 (reserve → forward 보충 겸용) | ⬜ |
| Task interleaving (적치+보충 동선 묶기) | ⬜ |
| 조건부 적치 (냉장/위험물/중량 분리) | ⬜ |

## 8. 시스템 · 문서 · 연동

| 개념 | 설명 | 시각화 |
|---|---|---|
| 입고 상태 머신 | 예정→도착→검수중→검사중→가용 | ✅ inbound-lifecycle |
| 문서 흐름 | PO(850) → ASN(856) → GR(861) → 송장(810) | ✅ inbound-lifecycle |
| LPN 부여 | 검수 시점에 새 팔레트 LPN 생성·라벨 출력 | ⬜ |
| GR 전기 (ERP) | 재계정 이동 — 재무와의 접점 | ✅ inbound-lifecycle |
| 3-way match | PO · GR · 송장 대조 → 지급 | ✅ inbound-lifecycle |
| 2-way match (GR 기반) | 서비스/무발주 품목용 | ⬜ |

## 9. 수입 입고 (International Inbound)

| 개념 | 시각화 |
|---|---|
| Incoterms (FOB/CIF/DDP — 비용·위험 인계점) | ✅ import-inbound |
| B/L (선하증권) — 화물 인도 증서 | ✅ import-inbound |
| 수입 통관 (수입신고 → 관세·부가세 → 수리) | ✅ import-inbound |
| 보세구역 (집하장 반입) | ✅ import-inbound |
| 트러킹 (항만→창고 국내 운송) | ✅ import-inbound |
| FTA 원산지 증명 (관세 절감) | ⬜ |

## 10. 지표 (KPI)

| 지표 | 정의 | 시각화 |
|---|---|---|
| Dock-to-stock time | 도착→가용 전환 리드타임 | ⬜ (후보 높음) |
| Receiving accuracy | 검수 정확도 | ⬜ |
| ASN compliance | 공급사 ASN 전송율 | ⬜ |
| Supplier OTD | 약속일 준수율 | ⬜ |
| 하역 생산성 | pallets/hour per dock | ⬜ |

## 11. 자동화 · 고급

| 개념 | 설명 |
|---|---|
| 자동 하역 로봇 | 컨테이너 내 자동 언로딩 |
| RFID 게이트 리딩 | 문 통과만으로 입고 확정 |
| 비전 검수 | 박스 라벨 이미지 판독 |
| 인바운드 프리소팅 | 공급사가 출고처별로 미리 정렬해 배송 |
| 디지털 트윈 시뮬레이션 | 도착 볼륨 예측 기반 도크·인력 계획 |

## 시각화 제작 현황 (9편)

| # | 파일 | 개념 |
|---|---|---|
| 1 | crossdock.html | 크로스독 |
| 2 | asn.html | ASN |
| 3 | blind-receiving.html | 블라인드 vs ASN 스캔 |
| 4 | putaway-strategies.html | 적치 전략 3종 |
| 5 | yms-dock-scheduling.html | YMS · 도크 스케줄링 |
| 6 | qc-stock-status.html | 검품 · 상태 전환 |
| 7 | inbound-lifecycle.html | 생애주기 · 문서 흐름 · 3-way match |
| 8 | osd-exceptions.html | 과다 · 부족 · 파손 예외 |
| 9 | import-inbound.html | 수입 입고 체인 |

## 다음 제작 우선순위 제안

1. Dock-to-stock KPI 분해 (시간 구간별 측정 — WMS 리포트 설계에 직결)
2. 입고 유형 카탈로그 (무PO · RMA · 이전 · 위탁 — 유형별 플로우 차이)
3. LPN 팔레트 입고 (팔레트 스캔 vs 개품 스캔)
4. 데버닝 (컨테이너 하역)
5. RFID 게이트 리딩 vs 바코드 스캔

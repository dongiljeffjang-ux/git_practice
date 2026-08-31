# 중국 데이터 소스 검증 리포트 (2026-08-31)

> 목적: China Battery Lens의 소스 스택 확정. 판정·접근조건·제약을 검증한 결과. finlight.me / webapi.cninfo.com.cn 두 도메인은 검증 환경 프록시 차단으로 직접 fetch 불가 → 검색 기반 확인, 해당 비용/티어는 "미확인"으로 표기(계약 전 직접 견적 필요).

## 판정표

| 소스 | 판정 | 접근조건 | Python | 핵심 제약 |
|---|---|---|---|---|
| **AKShare** | 가능 | 무료·API키 불필요, MIT | 네이티브(pip, DataFrame) | 상류(东方财富·新浪) 포맷 변경 시 인터페이스 수시 파손. "학술용 한정" 명시 → 잦은 업데이트 필요 |
| East Money 东方财富(비공식) | 조건부 | 무료·비문서화 | AKShare `_em` 함수로 래핑 | SLA 없음, 예고 없이 변경 |
| **CNINFO 巨潮资讯(공개사이트)** | 가능 | 무료·로그인 불필요 | AKShare 일부 래핑 + 자체 크롤러 | 全 A주(SSE·SZSE·科创·创业·北交소) 공시·PDF. 비정형 PDF 파싱 부담, IP throttling |
| CNINFO Data Service(공식 API) | 조건부 | 계정+실명인증, 포인트제 | REST | 무료/유료 경계·비용 **미확인** |
| SSE 상하이거래소 | 조건부 | 무료 | 네이티브 REST 미확인 | 실무상 CNINFO 경유가 표준(영문 포털 별도) |
| **HKEXnews(홍콩·H주)** | 가능 | 무료·쿠키 불필요 | Title/Full-text Search가 JSON 반환 | 연차·실적발표·공시 PDF 직링크. 가장 깔끔 |
| GGII 高工锂电/高工产研 | 조건부 | 웹 무료, 심층리포트 유료 | 크롤링만(RSS/API 없음) | 핵심 출하량·점유율 원데이터는 유료 컨설팅. 公众号 "高工产研" |
| 电池中国 cbea | 미확인 | 웹 무료 추정 | 미확인 | 공식 피드/API 확인 못 함 |
| 36kr | 가능 | 무료 | 네이티브 `/feed` + RSSHub | — |
| CnEVPost(영문) | 조건부 | 무료 | 네이티브 RSS 미확인 | 뉴스레터·Telegram·Google News 채널 |
| WeChat 公众号 → WeWe RSS/RSSHub | 조건부 | 무료·셀프호스팅(微信读书 세션) | Docker 셀프호스트 | 세션 만료·반크롤링 → **안정성 낮음** |
| Finlight(중문 금융뉴스, 유료) | 가능 | 무료 5,000req/월(12h 지연)~, $29/월~ | REST+WebSocket | 财新·财联社·东方财富·第一财经. **뉴스 전용(공시 아님)** |
| FinancialFilings China(유료) | 가능 | 무료 100calls/년 테스트, 유료 | REST(JSON/MD/PDF) | 구조화 공시 clean Markdown, 지연 <1분. 상위 티어 비용 미확인 |

## 비상장·자회사 소재사 커버리지

| 회사 | 상태 | 재무·공시 포착 |
|---|---|---|
| Ronbay 容百(688005 科创) | 상장 | O (AKShare·CNINFO) |
| Hunan Yuneng 湖南裕能(301358 创业) | 상장 | O |
| Dynanonic 德方纳米(300769 创业) | 상장 | O |
| Easpring 当升科技(300073) | 상장 | O |
| Putailai 璞泰来(603659, 음극) | 상장 | O |
| BTR 贝特瑞(835185 → **920185** 北交소) | 상장 | O (北交소 포함) |
| **Brunp 邦普**(CATL 자회사) | 비상장 | X — CATL(300750) 연결재무·세그먼트 주석만 |
| **Bamo 巴莫**(Huayou 완전자회사) | 비상장 | X — Huayou(603799) 연결재무 내에서만 |

## 확정 소스 전략

- **MVP 무료 스택** (Phase 1 착수):
  - 시세·재무: **AKShare** (상장 소재사)
  - 공시: **CNINFO 공개사이트**(A주 전반) + **HKEXnews**(H주)
  - 뉴스: **36kr RSS + Google News zh-CN + CnEVPost(영문)** + GGII/公众号 헤드라인(크롤링, 불안정 감안)
- **유료 헤지** (발주자 승인 후, 안정성·ToS 리스크 대비 이중화):
  - **Finlight**(중문 금융뉴스, 무료 5,000/월 티어부터 시작 가능) — 뉴스 안정화용
  - **FinancialFilings China**(구조화 공시) / **CNINFO Data Service**(구조화) — 공시 정형화용
- **무료 스택 밖 → 수동 업로드·정성**:
  - 비상장 소재사 재무, 산업 출하량·점유율·가동률 원데이터(GGII·SNE 유료 리포트) → 관리자 파일 업로드

## robots/ToS
- CNINFO·SSE·HKEXnews·东方财富는 비문서화 크롤링이 광범위(AKShare가 이를 전제). 학술/개인은 사실상 묵인이나, **상업 서비스화 시 대량 재배포엔 법무 리뷰 권장**.
- 微信 公众号는 강한 반크롤링 → RSS 우회 근본적으로 불안정. 보조로만.

## 커버 안 되는 것 (요약)
1. **비상장 소재 자회사 재무 공백이 최대 구멍** — Brunp·Bamo 등은 독립 공시 없이 모회사 연결 주석 세그먼트 수준으로만 추정.
2. **산업 원데이터(출하·점유·가동률)는 무료 밖** — GGII 등 유료 리포트에 갇힘, API 없음. 헤드라인만 우회 취득.
3. **CNINFO 공식 API·FinancialFilings 상위 티어 비용 미확인** — 계약 전 직접 견적 필요.

*출처: AKShare(github.com/akfamily/akshare), CNINFO(cninfo.com.cn / webapi.cninfo.com.cn), HKEXnews(hkexnews.hk), SSE(english.sse.com.cn), GGII(gg-lb.com), 36kr/RSSHub, CnEVPost(cnevpost.com), WeWe RSS(github.com/cooderl/wewe-rss), Finlight(finlight.me), FinancialFilings(financialfilings.com), 기업 상장상태(marketscreener·stockanalysis·CATL/Huayou IR).*

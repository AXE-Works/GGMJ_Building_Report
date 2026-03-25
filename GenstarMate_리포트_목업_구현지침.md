# GenstarMate AI 임차인 리포트 — 목업 웹페이지 구현 지침

> **목적**: 이 문서는 Claude Code에서 프론트엔드 목업 페이지를 생성하기 위한 구현 지침입니다.
> 백엔드 없이, 하드코딩된 mock 데이터로 실제 AI 리포트처럼 보이는 인터랙티브 웹페이지를 만듭니다.

---

## 1. 프로젝트 개요

### 무엇을 만드는가
- 부동산 임차 자문 회사(GenstarMate)가 임차인에게 제공하는 **오피스 이전 후보지 비교 리포트 웹페이지**
- 기존에 PPT/Excel로 수작업하던 제안서를 웹 인터랙티브 리포트로 전환
- AI가 자동 생성한 것처럼 보이는 목업 (실제 AI 호출 없음, mock 데이터 사용)

### 핵심 요구사항
1. **지도(Map) 기반 UI** — 후보 빌딩들의 위치를 지도 위에서 탐색 (필수)
2. **빌딩 상세 카드** — 사진, 스펙, 임대조건, 장단점을 깔끔하게 표시
3. **비용 비교 차트** — ENOC(실질임대비용) 바 차트, 월비용 비교
4. **AI 테스트핏 UI** — 조건 입력 폼 + "생성 중..." 애니메이션 + 결과 이미지 표시
5. **깔끔한 디자인** — 건물/공간/도면 이미지가 잘 보이는 것이 최우선

### 기술 스택
- **React (JSX)** — 단일 파일 artifact (.jsx)
- **Tailwind CSS** — 스타일링 (core utility classes only)
- **Recharts** — 차트 (import 가능)
- **Lucide React** — 아이콘 (import 가능)
- 외부 CDN: Leaflet.js (지도, CDN으로 로드)
- 이미지: Unsplash/placeholder URL 사용

---

## 2. Mock 데이터

### 2.1 빌딩 데이터 (9개 후보지)

아래 데이터를 상수로 하드코딩합니다. 실제 첨부 파일에서 추출한 데이터입니다.

```javascript
const BUILDINGS = [
  {
    id: 1,
    name: "교원명동빌딩",
    region: "CBD",
    address: "서울시 중구 을지로 55",
    station: "2호선 을지로입구역 도보 1분",
    lat: 37.5660, lng: 126.9830,
    grossArea: 4006,        // 연면적 (평)
    netRatio: 54.24,        // 전용률 (%)
    floors: "B3/15F",
    builtYear: 1979,
    remodelYear: null,
    floorNetArea: 145.32,   // 기준층 전용면적
    floorLeaseArea: 267.91, // 기준층 임대면적
    elevator: 4,
    parkingTotal: 67,
    deposit: 1050000,       // 보증금 (원/평)
    rent: 105000,           // 임대료 (원/평)
    mgmt: 47000,            // 관리비 (원/평)
    rentFree: 3.0,          // RF (개월/년)
    fitOut: 2.0,            // FO (개월)
    foMgmtFree: 0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "2, 9~11층",
    proposedLeaseArea: 1071.64,
    proposedNetArea: 581.28,
    enoc: 227148,
    monthlyTotal: 132036419,
    totalCost: 8186257960,  // 점유기간 총비용
    pros: ["을지로입구 인접 – 교통 편의성", "정사각형 모양의 평면 – 내부 활용도 증가"],
    cons: ["준공된 지 45년 이상 – 공용공간 시설 낙후", "중간층에 병/의원이 입주하여 전문 사무 건물 수준의 관리는 부족"],
    image: "https://images.unsplash.com/photo-1486406146926-c627a92ad1ab?w=600&h=400&fit=crop",
    floorPlanDesc: "정사각형 기준층, 코어 중앙 배치"
  },
  {
    id: 2,
    name: "씨티센터타워",
    region: "CBD",
    address: "서울시 중구 수표로 34",
    station: "2,3호선 을지로3가역 도보 4분",
    lat: 37.5645, lng: 126.9910,
    grossArea: 11273,
    netRatio: 72.25,
    floors: "B2/18F",
    builtYear: 1969,
    remodelYear: 2014,
    floorNetArea: 490.78,
    floorLeaseArea: 679.30,
    elevator: 26,
    parkingTotal: 91,
    deposit: 1300000,
    rent: 130000,
    mgmt: 47000,
    rentFree: 1.0,
    fitOut: 2.0,
    foMgmtFree: 0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "17층 전체",
    proposedLeaseArea: 679.30,
    proposedNetArea: 490.78,
    enoc: 224674,
    monthlyTotal: 110265729,
    totalCost: 6836475200,
    pros: ["높은 전용률로 인한 CBD권역 내 다소 저렴한 월임관리비"],
    cons: ["충무로 주변 이면도로 위치 (왕복 2차선)", "다소 부족한 내부 어메니티 및 주차공간"],
    image: "https://images.unsplash.com/photo-1554435493-93422e8220c8?w=600&h=400&fit=crop",
    floorPlanDesc: "장방형 기준층, 높은 전용률"
  },
  {
    id: 3,
    name: "ENA센터",
    region: "CBD",
    address: "서울시 중구 서소문로 120",
    station: "1,2호선 시청역 도보 1분",
    lat: 37.5635, lng: 126.9755,
    grossArea: 8566,
    netRatio: 55.27,
    floors: "B6/20F",
    builtYear: 2014,
    remodelYear: null,
    floorNetArea: 219.28,
    floorLeaseArea: 396.72,
    elevator: 4,
    parkingTotal: 81,
    deposit: 750000,
    rent: 75000,
    mgmt: 38000,
    rentFree: 0,
    fitOut: 1.0,
    foMgmtFree: 0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "6,8층 전체, 11층 일부",
    proposedLeaseArea: 1046.03,
    proposedNetArea: 568.62,
    enoc: 205612,
    monthlyTotal: 116915288,
    totalCost: 7131832540,
    pros: ["2호선 시청역 인접으로 뛰어난 접근성", "인테리어가 용이한 직사각형 평면도"],
    cons: ["다소 부족한 내부 어메니티 및 주차공간"],
    image: "https://images.unsplash.com/photo-1577495508048-b635879837f1?w=600&h=400&fit=crop",
    floorPlanDesc: "직사각형 평면, 인테리어 효율적"
  },
  {
    id: 4,
    name: "TOWER50",
    region: "CBD",
    address: "서울시 서대문구 충정로 50",
    station: "2,5호선 충정로역 도보 4분",
    lat: 37.5597, lng: 126.9640,
    grossArea: 9648,
    netRatio: 56.96,
    floors: "B4/17F",
    builtYear: 1989,
    remodelYear: 2025,
    floorNetArea: 401.95,
    floorLeaseArea: 705.62,
    elevator: 6,
    parkingTotal: 146,
    deposit: 1190000,
    rent: 119000,
    mgmt: 41000,
    rentFree: 1.0,
    fitOut: 4.0,
    foMgmtFree: 0,
    tiPerPy: 800000,
    contractYears: 5,
    proposedFloors: "3층 전체, 4층 일부",
    proposedLeaseArea: 870.44,
    proposedNetArea: 499.29,
    enoc: 237263,
    monthlyTotal: 118463006,
    totalCost: 7581632360,
    pros: ["2,5호선 충정로역 도보 5분 거리 위치", "최근 (2025년) 리모델링을 완료한 신규 물건"],
    cons: ["내부 기둥으로 인해 공간활용성 저하"],
    image: "https://images.unsplash.com/photo-1545324418-cc1a3fa10c00?w=600&h=400&fit=crop",
    floorPlanDesc: "리모델링 완료, 내부 기둥 존재"
  },
  {
    id: 5,
    name: "용산 더프라임타워",
    region: "CBD",
    address: "서울시 용산구 원효로90길 11",
    station: "1호선 남영역 도보 3분",
    lat: 37.5440, lng: 126.9720,
    grossArea: 11800,
    netRatio: 62.03,
    floors: "B6/31F",
    builtYear: 2014,
    remodelYear: 2025,
    floorNetArea: 240.91,
    floorLeaseArea: 418.85,
    elevator: 9,
    parkingTotal: 152,
    deposit: 1200000,
    rent: 120000,
    mgmt: 40000,
    rentFree: 4.5,
    fitOut: 6.0,
    foMgmtFree: 3.0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "14,15층 전체",
    proposedLeaseArea: 837.70,
    proposedNetArea: 527.76,
    enoc: 168828,
    monthlyTotal: 89100818,
    totalCost: 5880654000,
    pros: ["1호선 남영역 도보 2분, 6호선/경의중앙선 효창공원역 도보 15분", "최근 리모델링 완료, 저렴한 실질임대가"],
    cons: ["전통적인 사무 공간이 아닌 주거지역에 위치"],
    image: "https://images.unsplash.com/photo-1512917774080-9991f1c4c750?w=600&h=400&fit=crop",
    floorPlanDesc: "리모델링 완료, 넓은 전용면적"
  },
  {
    id: 6,
    name: "한샘상암사옥",
    region: "상암DMC",
    address: "서울시 마포구 성암로 179",
    station: "DMC역 도보 1분",
    lat: 37.5780, lng: 126.8910,
    grossArea: 20161,
    netRatio: 53.42,
    floors: "B5/22F",
    builtYear: 2007,
    remodelYear: null,
    floorNetArea: 508.00,
    floorLeaseArea: 958.58,
    elevator: 19,
    parkingTotal: 444,
    deposit: 630000,
    rent: 63000,
    mgmt: 35000,
    rentFree: 1.0,
    fitOut: 2.0,
    foMgmtFree: 0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "21층 전체",
    proposedLeaseArea: 958.58,
    proposedNetArea: 517.32,
    enoc: 168411,
    monthlyTotal: 87122553,
    totalCost: 5401598300,
    pros: ["6호선, 경의중앙선, 공항철도 DMC역과 도보 1~5분", "상징적 위치와 뛰어난 건물 스펙으로 상암 DMC 랜드마크", "넓은 바닥 면적 – 1개층 전체 사용 가능"],
    cons: ["지역 내 가장 높은 임대료 단가"],
    image: "https://images.unsplash.com/photo-1497366216548-37526070297c?w=600&h=400&fit=crop",
    floorPlanDesc: "대형 기준층, 1개층 전체 사용"
  },
  {
    id: 7,
    name: "상암 IT타워",
    region: "상암DMC",
    address: "서울시 마포구 월드컵북로 434",
    station: "경의중앙선 DMC역 도보 15분",
    lat: 37.5770, lng: 126.8870,
    grossArea: 13961,
    netRatio: 54.70,
    floors: "B5/12F",
    builtYear: 2007,
    remodelYear: null,
    floorNetArea: 706.62,
    floorLeaseArea: 1270.88,
    elevator: 8,
    parkingTotal: 356,
    deposit: 500000,
    rent: 50000,
    mgmt: 32000,
    rentFree: 2.0,
    fitOut: 3.0,
    foMgmtFree: 0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "3층 전체",
    proposedLeaseArea: 1270.88,
    proposedNetArea: 706.62,
    enoc: 128923,
    monthlyTotal: 91099906,
    totalCost: 5739294080,
    pros: ["넓은 바닥 면적 – 제안된 물건 중 가장 넓은 면적을 1개층 내 사용 가능", "신규 임차에 적극적인 임대인 – 추가 조건 협의 긍정적 (TI 및 렌트프리 등)", "하니웰, 카카오 등 국내/외 대형 임차인 입주"],
    cons: ["6호선, 경의중앙선, 공항철도 DMC역과 도보 15분 거리 위치 (마을버스 탑승 요)"],
    image: "https://images.unsplash.com/photo-1486325212027-8081e485255e?w=600&h=400&fit=crop",
    floorPlanDesc: "최대 바닥면적, 대형 오픈 플로어"
  },
  {
    id: 8,
    name: "상암 S-City",
    region: "상암DMC",
    address: "서울시 마포구 월드컵북로54길 25",
    station: "6호선,공항철도,경의선 DMC역 버스 6분",
    lat: 37.5760, lng: 126.8860,
    grossArea: 14106,
    netRatio: 54.65,
    floors: "B8/16F",
    builtYear: 2017,
    remodelYear: null,
    floorNetArea: 484.00,
    floorLeaseArea: 886.31,
    elevator: 6,
    parkingTotal: 290,
    deposit: 600000,
    rent: 60000,
    mgmt: 30000,
    rentFree: 1.0,
    fitOut: 1.0,
    foMgmtFree: 0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "7층 전체",
    proposedLeaseArea: 886.31,
    proposedNetArea: 484.00,
    enoc: 154003,
    monthlyTotal: 74537218,
    totalCost: 4546770300,
    pros: ["비교적 신축 건물 (2017년 준공)", "저렴한 주차비"],
    cons: ["6호선, 경의중앙선, 공항철도 DMC역과 도보 15분 거리 위치", "요구 면적 대비 다소 협소한 전용면적"],
    image: "https://images.unsplash.com/photo-1554435493-93422e8220c8?w=600&h=400&fit=crop",
    floorPlanDesc: "2017 신축, 표준 기준층"
  },
  {
    id: 9,
    name: "상암전자회관",
    region: "상암DMC",
    address: "서울시 마포구 월드컵북로54길 11",
    station: "경의중앙선 DMC역 버스 6분",
    lat: 37.5755, lng: 126.8850,
    grossArea: 6041,
    netRatio: 49.44,
    floors: "B5/12F",
    builtYear: 2007,
    remodelYear: null,
    floorNetArea: 232.00,
    floorLeaseArea: 469.00,
    elevator: 4,
    parkingTotal: 149,
    deposit: 462000,
    rent: 38500,
    mgmt: 28500,
    rentFree: 1.0,
    fitOut: 3.0,
    foMgmtFree: 0,
    tiPerPy: 0,
    contractYears: 5,
    proposedFloors: "2층 전체 / 5,6층 일부",
    proposedLeaseArea: 1126.16,
    proposedNetArea: 556.76,
    enoc: 125632,
    monthlyTotal: 69947049,
    totalCost: 4406664080,
    pros: ["제안된 물건 중 가장 저렴한 옵션"],
    cons: ["6호선, 경의중앙선, 공항철도 DMC역과 도보 15분", "지역 내 타 물건 대비 좁은 바닥면적 – 여러 층으로 분리 필요"],
    image: "https://images.unsplash.com/photo-1460317442991-0ec209397118?w=600&h=400&fit=crop",
    floorPlanDesc: "소형 기준층, 멀티플로어 사용"
  }
];
```

### 2.2 임차인 조건 (고정 데이터)

```javascript
const TENANT_CONDITIONS = {
  company: "SK네트웍스서비스",
  requiredNetArea: "전용 500평 이상",
  budget: "ENOC 25만원 이하",
  preferredRegion: ["CBD (도심권)", "상암 DMC"],
  contractPeriod: "5년",
  moveInDate: "2026년 8월 (Target)",
  currentLocation: "퍼시픽타워 (종로구)",
  headcount: "약 190명",
  priorities: ["비용 효율성", "교통 접근성", "건물 등급"]
};
```

### 2.3 시장 데이터 (고정)

```javascript
const MARKET_DATA = {
  cbd: { vacancyRate: 5.1, vacancyChange: 0.9, avgRent: 113000, avgMgmt: 45600 },
  gbd: { vacancyRate: 3.9, vacancyChange: 0, avgRent: 112701, avgMgmt: 40997 },
  ybd: { vacancyRate: 2.2, vacancyChange: -1.3, avgRent: 96066, avgMgmt: 40762 },
  bbd: { vacancyRate: 3.2, vacancyChange: 1.2, avgRent: 84800, avgMgmt: 31700 },
  others: { vacancyRate: 14.9, vacancyChange: 1.4, avgRent: 65419, avgMgmt: 32740 }
};
```

---

## 3. 페이지 구조

### 전체 레이아웃

```
┌─────────────────────────────────────────────────────────────┐
│  Top Header Bar (고정, h-16)                                 │
│  [GenstarMate 로고]  AI 임차인 리포트   [PDF] [공유] [설정]    │
├──────────┬──────────────────────────────────────────────────┤
│ Sidebar  │  Main Content (스크롤)                            │
│ (w-64)   │                                                  │
│ 고정     │  Section 0: Hero / 요약                           │
│          │  Section 1: 임차인 조건                            │
│ ● 요약    │  Section 2: 시장 개요                             │
│ ○ 조건    │  Section 3: 맵 뷰 ← ★ 핵심                       │
│ ○ 시장    │  Section 4: 빌딩 상세 카드                        │
│ ○ 맵뷰    │  Section 5: 비용 비교                             │
│ ○ 상세    │  Section 6: 주변환경                              │
│ ○ 비교    │  Section 7: AI 테스트핏 ← ★ 핵심                  │
│ ○ 주변    │  Section 8: 향후 일정                             │
│ ○ 테스트핏│                                                  │
│ ○ 일정    │                                                  │
└──────────┴──────────────────────────────────────────────────┘
```

### 반응형 규칙
- Desktop (≥1280px): 사이드바 + 메인 콘텐츠
- Tablet (768~1279px): 사이드바 접기/펼치기 (햄버거)
- Mobile (<768px): 사이드바 숨김, 하단 탭 네비

---

## 4. 섹션별 구현 상세

### Section 0: Hero / 요약 카드

**목적**: 첫 화면에서 리포트의 핵심 결과를 즉시 보여줌

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│   SK네트웍스서비스 님을 위한                            │
│   AI 오피스 이전 분석 리포트                            │
│                                                      │
│   GenstarMate AI가 42개 빌딩을 분석하여                 │
│   9개 최적 후보지를 선정했습니다.                        │
│                                                      │
│   ┌──────┐  ┌──────┐  ┌──────┐                      │
│   │ 🥇   │  │ 🥈   │  │ 🥉   │   ← TOP 3 추천      │
│   │상암전자│  │상암IT │  │더프라임│                      │
│   │125,632│  │128,923│  │168,828│   ← ENOC           │
│   │원/평  │  │원/평  │  │원/평  │                      │
│   └──────┘  └──────┘  └──────┘                      │
│                                                      │
│   2026.03.24 생성  |  GenstarMate LM본부               │
└──────────────────────────────────────────────────────┘
```

- 배경: 그라데이션 (navy → dark blue) 또는 서울 스카이라인 배경 + 오버레이
- TOP 3 카드: ENOC가 가장 낮은 3개 빌딩 자동 표시
- 카드에 빌딩 이미지 썸네일 + ENOC 수치 크게
- "AI가 분석했습니다" 텍스트에 타이핑 애니메이션 효과
- 스크롤 유도 화살표 (bounce 애니메이션)

### Section 1: 임차인 조건 요약

**목적**: 이 리포트가 어떤 조건 기반인지 한눈에 파악

- `TENANT_CONDITIONS` 데이터를 **칩(Chip)/태그** 형태로 나열
- 카드 배경: 연한 파란색 (bg-blue-50)
- 각 조건 옆에 아이콘 표시 (MapPin, DollarSign, Calendar, Users 등)
- 비활성화된 "조건 수정" 버튼 (목업이므로 클릭 불가, 툴팁으로 "Pro 버전에서 사용 가능" 표시)

### Section 2: 시장 개요

**목적**: 서울 오피스 시장 현황을 차트로 간결하게 표시

- **권역별 공실률 카드**: CBD/GBD/YBD/BBD/Others 5개 카드 가로 나열
  - 각 카드에 공실률 수치 (크게) + 전분기 대비 변동 (▲▼ 색상)
- **임대료 바 차트**: Recharts `BarChart`로 권역별 평균 임대료 비교
- 작은 캡션: "출처: 2025.4Q GenstarMate Office Market Report"
- 간결하게. 2~3개 차트 이내.

### Section 3: 후보지 맵 뷰 ★★★

**이 섹션이 가장 중요합니다.**

**구현 방법**: Leaflet.js를 CDN으로 로드하여 사용 (Google Maps API 키 불필요)

```html
<!-- CDN (index.html 또는 artifact 내 <script> 태그) -->
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

**React 내부 구현**: `useEffect`에서 Leaflet 맵 초기화

**맵 기능**:
1. 서울 전체가 보이는 줌 레벨로 초기화 (center: [37.56, 126.94], zoom: 12)
2. 9개 빌딩을 마커로 표시
3. **마커 색상**: ENOC 기준으로 분류
   - 초록 (≤150,000): 상암전자회관, 상암IT타워, S-City
   - 노랑 (150,001~200,000): 더프라임타워, 한샘상암사옥
   - 빨강 (≥200,001): 교원명동, 씨티센터, ENA, TOWER50
4. **마커 클릭 시 팝업**: 빌딩명, ENOC, 전용면적, "상세보기" 버튼
5. **우측 사이드 패널 (맵 위 오버레이)**: 빌딩 리스트 (스크롤), 클릭 시 해당 마커로 이동
6. **상단 필터 바**: 권역 필터 (CBD / 상암DMC / 전체)

**레이아웃**:
```
┌─────────────────────────────────────────────────┐
│  [전체 v] [CBD] [상암DMC]   정렬: [ENOC순 v]     │  ← 필터바
├───────────────────────────────┬─────────────────┤
│                               │ 빌딩 리스트      │
│                               │ ┌─────────────┐ │
│    [ Leaflet 지도 ]            │ │상암전자회관   │ │
│    📍 📍 📍                   │ │ENOC 125,632 │ │
│                               │ │전용 556평    │ │
│         📍 📍 📍              │ └─────────────┘ │
│              📍               │ ┌─────────────┐ │
│                               │ │상암IT타워    │ │
│          📍  📍               │ │ENOC 128,923 │ │
│                               │ └─────────────┘ │
│                               │ ...             │
├───────────────────────────────┴─────────────────┤
│  9개 후보지  |  평균 ENOC: 178,488원/평           │
└─────────────────────────────────────────────────┘
```

**중요**: 맵이 화면의 최소 60vh 이상 높이를 차지해야 합니다.

### Section 4: 빌딩 상세 카드

**목적**: 각 빌딩의 풀 정보를 카드로 깔끔하게 표시

**카드 그리드**: 1열 (모바일) / 2열 (태블릿) / 기본 1열 풀너비 (데스크톱에서 좌우 분할)

**각 카드 레이아웃**:
```
┌──────────────────────────────────────────────────────┐
│ ┌─────────────────┐  빌딩명              ENOC 뱃지    │
│ │                 │  주소                             │
│ │  빌딩 이미지     │  역 도보 N분                      │
│ │  (300x200)      │  ─────────────────────────────── │
│ │                 │  연면적    4,006평  │ 보증금  1,050,000│
│ │  [A등급 뱃지]    │  규모     B3/15F  │ 임대료    105,000│
│ └─────────────────┘  전용률   54.24%  │ 관리비     47,000│
│                      준공    1979    │ RF      3.0개월/년│
│                      주차    67대    │ FO       2.0개월  │
│ ──────────────────────────────────────────────────── │
│  ✅ Pros                      ❌ Cons                 │
│  • 교통 편의성                  • 시설 낙후              │
│  • 정사각형 평면도               • 관리 수준 부족         │
│ ──────────────────────────────────────────────────── │
│  [🏗️ 테스트핏 생성]    [📊 비교에 추가]    [📋 상세보기]  │
└──────────────────────────────────────────────────────┘
```

- 이미지: Unsplash placeholder 사용 (빌딩 이미지)
- ENOC 뱃지: 원형 또는 둥근 사각형, 색상은 가격대별 (초록/노랑/빨강)
- Pros는 초록 배경 태그, Cons는 빨간 배경 태그
- "비교에 추가" 클릭 시 체크 상태 토글 (state 관리)
- "테스트핏 생성" 클릭 시 Section 7로 스크롤 + 해당 빌딩 선택 상태

### Section 5: 비용 비교

**목적**: 선택한 빌딩들의 비용을 차트로 한눈에 비교

**차트 1: ENOC 바 차트**
- Recharts `BarChart`
- X축: 빌딩명 (9개)
- Y축: ENOC (원/평)
- 바 색상: 가격대별 그라데이션 (초록→노랑→빨강)
- 평균선 표시 (ReferenceLine)

**차트 2: 월 임관리비 바 차트**
- X축: 빌딩명
- Y축: 월 임관리비 (원/월)
- 금액은 `toLocaleString()`으로 포맷

**하단: 비교 테이블**
- 가로 스크롤 테이블
- 행: 임대면적/전용면적/제안층/보증금/임대료/관리비/RF/FO/TI/ENOC/월합계
- 열: 각 빌딩
- 가장 유리한 수치에 초록 하이라이트

### Section 6: 주변 환경 (간략)

**목적**: 후보 빌딩 주변의 편의시설/교통 정보

- **간략하게 구현** — 2~3개 빌딩만 샘플로 표시
- 각 빌딩별: 도보 5분 내 편의시설 아이콘 + 개수
  - 🍽️ 음식점 32개 / ☕ 카페 18개 / 🏦 은행 5개 / 🏥 병원 3개 / 🏪 편의점 8개
- 하드코딩된 목업 데이터 사용
- "AI가 공공데이터를 분석하여 생성했습니다" 캡션 표시

### Section 7: AI 테스트핏 ★★★

**이 섹션이 두 번째로 중요합니다.**

**목적**: AI가 평면도 기반 레이아웃을 생성하는 것처럼 보이는 UI

**구현**:

```
┌──────────────────────────────────────────────────────┐
│  🏗️ AI 오피스 레이아웃 시뮬레이터                       │
│  선택한 빌딩의 평면도에 맞춰 AI가 공간을 배치합니다.      │
│                                                      │
│  ┌─── 조건 입력 ────────────────────────────────────┐ │
│  │ 대상 빌딩: [교원명동빌딩 ▾]                        │ │
│  │ 총 인원:   [190명        ]                        │ │
│  │ 부서 수:   [5개          ]                        │ │
│  │                                                  │ │
│  │ 좌석 유형: ◉ 오픈데스크  ○ 파티션  ○ 혼합           │ │
│  │ 회의실:    소(4인) [2]개  중(8인) [3]개  대(16인) [1]개│ │
│  │ 대표실:    [✓] 20평                               │ │
│  │ 탕비실:    [✓] 4평                                │ │
│  │ 라운지:    [✓] 22평                               │ │
│  │                                                  │ │
│  │ 분위기:  [모던] [내추럴] [인더스트리얼] [미니멀]      │ │
│  │                                                  │ │
│  │          [ 🤖 AI 레이아웃 생성하기 ]                │ │
│  └──────────────────────────────────────────────────┘ │
│                                                      │
│  (버튼 클릭 시)                                       │
│                                                      │
│  ┌─── 생성 결과 ────────────────────────────────────┐ │
│  │                                                  │ │
│  │  ┌──────────────┐  ┌──────────────┐             │ │
│  │  │ [평면도 이미지]│  │[3D 조감도]    │             │ │
│  │  │  2D Layout   │  │  3D View     │             │ │
│  │  └──────────────┘  └──────────────┘             │ │
│  │                                                  │ │
│  │  공간 배분 요약:                                   │ │
│  │  업무공간 62% | 회의 15% | 지원 13% | 동선 10%     │ │
│  │  총 좌석: 189석 | 좌석당 면적: 2.8평               │ │
│  │                                                  │ │
│  └──────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

**AI 생성 시뮬레이션**:
1. "생성하기" 버튼 클릭
2. 상태를 `generating`으로 변경
3. 프로그레스바 + 단계별 텍스트 애니메이션:
   - "평면도 분석 중..." (2초)
   - "공간 배치 최적화 중..." (2초)
   - "인테리어 렌더링 중..." (2초)
   - "완료!" (0.5초)
4. 미리 준비된 결과 이미지 표시 (Unsplash 오피스 레이아웃 이미지)
5. 공간 배분 요약 데이터 표시 (하드코딩)

**결과 이미지**: 아래 Unsplash URL 사용
```
2D 레이아웃: https://images.unsplash.com/photo-1497366216548-37526070297c?w=800&h=600&fit=crop
3D 조감도: https://images.unsplash.com/photo-1497215842964-222b430dc094?w=800&h=600&fit=crop
```

### Section 8: 향후 일정

**목적**: 임대 추진 타임라인 표시

- 가로 타임라인 UI (CSS flexbox/grid)
- 단계: 물건검색 → 현장실사 → LOI제출 → 계약체결 → 인테리어 → 입주
- 각 단계에 날짜 (2026.02 ~ 2026.09)
- 현재 단계 하이라이트 (물건검색 완료, 현장실사 진행중)

---

## 5. 디자인 시스템

### 5.1 컬러

```css
:root {
  --color-primary: #1B3A5C;       /* Deep Navy — 헤더, 사이드바, 타이틀 */
  --color-secondary: #2E75B6;     /* Brand Blue — 링크, 활성탭, 차트 */
  --color-accent: #00A3E0;        /* Bright Blue — CTA 버튼, 하이라이트 */
  --color-bg: #F8FAFC;            /* Off White — 페이지 배경 */
  --color-card: #FFFFFF;          /* White — 카드 배경 */
  --color-text: #1E293B;          /* Slate 800 — 본문 */
  --color-text-sub: #64748B;      /* Slate 500 — 보조 텍스트 */
  --color-border: #E2E8F0;        /* Slate 200 — 테두리 */
  --color-success: #10B981;       /* Emerald — Pros, 저렴 */
  --color-warning: #F59E0B;       /* Amber — 주의, 중간가 */
  --color-danger: #EF4444;        /* Red — Cons, 고가 */
}
```

### Tailwind 매핑
- Primary: `bg-slate-800`, `text-slate-800`
- Secondary: `bg-blue-600`, `text-blue-600`
- Accent: `bg-sky-500`
- Success: `bg-emerald-500`
- Warning: `bg-amber-500`
- Danger: `bg-red-500`
- Card: `bg-white rounded-xl shadow-sm border border-slate-200`

### 5.2 폰트

구글 폰트 CDN 사용:
```
Pretendard 대안으로 Noto Sans KR 사용 (CDN 가능)
```

Tailwind 기본 `font-sans`를 사용하되, 숫자 강조에는 `tabular-nums` 적용.

### 5.3 컴포넌트 스타일

| 컴포넌트 | Tailwind Classes |
|---------|-----------------|
| 카드 | `bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden` |
| 카드 호버 | `hover:shadow-lg hover:-translate-y-0.5 transition-all duration-200` |
| 헤더바 | `bg-slate-800 text-white h-16 flex items-center px-6 fixed top-0 z-50` |
| 사이드바 | `bg-slate-900 text-slate-300 w-64 fixed left-0 top-16 bottom-0 overflow-y-auto` |
| 사이드바 활성 | `bg-slate-700 text-white border-l-2 border-sky-400` |
| 버튼 Primary | `bg-sky-500 hover:bg-sky-600 text-white px-6 py-2.5 rounded-lg font-medium` |
| 버튼 Secondary | `bg-white border border-slate-300 hover:bg-slate-50 text-slate-700 px-4 py-2 rounded-lg` |
| 뱃지 (Pros) | `bg-emerald-50 text-emerald-700 px-3 py-1 rounded-full text-sm` |
| 뱃지 (Cons) | `bg-red-50 text-red-700 px-3 py-1 rounded-full text-sm` |
| ENOC 뱃지 | `px-3 py-1.5 rounded-lg font-bold text-lg` + 색상별 bg |
| 섹션 타이틀 | `text-2xl font-bold text-slate-800 mb-1` + 밑줄 accent |
| 필터 칩 | `bg-blue-50 text-blue-700 px-3 py-1 rounded-full text-sm cursor-pointer` |
| 테이블 헤더 | `bg-slate-50 text-slate-600 text-xs uppercase tracking-wider` |
| 테이블 셀 | `px-4 py-3 text-sm border-b border-slate-100` |

### 5.4 이미지 처리

- 모든 빌딩 이미지: `object-cover rounded-t-xl` (카드 상단)
- 이미지 높이: 고정 `h-48` 또는 `aspect-[3/2]`
- 로딩: `bg-slate-100 animate-pulse` placeholder (이미지 로드 전)
- 확대: 클릭 시 Lightbox 모달 (선택적 구현)

### 5.5 애니메이션

| 효과 | 적용 위치 | 구현 |
|-----|---------|-----|
| 타이핑 효과 | Hero 섹션 텍스트 | CSS `steps()` animation + `border-right` 커서 |
| 카운트업 | ENOC 수치 | `useEffect` + `setInterval`로 0에서 목표값까지 증가 |
| 스크롤 페이드인 | 각 섹션 진입 시 | `IntersectionObserver` + `opacity`/`translateY` 트랜지션 |
| 프로그레스바 | 테스트핏 생성 중 | CSS `width` transition + `setInterval`로 %증가 |
| 바운스 화살표 | Hero 하단 | `animate-bounce` (Tailwind) |
| 마커 드롭 | 지도 마커 | Leaflet marker 기본 애니메이션 |
| 카드 호버 | 빌딩 카드 | `transition-all duration-200` |

---

## 6. 구현 우선순위

파일이 크므로 **단계별로 구현**합니다.

### Step 1: 뼈대 + 네비게이션 (필수)
- 헤더바, 사이드바, 메인 콘텐츠 영역
- 사이드바 클릭 시 해당 섹션으로 스크롤
- 스크롤 위치에 따라 사이드바 활성 항목 변경

### Step 2: Hero + 조건 요약 (필수)
- Section 0, Section 1

### Step 3: 맵 뷰 (필수, 핵심)
- Leaflet 지도 + 9개 마커 + 클릭 팝업
- 우측 리스트 패널
- 권역 필터

### Step 4: 빌딩 상세 카드 (필수)
- 9개 빌딩 카드 렌더링
- Pros/Cons 표시

### Step 5: 비용 비교 차트 (필수)
- ENOC 바 차트
- 월 임관리비 바 차트

### Step 6: AI 테스트핏 UI (필수)
- 조건 입력 폼
- "생성 중" 프로그레스 애니메이션
- 결과 표시

### Step 7: 나머지 섹션 (선택)
- 시장 개요
- 주변 환경
- 향후 일정
- 추가 애니메이션

---

## 7. 기술 제약 및 주의사항

### React Artifact 제약
- **localStorage/sessionStorage 사용 불가** — React state만 사용
- **외부 라이브러리**: recharts, lucide-react, lodash, d3 사용 가능
- **Leaflet**: CDN 로드 필요 — `useEffect` 내에서 동적 로드
- **단일 파일**: 모든 코드가 하나의 .jsx 파일에 포함
- **기본 export**: `export default function App() {}` 형태

### Leaflet 통합 방법

React 컴포넌트 내에서 Leaflet을 사용하려면:

```jsx
// 1. useEffect에서 Leaflet CSS/JS 동적 로드
useEffect(() => {
  // CSS
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.css';
  document.head.appendChild(link);

  // JS
  const script = document.createElement('script');
  script.src = 'https://unpkg.com/leaflet@1.9.4/dist/leaflet.js';
  script.onload = () => {
    // Leaflet 로드 완료 후 맵 초기화
    initMap();
  };
  document.head.appendChild(script);
}, []);

// 2. 맵 초기화 함수
const initMap = () => {
  const map = window.L.map('map-container').setView([37.56, 126.94], 12);
  window.L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
  }).addTo(map);

  // 마커 추가
  BUILDINGS.forEach(b => {
    const color = b.enoc <= 150000 ? '#10B981' : b.enoc <= 200000 ? '#F59E0B' : '#EF4444';
    const marker = window.L.circleMarker([b.lat, b.lng], {
      radius: 10,
      fillColor: color,
      color: '#fff',
      weight: 2,
      fillOpacity: 0.9
    }).addTo(map);

    marker.bindPopup(`
      <div style="min-width:200px">
        <strong>${b.name}</strong><br/>
        <span>ENOC: ${b.enoc.toLocaleString()}원/평</span><br/>
        <span>전용: ${b.proposedNetArea}평</span>
      </div>
    `);
  });
};

// 3. JSX에서 div 렌더링
<div id="map-container" style={{ height: '500px', width: '100%' }} />
```

### 숫자 포맷 헬퍼

```javascript
const fmt = (n) => n.toLocaleString('ko-KR');
const fmtWon = (n) => `₩${fmt(n)}`;
const fmtEnoc = (n) => {
  if (n <= 150000) return { text: fmt(n), color: 'text-emerald-600', bg: 'bg-emerald-50' };
  if (n <= 200000) return { text: fmt(n), color: 'text-amber-600', bg: 'bg-amber-50' };
  return { text: fmt(n), color: 'text-red-600', bg: 'bg-red-50' };
};
```

---

## 8. 최종 체크리스트

구현 완료 후 확인할 항목:

- [ ] 헤더에 "GenstarMate" 로고 텍스트 + "AI 임차인 리포트" 표시
- [ ] 사이드바 네비게이션이 스크롤에 따라 활성 항목 변경
- [ ] Hero 섹션에 TOP 3 추천 빌딩 카드 표시
- [ ] **지도에 9개 빌딩 마커가 정상 표시** (ENOC별 색상)
- [ ] 마커 클릭 시 팝업 표시
- [ ] 권역 필터 (전체/CBD/상암) 작동
- [ ] 빌딩 상세 카드 9개 모두 렌더링
- [ ] 카드에 이미지, 스펙, 임대조건, Pros/Cons 표시
- [ ] ENOC 바 차트 정상 렌더링
- [ ] 월 임관리비 바 차트 정상 렌더링
- [ ] AI 테스트핏: 조건 입력 폼 표시
- [ ] 테스트핏 "생성" 버튼 클릭 시 프로그레스 애니메이션
- [ ] 테스트핏 완료 시 결과 이미지 + 요약 데이터 표시
- [ ] 전체적으로 깔끔하고 프로페셔널한 디자인
- [ ] 이미지가 잘 보이는 레이아웃 (잘리거나 찌그러지지 않음)
- [ ] 숫자 포맷 정상 (천단위 콤마, 원 단위)

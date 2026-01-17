# 2025 3Q–4Q Self Review (Draft)

> 아래 내용은 2025년 3분기(3Q)~4분기(4Q) 리뷰 기간 동안의 업무 성과(Accomplishments)와 성장/개선(Growth/Improvement) 항목을 **기능/개념 단위로 가능한 한 빠짐없이** 정리한 버전이다.
>
> - **원칙**: “무엇을 했는가(Deliverable) / 왜 중요한가(Impact) / 어떻게 했는가(Approach) / 결과(Outcome) / 협업(Alignment)” 흐름이 보이도록 작성함.

---

## 1. Accomplishments![[search-core/프로젝트 구조]]

### A. Sound System (3Q~4Q)

#### A-1. New Sound System: Multi Sound Zone 구현 (3Q)
- **Deliverable**: Multi Sound Zone 기능 구현 및 universe,  Robotics infraStructure 과 연동 
- **Key Actions**
  - 사운드 기능 구현 범위를 Robotics system 코드에 국한하지 않고, 필요 시 **annotation server등 수정이 필요한 부분을 파악하여 직접 구현 및 담당자 확인 및 요청을 **통하여 end-to-end로 동작하도록 완성
  - 기존 사운드 처리 코드 구조를 이해하고, Multi-zone 동작을 위해 데이터 흐름/상태 전이를 설계하여 적용
- **Impact/Outcome**
  - 다중 존 환경에서 일관된 사운드 동작을 가능하게 하여 사용자 경험 및 시스템 확장성 확보

#### A-2. Sound Profile Update: Non-Reboot 업데이트 기능 추진 및 실행 (3Q)
- **Deliverable**: 리부트 없이 사운드 프로파일 업데이트가 가능하도록 기능 기획/추진/개발 지원
- **Key Actions**
  - 사용자 편의성과 더불어 **개발/QA 과정의 테스트·디버깅 시간을 줄이는 효과**를 근거로 PM에게 필요성을 설득하고, 기능이 실제 프로젝트로 진행되도록 드라이브
  - 진행 과정에서 **관련 팀/작업 범위/역할 분담**을 정리하여 PM에게 전달, 프로젝트 진행 방향성을 제시
  - 각 팀의 진행 상황을 지속적으로 파악하고, 내 개발 내용 및 요구 사항을 수시 공유하여 참여자들이 동일한 정보로 움직이도록 정렬
- **Impact/Outcome**
  - 기능 요구가 “아이디어” 단계에서 끝나지 않고 실제 릴리즈 가능한 방향으로 추진되도록 기여
  - 리부트 없는 업데이트라는 UX/운영 개선 포인트를 확보하고, 협업 프로젝트로서의 실행력을 높임

#### A-3. New Sound System: Customization 및 Non-Reboot Sync를 위한 데이터 플로우 이해/가이드 (4Q)
- **Deliverable**: Sound Customization 기능 대응 + Non-Reboot Sync를 위한 데이터 흐름 파악 및 이슈 트리아지
- **Key Actions**
  - **RI ↔ Universe ↔ Robot** 사이드의 데이터 플로우/구조를 전체적으로 이해하기 위해 Universe 팀/RI 팀과 집중적으로 소통
  - 데이터가 로봇으로 전달되기까지의 단계별 인터페이스/연결 구조/송수신 방식을 확인 및 정리
  - 구현/테스트 중 문제 발생 시 TL/PM에게 원인을 설명하고, **해결 주체(어느 팀이 해결해야 하는지)**까지 제안하여 이슈 처리 속도 향상
  - 팀원들에게 전체 흐름을 가이드하여, 단일 담당자의 암묵지로 남지 않도록 공유
- **Impact/Outcome**
  - 기능 구현 중 발생하는 cross-team 이슈에 대해 “원인/주체/해결 방향”을 빠르게 제시할 수 있는 기반 확보
  - 데이터 흐름에 대한 팀의 이해도를 끌어올려 전체 개발 효율 향상

---

### B. LiDAR Driver / Parser (3Q)

#### B-1. 제조사 SDK 의존성 문제 제기 및 대체 구현 방향 제시
- **Problem Statement**
  - 기존 제조사 제공 data parser 및 network SDK는
    - 코드 가독성이 낮고,
    - 우리 상황에 맞춘 데이터 처리/전처리/통신 방식 변경이 어려우며,
    - Ethernet/USB 연결 변화에 대한 대응이 용이하지 않은 등 코드 품질/확장성 이슈가 존재
- **Action**
  - 위 문제를 근거로 “내부에서 통제 가능한 Driver 구현” 필요성을 제기하고, 실제 구현으로 해결

#### B-2. Servi Q Pacecat LiDAR Driver 자체 제작 (Network 연결 + 데이터 파싱)
- **Deliverable**: Pacecat LiDAR driver 및 parser 구현
- **Key Actions**
  - **Data Parsing**
    - LiDAR network packet을 parsing하여 x,y point 좌표 데이터 도출
    - 도출 데이터를 ROS sensor data format으로 변환하여 출력
    - RViz에서 데이터 시각화까지 end-to-end로 확인 가능하도록 연결
  - **Network**
    - destination IP/port를 사용자가 실행 시 설정 가능하도록 구성
    - 제조사 SDK에서 동작하지 않았던 LiDAR IP 변경 기능을 해결하기 위해
      - LiDAR IP/PORT 세팅 명령 packet 구조를 분석
      - parser 프로그램에 설정 기능을 추가
- **Impact/Outcome**
  - 제조사 SDK 의존도를 낮추고 내부 품질/가독성/확장성을 확보
  - 현장 이슈(동작 불가 기능)를 분석 기반으로 해결하며 안정성 개선

---

### C. Cart Carrier: Sound/UX 개선 및 Error 전파 구조 개선 (3Q)

#### C-1. Stuck 상황 알림음 기능 추가
- **Deliverable**: Stuck 상황에서 사용자 알림음 발생
- **Impact/Outcome**
  - 사용자에게 시스템 상태를 더 빠르게 인지시키는 UX 개선

#### C-2. Error 상태 전파 문제 분석 및 더 확실한 사용자 알림 구조 제안
- **Problem Statement**
  - 프로세스/시스템 에러 상태가 사용자에게 충분히 전파되지 않는 문제 존재
- **Key Actions**
  - 현재 Error 처리 흐름과 문제점을 파악하고 공유
  - 사용자에게 가장 확실하고 논리적인 전달 방식으로서
    - 터치스크린에서 에러 상태를 관리/노출하는 방법을 제안하여 요구사항 해결 방향 제시
- **Impact/Outcome**
  - 단순한 증상 해결이 아니라 “에러 전달 구조” 자체를 개선하는 방향을 제시

---

### D. Servi Q – AD Display 프로젝트 주도 및 기반 구축 (4Q)

#### D-1. 프로젝트 소유권(Ownership) 및 RS 주도 제안
- **Deliverable**: AD display 컴포넌트 개발을 RS에서 주도하도록 방향 설정
- **Rationale**
  - 터치스크린 영상 렌더링 자원 사용량이 높아 최적화가 필요
  - 하드웨어 수준의 세세한 제어가 요구됨
  - 신규 장비 추가가 포함됨 → RS 영역으로 판단
- **Impact/Outcome**
  - 책임 소재 및 실행 조직을 명확히 하여 프로젝트 진행 속도와 의사결정 품질을 확보

#### D-2. 초기 기능 빠른 구현 및 성능/리스크 탐색
- **Deliverable**: 빠른 영상 출력 및 초기 요구사항 만족
- **Key Actions**
  - 성능을 조기 파악하여 ARM 시스템에서 발생하는 CPU 사용량 문제를 확인
  - 하드웨어 디코더 적용 방법 및 후보 라이브러리 비교 검증
  - 다양한 테스트를 통해 적합한 기술 스택 및 SW 구조를 빠르게 설계
- **Impact/Outcome**
  - “일단 만들기”가 아니라 초기부터 성능/아키텍처/운영 리스크를 포함해 방향을 결정

#### D-3. 프로젝트 운영(Execution) 안정화: 업무 분배 + 데일리 스크럼
- **Deliverable**: 참여 인원들의 실행 구조를 구축하여 문제 장기화를 방지
- **Key Actions**
  - 다른 팀원들에게 업무를 할당하고, 현상 파악을 주기적으로 하기 위해 데일리 스크럼 수행
  - 이슈가 길게 지속되는 것을 근본적으로 막고, 의사결정/실행을 단축
- **Impact/Outcome**
  - 프로젝트 리듬과 진행 가시성을 확보하여 일정/품질 리스크를 낮춤

#### D-4. 빌드 시스템/운영 리스크 선제 대응 및 합의된 해결책 도출
- **Deliverable**: 새로운 시스템 도입 시 발생 가능한 빌드/운영 혼란을 사전에 차단
- **Key Actions**
  - 빌드 시스템 문제를 선제적으로 인지하고 다른 팀 및 하이레벨 개발자들과 적극 소통
  - 합의된 해결 방법을 조기 도출하여 팀에 공유
- **Impact/Outcome**
  - 개발 완료 이후 “적용/빌드/운영” 단계에서 생길 수 있는 병목을 선제적으로 제거

#### D-5. 문서 체계 구축: 요구사항 문서화 + Design Doc 템플릿 제작/확산
- **Problem Statement**
  - SW 설계 문서도 없고 요구사항이 정리된 문서도 없던 상황
- **Key Actions**
  - PM에게 요구사항 목록화를 위한 Confluence 문서 생성/운영을 요청하여 실제 적용되도록 함
  - SW 구조 설계서 문서 양식을 자체 제작하여 팀에 공유하고 작성 요청
  - feature owner가 긍정적으로 수용하여 Design Doc을 내가 만든 포맷/초안 기반으로 작성하기로 합의
- **Impact/Outcome**
  - 프로젝트 진행이 회의 중심이 아니라 문서 기반으로 정리되며 의사결정/추적성이 향상
  - 이후 유사 프로젝트에서도 재사용 가능한 템플릿 자산 확보

---

### E. Servi Q – Issue 대응 및 Bring-up 지원 (4Q)

#### E-1. 신규 로봇 bring-up/개발 환경 확보 지원
- **Deliverable**: 신규 로봇 OS 설치 이후 Pennybot SW 설치 확인 및 하드웨어 연결 점검, 부팅 확인
- **Key Actions**
  - 개발을 위한 로봇이 정상 가동되도록 준비 작업에 적극 참여
  - 문제 발생 시 해결하여 개발이 막히지 않도록 지원
- **Impact/Outcome**
  - 개발 생산성에 직접적으로 영향을 주는 기반 작업을 빠르게 안정화

#### E-2. 팀 간 HW 설정/구성 이해를 통한 문제 해결력 강화
- **Key Actions**
  - 내 개발 분야뿐 아니라 각 팀의 하드웨어 설정과 구성 일부를 이해하기 위해 팀원들과 적극 소통
- **Impact/Outcome**
  - bring-up/현장 이슈에서 원인 파악 및 해결 협업 속도 향상

---

### F. Cross-layer Problem Solving / Execution Attitude (3Q~4Q)

#### F-1. 내 스코프를 넘어 타 레이어까지 이해하고 원인/해결 주체를 제시하는 역량 강화
- **Key Actions**
  - Universe/Cloud infra/Robotics 등 다른 레이어의 작업과 이슈도 이해하고 해결 방법까지 전달받은 뒤 스스로 소화하려고 노력
  - 이슈 발생 시 “어디가 원인인지/어디에서 해결 가능한지”를 가늠하고 PM/TL에게 설명 가능하도록 역량 강화
- **Impact/Outcome**
  - 이슈 등록/할당 초기 단계에서 트리아지까지 일부 감당할 수 있는 수준의 실행력 확보

#### F-2. 기술부채/비효율 발견 시 공유 → 근본 해결책 설계 → 실제 적용까지 드라이브
- **Key Actions**
  - 비효율적 구성/상황 발견 시 팀 내 임시 처리로 끝내지 않고 관련자에게 공유/문의
  - 근본적인 해결책을 고안하고 단순 논의가 아니라 실제 적용까지 이어지도록 지속적으로 드라이브
- **Impact/Outcome**
  - 단기 대응이 아니라 장기 유지보수성과 조직 효율 관점에서 개선을 추진

#### F-3. 도움 제공의 재현성 확보: 문서화/공유로 팀이 함께 성장하도록 유도
- **Key Actions**
  - 내가 도움 줄 수 있는 이슈는 외면하지 않고 적극적으로 지원
  - 단, 내 개인 역량으로만 유지되는 구조가 되지 않도록 문서화/공유하여 팀이 같이 성장하도록 함

---

## 2. Growth / Improvement

### G-1. 시스템 레벨(End-to-End) 사고 및 아키텍처 이해 확장
- **Current Strength (This Period)**
  - Sound customization/non-reboot sync, AD display 등에서 로봇 단의 구현을 넘어 **Universe/RI/Robot 전체 데이터 흐름과 책임 분리를 이해**하려고 노력했고 실제 이슈 트리아지에 활용함.
- **Growth Goal**
  - 앞으로는 cross-layer 이슈에서
    - “문제 재현 → 원인 후보 축소 → 소유 팀 정의 → 해결안 제안”을 더 구조적으로 수행할 수 있도록
    - observability(로그/메트릭/트레이스) 관점의 분석 능력까지 확장할 필요가 있음.
- **Action Plan**
  - 주요 데이터 플로우/프로토콜에 대한 문서화를 정기적으로 수행
  - 이슈 트리아지 체크리스트(증상/로그 위치/재현 조건/소유 팀 가이드)를 만들고 팀 표준으로 확장

### G-2. 하드웨어/미디어 파이프라인 성능 최적화 역량 강화
- **Current Strength (This Period)**
  - AD display에서 ARM 환경 CPU 사용량, HW decoder 적용, 라이브러리 선택 등 성능 이슈를 조기 검증하고 구조를 설계함.
- **Growth Goal**
  - 미디어 파이프라인에서
    - 프레임 드롭/latency/buffering
    - HW acceleration 적용 기준
    - 리소스 프로파일링/병목 분석
    을 더 체계적으로 수행할 수 있도록 성능 엔지니어링 역량을 강화해야 함.
- **Action Plan**
  - profiling 도구/방법론 표준화(측정 지표, 실험 방식, 비교 템플릿)
  - 대표적인 파이프라인 구성(GStreamer/FFmpeg/OpenGL/DRM 등)별 베스트 프랙티스 학습 및 내부 공유

### G-3. Driver/Network 프로토콜 분석 및 신뢰성(robustness) 강화
- **Current Strength (This Period)**
  - LiDAR driver를 직접 구현하며 packet 구조 분석 및 네트워크 설정 문제를 해결함.
- **Growth Goal**
  - 앞으로는 device integration에서
    - fault tolerance(재연결, timeout, backoff)
    - packet loss/ordering
    - 설정/상태 동기화
    를 고려한 견고한 driver 구조를 표준화할 필요가 있음.
- **Action Plan**
  - driver 개발 시 공통 템플릿(연결 관리/에러 처리/설정 관리/테스트)을 만들고 재사용
  - 통신/프로토콜 관련 테스트(시뮬레이션, replay, fuzzing) 방향을 팀에 제안

### G-4. 문서/프로세스 기반 실행력 강화(팀 생산성 증대)
- **Current Strength (This Period)**
  - AD display에서 요구사항 문서화, Design Doc 템플릿 제작 및 확산을 통해 프로젝트 진행을 구조화함.
- **Growth Goal**
  - 프로젝트 초기에
    - 요구사항 명세(Functional/Non-functional)
    - 리스크/의존성 맵
    - 테스트/릴리즈 계획
    까지 포함한 “표준 실행 패키지”를 더 빠르게 만들 수 있어야 함.
- **Action Plan**
  - Design Doc 템플릿을 요구사항/테스트/릴리즈 섹션 포함으로 개선
  - 팀 내에서 문서 기반 진행을 자연스럽게 하기 위한 최소 규칙(예: PRD/Design 필수 항목)을 제안

### G-5. 리더십: 조율/우선순위/프로젝트 운영 능력 강화
- **Current Strength (This Period)**
  - AD display에서 팀원 업무 분배, 데일리 스크럼 운영, 타 팀과의 사전 합의 등 프로젝트 운영 역할을 수행함.
- **Growth Goal**
  - 앞으로는 프로젝트 규모가 커질수록
    - 우선순위 결정 기준
    - 의존성/리스크 관리
    - stakeholder 커뮤니케이션
    의 수준을 더 체계화해야 함.
- **Action Plan**
  - 이슈/리스크 레지스터 운영(주요 리스크, 소유자, 완화 계획)
  - 주간 단위로 milestone/metric 기반 진행 상황을 공유하는 습관화

---

## 3. Notes (선택)

### 앞으로 더 강조하면 좋은 메시지(자기평가 관점)
- “단순 구현”이 아니라
  - **제품/운영 관점(Non-reboot, Error 전파, Bring-up)**
  - **시스템 관점(RI/Universe/Robot 데이터 플로우)**
  - **품질/확장성 관점(제조사 SDK 의존성 제거, Driver 자체 구현)**
  - **프로젝트 운영 관점(스택 선정, 성능 검증, 문서 템플릿 구축, 리스크 선제 대응)**
  로 영향 범위를 넓혔다는 점을 강조할 수 있음.


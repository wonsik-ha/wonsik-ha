# 마케팅 파트너 정산 자동화 시스템 유지보수 가이드

## 📋 시스템 개요

본 시스템은 마이리얼트립의 마케팅 파트너 정산 업무를 자동화하는 Chrome Extension입니다. ERP 시스템의 결의서 데이터를 Dooray 기안 시스템으로 자동 전송하여 결재 프로세스를 효율화합니다.

### 주요 기능
- ERP 결의서 데이터 추출 및 검증
- 첨부파일 자동 변환 및 저장
- Dooray 기안 양식 자동 작성
- 결재선 자동 설정
- 실시간 진행 상태 모니터링

## 📁 디렉토리 구조

```
202507_marketing_partner_draft_automation/
├── codebase/
│   ├── manifest.json           # Chrome Extension 설정 파일
│   ├── package.json            # Node.js 프로젝트 설정
│   ├── package-lock.json       # 의존성 잠금 파일
│   ├── readme.md              # 프로젝트 설명서
│   │
│   ├── assets/                # 아이콘 리소스
│   │   ├── icon16.png
│   │   ├── icon32.png
│   │   ├── icon48.png
│   │   └── icon128.png
│   │
│   ├── config/                # 설정 파일
│   │   ├── babel.config.js    # Babel 트랜스파일러 설정
│   │   └── jest.config.js     # Jest 테스트 설정
│   │
│   ├── src/                   # 소스 코드
│   │   ├── background/        # Background Service Worker
│   │   │   ├── background.js  # 메인 백그라운드 스크립트
│   │   │   ├── main.js       # 백그라운드 진입점
│   │   │   │
│   │   │   ├── core/         # 핵심 모듈
│   │   │   │   ├── Application.js    # 애플리케이션 메인 클래스
│   │   │   │   └── EventEmitter.js   # 이벤트 처리 기반 클래스
│   │   │   │
│   │   │   ├── automation/   # 자동화 로직
│   │   │   │   └── DraftAutomation.js # 기안 자동화 메인 클래스
│   │   │   │
│   │   │   ├── managers/     # 매니저 클래스
│   │   │   │   ├── FileManager.js       # 파일 처리 매니저
│   │   │   │   ├── LogManager.js        # 로그 관리 매니저
│   │   │   │   ├── StorageManager.js    # 저장소 관리 매니저
│   │   │   │   └── ValidationManager.js # 유효성 검증 매니저
│   │   │   │
│   │   │   └── utils/        # 유틸리티 함수
│   │   │       ├── MessageUtils.js  # 메시지 통신 유틸
│   │   │       └── Utils.js        # 일반 유틸리티
│   │   │
│   │   ├── content/          # Content Scripts
│   │   │   ├── erp_content.js      # ERP 페이지 자동화
│   │   │   ├── dooray_content.js   # Dooray 페이지 자동화
│   │   │   │
│   │   │   ├── base/         # (현재 비어있음)
│   │   │   │
│   │   │   └── common/       # 공통 유틸리티
│   │   │       ├── DateUtils.js        # 날짜 처리 유틸
│   │   │       ├── FlowUtils.js        # 플로우 제어 유틸
│   │   │       ├── JsonMonitor.js      # JSON 모니터링
│   │   │       ├── MessageUtils.js     # 메시지 유틸
│   │   │       └── SelectorObserver.js # DOM 셀렉터 감시
│   │   │
│   │   └── popup/           # Extension Popup UI
│   │       ├── popup.html   # 팝업 HTML
│   │       └── popup.css    # 팝업 스타일
│   │
│   ├── tests/               # 테스트 파일
│   │   ├── README.md
│   │   ├── fixtures/        # 테스트 데이터
│   │   ├── util/           # 테스트 유틸리티
│   │   ├── test_drafters.html         # 기안자 테스트
│   │   ├── test_finalindex_validation.html # 인덱스 검증 테스트
│   │   └── test_team_names.html      # 팀명 테스트
│   │
│   └── node_modules/        # NPM 패키지 (자동 생성)
│
└── dooray-draft-automation.zip  # 배포용 압축 파일
```

### 디렉토리별 역할

- **`src/background/`**: Chrome Extension의 백그라운드 로직. Service Worker로 동작하며 모든 메시지 라우팅과 데이터 관리를 담당
- **`src/content/`**: 웹페이지에 주입되는 스크립트. ERP와 Dooray 페이지를 직접 조작
- **`src/popup/`**: Extension 아이콘 클릭 시 나타나는 UI. 사용자 입력과 실행 제어를 담당
- **`tests/`**: HTML 기반 테스트 파일들. 주요 기능을 독립적으로 테스트 가능

## 🏗️ 시스템 아키텍처

```mermaid
graph TB
    subgraph CE["Chrome Extension"]
        Popup["Popup UI<br/>popup.html/css"]
        BG["Background Service Worker<br/>background.js"]
        
        subgraph CS["Content Scripts"]
            ERP["ERP Content<br/>erp_content.js"]
            Dooray["Dooray Content<br/>dooray_content.js"]
        end
        
        subgraph CM["Core Modules"]
            App[Application.js]
            Draft[DraftAutomation.js]
            Event[EventEmitter.js]
        end
        
        subgraph MG["Managers"]
            Storage[StorageManager]
            File[FileManager]
            Log[LogManager]
            Validation[ValidationManager]
        end
        
        subgraph UT["Utils"]
            MsgUtil[MessageUtils]
            Utils[Utils]
            DateUtil[DateUtils]
            FlowUtil[FlowUtils]
        end
    end
    
    subgraph ES["External Systems"]
        ERPUI["ERP System<br/>erp.myrealtrip.net"]
        DoorayUI["Dooray System<br/>myrealtrip.dooray.com"]
    end
    
    subgraph ST["Storage"]
        IndexedDB[(IndexedDB)]
        ChromeStorage["Chrome Storage"]
    end
    
    Popup --> App
    App --> Draft
    Draft --> Storage
    Draft --> File
    Draft --> Log
    
    BG <--> ERP
    BG <--> Dooray
    BG <--> Storage
    
    ERP <--> ERPUI
    Dooray <--> DoorayUI
    
    Storage --> IndexedDB
    Storage --> ChromeStorage
```

## 📬 메시지 흐름

### 1. 자동화 실행 플로우

```mermaid
sequenceDiagram
    participant User
    participant Popup
    participant BG as Background
    participant ERP as ERP Content
    participant Dooray as Dooray Content
    participant Storage
    
    User->>Popup: 실행 버튼 클릭
    Popup->>Popup: 데이터 검증
    Popup->>Storage: executionData 저장
    Popup->>ERP: startAutomation
    
    ERP->>ERP: ERP 데이터 추출
    ERP->>BG: saveErpTabInfo
    ERP->>BG: getAllErpRowObjects
    BG->>Storage: erpTabId 저장
    
    ERP->>ERP: 새 탭 열기
    ERP->>Dooray: 페이지 로드
    
    Dooray->>BG: getExecutionData
    BG->>Storage: 데이터 조회
    Storage-->>Dooray: executionData
    
    Dooray->>Dooray: 기안 양식 작성
    Dooray->>BG: changeDraftTitle
    Dooray->>Dooray: 결재선 설정
    Dooray->>Dooray: 첨부파일 업로드
    
    Dooray->>BG: automationComplete
    BG->>Popup: 완료 알림
```

### 2. 데이터 검증 플로우

```mermaid
graph LR
    Start([시작]) --> Check1{기안일자<br/>입력?}
    Check1 -->|No| Error1[오류: 일자 필요]
    Check1 -->|Yes| Check2{첨부파일<br/>존재?}
    Check2 -->|No| Error2[오류: 파일 필요]
    Check2 -->|Yes| Check3{ERP 데이터<br/>확인}
    Check3 -->|No| Error3[오류: 데이터 없음]
    Check3 -->|Yes| Process[자동화 실행]
    
    Process --> Type{증빙 유형<br/>분류}
    Type -->|원화| Payment[원화 처리]
    Type -->|국세| IncomeTax[국세 처리]
    Type -->|지방세| LocalTax[지방세 처리]
    
    Payment --> Complete[완료]
    IncomeTax --> Complete
    LocalTax --> Complete
```

## 🔑 주요 컴포넌트 설명

### Background Service Worker (`background.js`)
- **역할**: 중앙 메시지 라우터 및 데이터 관리
- **주요 기능**:
  - Chrome Extension API 메시지 처리
  - IndexedDB 데이터 저장/조회
  - 탭 간 통신 중개
  - Script 실행 (executeScript)

### Content Scripts

#### ERP Content (`erp_content.js`)
- **역할**: ERP 페이지 자동화
- **주요 기능**:
  - Grid 데이터 추출 (`dewsControl._grid`)
  - 증빙 유형 분류 (원화/국세/지방세)
  - 새 탭 열기 트리거

#### Dooray Content (`dooray_content.js`)
- **역할**: Dooray 기안 양식 자동 작성
- **주요 기능**:
  - 제목 자동 생성 (월별, 유형별)
  - 결재선 자동 설정
  - 첨부파일 업로드

### Core Modules

#### DraftAutomation.js
- **역할**: 자동화 로직 총괄
- **주요 메서드**:
  - `executeAutomation()`: 자동화 실행
  - `getDraftersData()`: 기안자 정보 수집
  - `saveDraftersToIndexedDB()`: 기안자 정보 저장
  - `validateFormData()`: 폼 데이터 검증

### Manager Classes

#### StorageManager
- **저장소**: IndexedDB + Chrome Storage
- **데이터 타입**:
  - `executionData`: 실행 데이터
  - `erpTabId`: ERP 탭 ID
  - `indexInfo`: 진행 상태 정보
  - `draftFormData`: 폼 데이터

#### FileManager
- **역할**: 첨부파일 관리
- **기능**:
  - File → Blob 변환
  - Base64 인코딩
  - 파일 리스트 관리

## 🎯 중요 메시지 타입

### Background ↔ Content Scripts

| 메시지 | 방향 | 설명 |
|--------|------|------|
| `startAutomation` | Popup → ERP | 자동화 시작 |
| `saveExecutionData` | Popup → BG | 실행 데이터 저장 |
| `getExecutionData` | Dooray → BG | 실행 데이터 조회 |
| `saveErpTabInfo` | ERP → BG | ERP 탭 정보 저장 |
| `getAllErpRowObjects` | Popup → BG → ERP | ERP Grid 데이터 조회 |
| `changeDraftTitle` | Dooray → BG | 기안 제목 변경 |
| `changeApprovalLineOrder` | Dooray → BG | 결재선 순서 변경 |
| `automationComplete` | Dooray → BG → Popup | 자동화 완료 알림 |
| `automationError` | Any → BG → Popup | 오류 알림 |

## 🔧 유지보수 체크리스트

### 배포 전 확인사항
- [ ] `manifest.json` 버전 업데이트
- [ ] 권한 설정 확인 (`permissions`, `host_permissions`)
- [ ] Content Scripts 매칭 URL 확인
- [ ] 테스트 환경/운영 환경 URL 분기 처리

### 자주 발생하는 이슈

#### 1. Vue 인스턴스 접근 실패
```javascript
// Dooray는 Vue 3를 사용하므로 인스턴스 접근 방식 주의
const vue = element.__vue__ || element.__vueParentComponent;
```

#### 2. ERP Grid 데이터 접근
```javascript
// jQuery와 dewsControl 의존성
let grid = $('#grid').data('dewsControl')._grid;
let grid_gv = grid._gv;
```

#### 3. IndexedDB 업그레이드
```javascript
request.onupgradeneeded = (event) => {
    const db = event.target.result;
    if (!db.objectStoreNames.contains('drafters')) {
        const store = db.createObjectStore('drafters', { keyPath: 'id' });
        store.createIndex('name', 'name', { unique: false });
    }
};
```

### 디버깅 팁

1. **Chrome DevTools 활용**
   - Service Worker: `chrome://extensions` → 서비스 워커 검사
   - Content Scripts: 해당 페이지 개발자 도구
   - IndexedDB: Application 탭 → Storage → IndexedDB

2. **로그 확인 위치**
   - Popup 로그: Popup 창 우클릭 → 검사
   - Background 로그: Service Worker 콘솔
   - Content Script 로그: 해당 웹페이지 콘솔

3. **메시지 추적**
   ```javascript
   // 모든 메시지에 로그 추가
   console.log('Sending message:', message);
   console.log('Received response:', response);
   ```

## 📝 개선 제안사항

1. **에러 핸들링 강화**
   - 네트워크 타임아웃 처리
   - 재시도 로직 구현
   - 사용자 친화적 에러 메시지

2. **성능 최적화**
   - 대량 데이터 처리 시 청크 단위 처리
   - 불필요한 DOM 조작 최소화
   - 메모리 누수 방지

3. **테스트 자동화**
   - Jest 단위 테스트 추가
   - E2E 테스트 시나리오 작성
   - CI/CD 파이프라인 구축

## 📚 참고 자료

- [Chrome Extension Manifest V3 문서](https://developer.chrome.com/docs/extensions/mv3/)
- [IndexedDB API](https://developer.mozilla.org/ko/docs/Web/API/IndexedDB_API)
- [Chrome Extension Message Passing](https://developer.chrome.com/docs/extensions/mv3/messaging/)

---

*마지막 업데이트: 2025년 1월*

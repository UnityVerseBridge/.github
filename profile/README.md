# UnityVerseBridge

Quest VR 헤드셋과 모바일 기기 간의 실시간 WebRTC 통신을 구현한 Unity 기반 브릿지 시스템입니다.

## 🎯 프로젝트 개요

UnityVerseBridge는 Meta Quest VR 환경과 모바일 애플리케이션을 WebRTC 기술로 연결하여, VR 화면을 모바일로 스트리밍하고 모바일 터치 입력을 VR 공간에서 처리할 수 있게 하는 통합 솔루션입니다.

### 시스템 아키텍처

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│   Quest App     │     │ Signaling Server │     │   Mobile App    │
│  (비디오 송신)   │◄────┤   (WebSocket)    ├────►│  (비디오 수신)   │
│  (터치 수신)     │     │   (중계 서버)    │     │  (터치 송신)     │
└────────┬────────┘     └──────────────────┘     └────────┬────────┘
         │                                                │
         │              ┌──────────────────┐              │
         └──────────────┤      Core        ├──────────────┘
                        │ (WebRTC Package) │
                        └──────────────────┘
```

### 저장소 구조

- **[core](https://github.com/UnityVerseBridge/core)**: WebRTC 및 시그널링 공통 모듈 (Unity Package)
- **[quest-app](https://github.com/UnityVerseBridge/quest-app)**: Quest 디바이스용 Unity 앱
- **[mobile-app](https://github.com/UnityVerseBridge/mobile-app)**: 모바일 디바이스용 Unity 앱  
- **[signaling-server](https://github.com/UnityVerseBridge/signaling-server)**: WebSocket 기반 시그널링 서버

## 🚀 주요 기능

### 비디오 스트리밍
- Quest VR 카메라 뷰를 모바일로 실시간 전송
- RenderTexture 기반 고품질 스트리밍
- 적응형 비트레이트 및 해상도 조정

**작동 원리**: Quest의 VR 카메라가 렌더링하는 화면을 RenderTexture로 캡처합니다. 이 텍스처는 Unity WebRTC의 VideoStreamTrack으로 변환되어 H.264 코덱으로 인코딩됩니다. 네트워크 상태에 따라 비트레이트를 동적으로 조정하여 끊김 없는 스트리밍을 보장합니다.

### 터치 입력 제어
- 모바일 터치를 VR 3D 공간 좌표로 변환
- 정규화된 좌표 시스템 (0-1)
- VR UI 및 3D 오브젝트 상호작용

**변환 과정**: 
1. 모바일 화면의 터치 좌표를 화면 크기로 나누어 0-1 범위로 정규화
2. WebRTC DataChannel을 통해 JSON 형식으로 전송
3. Quest에서 정규화된 좌표를 카메라 viewport 좌표로 변환
4. Camera.ViewportPointToRay()로 3D 공간의 Ray 생성
5. Physics.Raycast()로 충돌 지점 계산

### 햅틱 피드백
- Quest → Mobile 진동 명령 전송
- 플랫폼별 햅틱 API 통합
- 실시간 피드백 동기화

**구현 방식**: Quest 컨트롤러의 입력 이벤트가 발생하면 햅틱 강도와 지속 시간을 DataChannel로 전송합니다. 모바일 앱은 플랫폼별 API(iOS: UIImpactFeedbackGenerator, Android: Vibrator)를 사용하여 진동을 생성합니다.

### 네트워크 통신
- WebRTC P2P 연결로 저지연 통신
- 룸 기반 자동 매칭
- 연결 상태 모니터링 및 자동 재연결

**P2P 연결 과정**:
1. 두 피어가 시그널링 서버의 같은 룸에 참가
2. Quest(Offerer)가 RTCPeerConnection 생성 및 Offer SDP 생성
3. 시그널링 서버를 통해 Mobile(Answerer)로 Offer 전달
4. Mobile이 Answer SDP 생성 및 전송
5. ICE candidate 교환으로 최적 경로 탐색
6. STUN 서버를 통한 공인 IP 획득
7. P2P 연결 수립 후 시그널링 서버 연결 불필요

## 💻 시스템 요구사항

### 개발 환경
- **Unity**: 6000.0.33f1 (Unity 6 LTS) 또는 2022.3.34f1 LTS
- **Unity WebRTC**: 3.0.0-pre.8 이상
- **Node.js**: 16.x 이상 (시그널링 서버)

### 하드웨어 요구사항

#### Quest 기기
- Meta Quest 2, Quest 3, Quest Pro
- 개발자 모드 활성화 필요

#### 모바일 기기
- **iOS**: iOS 12.0 이상
- **Android**: API Level 26 (Android 8.0) 이상

#### 서버
- WebSocket 지원
- 포트 설정 가능 (.env 파일)
- 공용 IP 또는 로컬 네트워크

## 📦 설치 및 설정

### 1. 프로젝트 클론

각 저장소를 개별적으로 클론합니다:

```bash
# Core 패키지
git clone https://github.com/UnityVerseBridge/core.git

# Quest 앱
git clone https://github.com/UnityVerseBridge/quest-app.git

# 모바일 앱
git clone https://github.com/UnityVerseBridge/mobile-app.git

# 시그널링 서버
git clone https://github.com/UnityVerseBridge/signaling-server.git
```

### 2. Core 패키지 설정

Unity Package Manager에서 로컬 패키지로 추가:
1. Window > Package Manager
2. `+` 버튼 > "Add package from disk..."
3. `core/package.json` 선택

또는 git URL로 추가:
```
https://github.com/UnityVerseBridge/core.git
```

### 3. 시그널링 서버 실행

```bash
cd signaling-server
npm install

# 환경 설정 (.env 파일 생성)
cp .env.example .env
# .env 파일을 편집하여 포트 설정
# PORT=8080 (원하는 포트로 변경 가능)

node server.js
```

### 시그널링 서버 설정

`.env` 파일 예시:
```env
# 서버 포트 (기본값: 8080)
PORT=8080

# 인증 모드 (선택사항)
REQUIRE_AUTH=false
AUTH_KEY=your-secret-key

# 토큰 만료 시간 (밀리초, 기본값: 24시간)
TOKEN_EXPIRY=86400000
```

서버가 시작되면 다음과 같은 메시지가 표시됩니다:
```
시그널링 서버가 포트 [설정한 포트]에서 실행 중입니다.
```

### 4. Quest 앱 설정

1. Unity에서 `quest-app/UnityProject` 열기
2. Build Settings > Android 플랫폼 선택
3. Player Settings 확인:
   - Minimum API Level: 29
   - Target API Level: 32+
4. `ConnectionConfig` 에셋 설정:
   ```
   Signaling Server URL: ws://YOUR_SERVER_IP:YOUR_PORT
   Room ID: default-room (또는 원하는 룸 이름)
   Client Type: Quest
   Auto Connect: true
   Connection Timeout: 30
   ```
   (YOUR_PORT는 .env 파일에서 설정한 포트)

### 5. 모바일 앱 설정

1. Unity에서 `mobile-app/UnityProject` 열기
2. Build Settings에서 iOS/Android 선택
3. `ConnectionConfig` 에셋 설정 (Quest와 동일):
   ```
   Signaling Server URL: ws://YOUR_SERVER_IP:YOUR_PORT
   Room ID: default-room (Quest와 동일해야 함)
   Client Type: Mobile
   Auto Connect: true
   ```
4. 빌드 및 배포

## 🎮 사용 방법

### 연결 순서

1. **시그널링 서버 시작**
   ```bash
   cd signaling-server
   npm start
   ```

2. **Quest 앱 실행** (Offerer)
   - Quest 헤드셋에서 앱 시작
   - "연결" 버튼 클릭
   - 모바일 앱 대기

3. **모바일 앱 실행** (Answerer)
   - 모바일 기기에서 앱 시작
   - "연결" 버튼 클릭
   - 자동으로 Quest와 연결

### 기능 테스트

#### 비디오 스트리밍 확인
- 모바일 화면에 Quest VR 뷰가 표시되는지 확인
- 프레임레이트와 화질 체크

#### 터치 입력 테스트
- 모바일 화면 터치 → Quest 공간에 포인터 표시
- UI 버튼 클릭 가능 여부 확인

#### 햅틱 피드백 테스트
- Quest 컨트롤러 버튼 → 모바일 진동 확인

## 🔧 문제 해결

### 연결 실패
- 두 기기가 같은 네트워크에 있는지 확인
- 시그널링 서버 URL이 올바른지 확인
- 방화벽이 설정한 포트를 차단하지 않는지 확인

### 비디오가 표시되지 않음
- Console에서 `VideoStreamTrack created successfully` 로그 확인
- RenderTexture가 Created 상태인지 확인
- WebRTC.Update() 코루틴이 실행 중인지 확인

### 터치가 작동하지 않음
- DataChannel이 열려있는지 확인
- 터치 좌표가 올바르게 전송되는지 로그 확인
- VR 공간의 Raycast 대상에 Collider가 있는지 확인

### 성능 문제
- RenderTexture 해상도를 720p로 낮춰보기
- 네트워크 대역폭 확인 (최소 5Mbps 권장)
- 불필요한 로그 출력 비활성화

## 📁 프로젝트 구조 상세

### Core 패키지
```
core/
├── Runtime/
│   ├── WebRtcManager.cs          # WebRTC 연결 관리
│   ├── SignalingClient.cs        # 시그널링 통신
│   ├── ConnectionConfig.cs       # 연결 설정
│   └── DataChannel/              # 데이터 채널 관련
└── Samples~/
    └── SimpleConnection/         # 기본 연결 예제
```

### Quest App
```
quest-app/UnityProject/Assets/
├── Scripts/
│   ├── QuestAppInitializer.cs    # 앱 초기화
│   ├── VrStreamSender.cs         # 비디오 송신
│   ├── VrTouchReceiver.cs        # 터치 수신
│   └── VrHapticRequester.cs      # 햅틱 요청
└── Scenes/
    └── SampleScene.unity         # 메인 씬
```

### Mobile App
```
mobile-app/UnityProject/Assets/
├── Scripts/
│   ├── MobileAppInitializer.cs   # 앱 초기화
│   ├── MobileVideoReceiver.cs    # 비디오 수신
│   ├── MobileInputSender.cs      # 터치 송신
│   └── MobileHapticReceiver.cs   # 햅틱 수신
└── Scenes/
    └── SampleScene.unity         # 메인 씬
```

## 🛠️ 개발 환경 설정

### Unity 설정
- Unity 6 LTS 권장
- Android Build Support (Quest용)
- iOS Build Support (iPhone용)

### 필수 패키지
- Unity WebRTC (com.unity.webrtc) 3.0.0-pre.8+
- Meta XR SDK (Quest용)
- TextMeshPro

### 추가 도구
- Meta Quest Developer Hub (Quest 개발)
- Android Studio (Android 빌드)
- Xcode (iOS 빌드)

## 🔐 보안 고려사항

### 인증 기능
시그널링 서버는 현재 간단한 토큰 기반 인증만 지원합니다. (JWT 미지원)

### 인증 모드로 서버 실행

   ```bash
   npm run start:auth
   ```

**현재 구현:**
- 간단한 토큰 생성 및 검증
- 24시간 만료 시간
- 메모리 기반 토큰 저장

**프로덕션 권장사항:**
- JWT(JSON Web Token) 라이브러리 사용
- 데이터베이스 기반 세션 관리
- OAuth 2.0 등 표준 인증 프로토콜 적용

### 프로덕션 설정
- WSS (WebSocket Secure) 사용
- HTTPS 인증서 설정
- 환경 변수로 민감한 정보 관리

## 📊 성능 최적화

### Quest 최적화 (계획)
- **Fixed Foveated Rendering**: VR에서는 눈이 보는 중심부만 선명하게 보이므로, 화면 가장자리는 낮은 품질로 렌더링하여 성능을 향상시킬 예정입니다.
- **프레임 레이트 최적화**: Quest의 기본 72Hz를 유지하면서 필요시 90Hz로 전환할 수 있도록 설정합니다.
- **드로우 콜 감소**: 같은 재질을 사용하는 오브젝트들을 묶어서 한 번에 그리는 기법을 적용할 예정입니다.

### 모바일 최적화 (일부 구현)
- **하드웨어 디코딩**: 비디오 디코딩을 CPU 대신 GPU에서 처리하여 배터리 소모를 줄입니다.
- **동적 해상도**: 네트워크 상태가 좋으면 고화질(1080p), 나쁘면 저화질(360p)로 자동 전환하는 기능을 구현 예정입니다.
- **백그라운드 처리**: 앱을 다른 앱으로 전환했을 때 불필요한 처리를 중단하여 배터리를 절약합니다.

### 네트워크 최적화 (기본 구현)

**현재 구현된 기능**:
- **P2P 직접 연결**: 시그널링 서버는 처음 연결할 때만 사용하고, 이후에는 두 기기가 직접 통신하여 지연시간을 최소화합니다.
- **STUN 서버 사용**: Google의 무료 STUN 서버를 통해 공인 IP를 찾아 연결합니다.

**계획 중인 최적화**:
- **스마트 품질 조절**: 인터넷이 느려지면 자동으로 화질을 낮추고, 빨라지면 다시 높이는 기능
- **패킷 손실 대응**: 데이터가 중간에 사라져도 중요한 정보는 다시 보내는 기능
- **연결 복구**: 와이파이가 끊겼다가 다시 연결되어도 자동으로 재연결되는 기능

**TURN 서버 (필요시 추가)**:
회사 방화벽이나 특수한 네트워크 환경에서는 TURN 서버가 필요할 수 있습니다. 이는 두 기기 사이의 중계 역할을 하는 서버로, 직접 연결이 불가능할 때 사용됩니다.

## 🚧 향후 개발 계획

### 우선순위 높음
- **오디오 스트리밍 추가**
  - VR 환경의 3D 공간 음향을 모바일로 전송
  - 마이크 입력을 통한 양방향 음성 통신
  - WebRTC의 오디오 트랙 활용

- **AR 모드 지원**
  - Quest 3의 컬러 패스스루 기능 활용
  - 현실 공간에 가상 오브젝트 배치 및 공유
  - 모바일에서 AR 오브젝트 제어

### 중간 우선순위
- **`VrStreamSender`의 RemoveTrack 기능 구현**
  - 현재는 비디오 트랙을 추가만 할 수 있고 제거할 수 없습니다
  - 구현 시 장점:
    - 스트리밍 일시정지/재개 기능으로 네트워크 대역폭 절약
    - 여러 카메라 뷰 전환 시 기존 트랙을 제거하고 새 트랙 추가 가능
    - 메모리 누수 방지 및 리소스 효율적 관리
  
- **1:N 연결 지원**
  - 하나의 Quest가 여러 모바일 기기에 동시 스트리밍
  - 용도: VR 수업, 프레젠테이션, 관전 모드
  - 각 모바일 기기별 독립적인 터치 입력 처리

### 장기 계획
- **클라우드 기반 시그널링 서버**
  - AWS/Azure 배포 가이드 제공
  - 자동 확장 및 로드 밸런싱

- **Unity 6 LTS 최적화**
  - 현재 Unity 6 LTS로 개발 중이며, 새로운 기능 활용 예정
  - 향상된 렌더링 파이프라인 적용
  - WebGPU 지원 대비

- **분석 및 모니터링**
  - 연결 품질 통계 대시보드
  - 사용자 행동 분석 도구

## 🤝 기여 방법

1. 이 저장소를 Fork
2. 새 기능 브랜치 생성 (`git checkout -b feature/AmazingFeature`)
3. 변경사항 커밋 (`git commit -m 'Add some AmazingFeature'`)
4. 브랜치에 Push (`git push origin feature/AmazingFeature`)
5. Pull Request 생성

## 📄 라이선스

이 프로젝트는 BSD 3-Clause 라이선스를 따릅니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참고하세요.

## 👥 제작자

- **kugorang** - *Initial work* - [GitHub](https://github.com/kugorang)

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

node server.js
```

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
│   ├── VrStreamSender.cs         # 비디오 송신
│   ├── VrTouchReceiver.cs        # 터치 수신
│   └── VrHapticRequester.cs      # 햅틱 요청
└── Scenes/
    └── QuestStreamingDemo.unity  # 메인 씬
```

### Mobile App
```
mobile-app/UnityProject/Assets/
├── Scripts/
│   ├── MobileVideoReceiver.cs    # 비디오 수신
│   ├── MobileInputSender.cs      # 터치 송신
│   └── MobileHapticReceiver.cs   # 햅틱 수신
└── Scenes/
    └── MobileStreamingDemo.unity # 메인 씬
```

## 🔐 보안 고려사항

### 인증 기능
시그널링 서버는 간단한 토큰 기반 인증을 지원합니다.

**현재 구현:**
- 간단한 토큰 생성 및 검증
- 24시간 만료 시간
- 메모리 기반 토큰 저장

**프로덕션 권장사항:**
- JWT(JSON Web Token) 사용
- 데이터베이스 기반 세션 관리
- WSS (WebSocket Secure) 사용
- HTTPS 인증서 설정

## 📊 성능 최적화

### Quest 최적화
- **프레임 레이트**: 72Hz 기본, 90Hz 옵션
- **렌더링**: Fixed Foveated Rendering 지원
- **스트리밍 해상도**: 640x360 (기본), 1280x720, 1920x1080

### 모바일 최적화
- **하드웨어 디코딩**: GPU 가속 활용
- **동적 해상도**: 네트워크 상태 기반 자동 조정
- **백그라운드 처리**: 앱 전환 시 리소스 절약

### 네트워크 최적화
- **P2P 직접 연결**: 최소 지연시간
- **STUN 서버**: Google의 무료 STUN 서버 사용
- **적응형 비트레이트**: 네트워크 상황에 따른 품질 조정

## 🚧 향후 개발 계획

### 우선순위 높음
- **오디오 스트리밍 추가**
- **AR 모드 지원** (Quest 3 패스스루)
- **VrStreamSender RemoveTrack 구현**

### 중간 우선순위
- **1:N 연결 지원**
- **클라우드 시그널링 서버**
- **Unity 6 최적화**

### 장기 계획
- **WebXR 표준 통합**
- **5G 네트워크 최적화**
- **분석 및 모니터링 도구**

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

---

<p align="center">
  <b>UnityVerseBridge</b><br>
  <i>Bridging VR and Mobile Worlds with WebRTC</i><br>
  <br>
  <a href="https://github.com/UnityVerseBridge">GitHub Organization</a> • 
  <a href="https://github.com/UnityVerseBridge/UnityVerseBridge/wiki">Wiki</a> • 
  <a href="https://github.com/UnityVerseBridge/UnityVerseBridge/issues">Issues</a>
</p>
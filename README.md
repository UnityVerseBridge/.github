# UnityVerseBridge - Quest와 모바일 앱 간 WebRTC 스트리밍

Meta Quest VR 헤드셋과 모바일 기기 간의 실시간 WebRTC 통신을 구현한 Unity 기반 브릿지 시스템입니다.

## 🎯 프로젝트 개요

UnityVerseBridge는 VR 환경을 모바일로 실시간 스트리밍하고, 모바일 터치 입력을 VR 공간에서 처리할 수 있는 양방향 통신 시스템입니다.

## 📋 주요 기능

- **비디오 스트리밍**: Quest에서 캡처한 가상 환경을 모바일 장치로 실시간 스트리밍
- **터치 입력 전송**: 모바일 앱에서 터치 입력을 Quest 앱으로 전송
- **햅틱 피드백**: Quest 앱에서 모바일 앱으로 햅틱 명령 전송
- **P2P 통신**: WebRTC 기반 저지연 연결 (50ms 이하)

## 🏗️ 시스템 구성

- **[@core](core/)**: WebRTC 및 시그널링 공통 모듈
- **[@quest-app](quest-app/)**: Quest 디바이스용 앱 (스트리밍 송신, 터치 수신, 햅틱 요청)
- **[@mobile-app](mobile-app/)**: 모바일 디바이스용 앱 (스트리밍 수신, 터치 송신, 햅틱 수신)
- **[@signaling-server](signaling-server/)**: WebSocket 기반 시그널링 서버

## 🚀 빠른 시작

### 1. 시그널링 서버 실행

```bash
cd signaling-server
npm install
node server.js
```

서버가 성공적으로 시작되면 `시그널링 서버가 포트 8080에서 실행 중입니다.` 메시지가 표시됩니다.

### 2. Quest 앱 설정 및 실행

1. Unity 에디터에서 `quest-app/UnityProject` 프로젝트를 엽니다.
2. `Scenes` 폴더에서 테스트 씬을 엽니다.
3. 시그널링 서버 URL이 올바르게 설정되어 있는지 확인합니다 (ex: `ws://192.168.0.100:8080`).
4. 테스트 씬을 실행하고 UI에서 "연결" 버튼을 클릭합니다.

### 3. 모바일 앱 설정 및 실행

1. Unity 에디터에서 `mobile-app/UnityProject` 프로젝트를 엽니다.
2. `Scenes` 폴더에서 테스트 씬을 엽니다.
3. 시그널링 서버 URL이 Quest 앱과 동일하게 설정되어 있는지 확인합니다.
4. 테스트 씬을 실행하고 UI에서 "연결" 버튼을 클릭합니다.

### 4. 연결 및 테스트

1. 두 앱이 모두 시그널링 서버에 연결된 후 WebRTC 연결이 자동으로 설정됩니다.
2. 연결 성공 후:
   - 모바일 앱에서 Quest의 비디오 스트림이 표시되는지 확인합니다.
   - 모바일 앱에서 화면을 터치하여 Quest 앱에서 해당 위치에 표시되는지 확인합니다.
   - Quest 앱에서 컨트롤러 버튼을 클릭하여 모바일 앱에서 진동이 발생하는지 확인합니다.

## 💻 요구사항

- Unity 6 LTS (6000.0.33f1) 또는 Unity 2022.3 LTS
- Unity WebRTC Package 3.0.0-pre.8
- Meta XR SDK 77.0.0
- Meta Quest 2/3/Pro
- iOS 12.0+ / Android API 26+
- Node.js 16+

## 🔧 문제 해결

- **연결 오류 발생 시**: 시그널링 서버 URL이 올바른지, 두 장치가 같은 네트워크에 연결되어 있는지 확인합니다.
- **비디오 스트림이 표시되지 않는 경우**: Quest 앱의 `VrStreamSender`에 RenderTexture가 올바르게 연결되어 있는지 확인합니다.
- **터치 입력이 전달되지 않는 경우**: 데이터 채널이 열려 있는지 확인하고, 로그를 통해 메시지가 전송되는지 확인합니다.
- **컴파일 오류 발생 시**: WebRTC 패키지와 NativeWebSocket 패키지가 올바르게 설치되어 있는지, 어셈블리 정의 파일의 참조가 올바른지 확인합니다.

## 📚 문서

상세한 기술 문서와 API 가이드는 [프로젝트 프로필](profile/README.md)을 참조하세요.

## 🤝 기여

기여를 환영합니다! 다음 단계를 따라주세요:

1. 프로젝트를 Fork 합니다
2. 기능 브랜치를 생성합니다 (`git checkout -b feature/AmazingFeature`)
3. 변경사항을 커밋합니다 (`git commit -m 'Add some AmazingFeature'`)
4. 브랜치에 Push 합니다 (`git push origin feature/AmazingFeature`)
5. Pull Request를 생성합니다

## 📄 라이선스

이 프로젝트는 BSD 3-Clause 라이선스를 따릅니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참고하세요.

## 📞 문의

- 이슈: [GitHub Issues](https://github.com/UnityVerseBridge/UnityVerseBridge/issues)
- 이메일: ialskdji@gmail.com

---

**UnityVerseBridge** - Bridging VR and Mobile Worlds
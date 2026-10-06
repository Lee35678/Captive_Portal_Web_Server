# Captive_Portal_Web_Server

ESP32를 AP로 열고 DNS 요청을 모두 자기 자신으로 돌려, 접속한 기기에 릴레이 제어 버튼 페이지를 띄우는 캡티브 포털 예제.

## 개요

ESP32가 비밀번호 없는 WiFi AP를 만들고, `DNSServer`가 모든 도메인 질의(`*`)에 ESP32의 AP IP로 응답한다. 그래서 AP에 접속한 스마트폰·PC가 어떤 주소를 열어도 같은 HTML 페이지가 나타나고, 페이지의 `Relay` 버튼을 누르면 릴레이 출력이 토글된다.

## 하드웨어

- 보드: ESP32 DOIT DevKit V1 (`esp32doit-devkit-v1`)
- 액추에이터: 릴레이 모듈

| 장치 | 신호 | ESP32 핀 |
|------|------|----------|
| 릴레이 | 제어 입력 | GPIO 15 (`RELAY`) |

## 동작 방식

- `WiFi.mode(WIFI_AP)` + `WiFi.softAPConfig(apIP, apIP, 255.255.255.0)`로 AP 주소 지정, `WiFi.softAP()`로 개방형 AP 시작
- `dnsServer.start(DNS_PORT, "*", apIP)` : 포트 53에서 모든 DNS 질의를 `apIP`로 응답
- 웹 서버(포트 80) 라우팅

| 경로 | 동작 |
|------|------|
| `/button` | `digitalWrite(RELAY, !digitalRead(RELAY))`로 릴레이 상태 반전, `OK` 응답, 시리얼에 `button pressed` 출력 |
| 그 외 모든 경로 | `responseHTML`(Captive Sample Server App 페이지)을 200으로 응답 |

- 페이지의 버튼은 `XMLHttpRequest`로 `/button`에 GET 요청을 보낸다
- `loop()`에서 `dnsServer.processNextRequest()`와 `webServer.handleClient()`를 반복 호출

## 개발 환경

- PlatformIO, platform `espressif32`, framework `arduino`
- 내장 라이브러리: `WiFi.h`, `DNSServer.h`, `WebServer.h`
- 업로드 속도 460800, 시리얼 모니터 속도 115200

## 설정

- `apIP` : AP 주소. 바꿀 경우 `responseHTML` 안 스크립트의 `/button` 요청 주소도 같이 바꿔야 한다
- AP 이름 : `setup()`의 `WiFi.softAP()` 인자
- `RELAY` : 릴레이 제어 핀

## 빌드 및 실행

```bash
pio run -t upload
pio device monitor -b 115200
```

시리얼에 `Captive Portal Started`가 출력되면 스마트폰 등으로 ESP32 AP에 접속한다.

## 폴더 구조

```
Captive_Portal_Web_Server/
├── platformio.ini   # 보드·프레임워크·속도 설정
└── src/
    └── main.cpp     # softAP + DNS 리다이렉트 + 릴레이 토글 웹 서버
```

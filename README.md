# 메타 레이밴 디스플레이 도슨트 테스트 앱

박물관 도슨트 시스템 프로토타입의 1차 테스트용 웹앱입니다.
BLE 비콘(이름 `01`~`08`) 근처에 가면 해당 작품 안내 텍스트가 텔레프롬프터 방식으로
자동 스크롤되고, TTS로 읽어줍니다.

- 단일 파일 `index.html` (빌드 도구 불필요)
- 600x600 정사각형 뷰포트 기준 디자인, 검정 배경 / 흰 텍스트
- 가장자리 안전영역 확보, 가로 스크롤 없음

## 사용 흐름
1. 페이지 진입 → **BLE 스캔 시작** 버튼만 표시
2. 버튼 클릭 → 스캔 시작, "스캔 중..."
3. 이름 `01`~`08` 디바이스 중 RSSI가 가장 강한 것을 현재 위치로 판단
   (노이즈 방지를 위해 약 2.5초간 최강 신호 유지 시 확정)
4. 위치가 바뀌면 화면 내용 교체 + 스크롤 처음부터 재시작

## Web Bluetooth 실험적 플래그 켜는 법
이 앱은 지속 스캔을 위해 실험적 API `navigator.bluetooth.requestLEScan()`을 사용합니다.
Chrome 계열 브라우저에서만 동작하며, 아래 플래그를 켜야 합니다.

1. 주소창에 `chrome://flags` 입력
2. **Experimental Web Platform features** 검색 → **Enabled**
3. 브라우저 재시작
4. (Android의 경우) 위치 권한 / 블루투스 권한 허용

> Web Bluetooth는 secure context가 필수이므로 반드시 **HTTPS**(또는 `localhost`)로 서빙해야 합니다.

## 테스트 / 디버그 모드
실제 비콘 8개가 없어도 테스트할 수 있습니다.

- **키보드 `1`~`8`**: 해당 번호 비콘에 진입한 것처럼 시뮬레이션
- 우측 상단 **디버그 ▾** 버튼: 감지된 디바이스 목록과 RSSI 값 패널 토글
- 디버그 패널에서 **TTS 켜기/끄기** 토글 가능
  - TTS On: 한국어(ko-KR) 음성으로 헤더+본문을 읽음
  - TTS Off: 비콘 진입 시 짧은 비프음 1회 재생

## HTTPS로 서빙하기

> Web Bluetooth는 secure context가 필수입니다. 글래스는 `localhost`가 아니라
> 네트워크 IP로 접속하므로 **반드시 HTTPS URL이 필요**합니다.
> (PC 크롬에서 키보드 테스트만 할 때는 `localhost`도 secure context로 인정됩니다.)

### 방법 A — GitHub Pages (배포됨, 추천)
이 저장소는 GitHub Pages로 호스팅됩니다. github.io는 자동으로 HTTPS를 제공하므로
글래스에서 바로 접속할 수 있습니다.

```
https://limjongyoon.github.io/metadisplaytest/
```

처음 배포하거나 다른 계정에 올릴 때:
```bash
git init -b main
git add .
git commit -m "도슨트 테스트 앱"
gh repo create metadisplaytest --public --source=. --remote=origin --push
# Pages 활성화 (main 브랜치 루트)
gh api -X POST repos/<USER>/metadisplaytest/pages \
  -f "source[branch]=main" -f "source[path]=/"
# 발급된 URL 확인 (빌드까지 1~2분)
gh api repos/<USER>/metadisplaytest/pages --jq .html_url
```
이후 코드를 바꾸면 `git push`만 하면 자동 재배포됩니다.

### 방법 B — ngrok 터널 (PC 빠른 테스트용 대안)
배포 없이 PC에서 임시 HTTPS URL을 띄울 때:
```bash
python3 -m http.server 8000          # 정적 서버
ngrok http 8000                      # 다른 터미널에서 HTTPS 터널
```
출력된 `https://xxxx.ngrok-free.app`을 글래스 브라우저에서 엽니다. (PC가 켜져 있어야 함)

## 데이터 / 커스터마이징
- 비콘 목록은 `index.html` 상단 `beacons` 배열에서 자동 생성됩니다(`01`~`08`).
- 본문은 `bodyLine`을 `BODY_REPEAT`(기본 50)회 반복해 긴 스크롤 콘텐츠를 자동 생성합니다.
- 스크롤 속도: `SCROLL_SPEED`(px/초), 신호 확정 시간: `CONFIRM_MS`,
  미감지 제외 시간: `STALE_MS` 상수로 조정합니다.

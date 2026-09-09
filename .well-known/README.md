# .well-known — 앱 링크 검증 파일

이 두 파일이 있어야 `https://jasper8350.github.io/brewmate/r/#…` 같은 공유 링크를
브라우저 대신 **앱이 바로** 열 수 있다(Android App Links / iOS Universal Links).

지금은 **자리표시자**가 들어 있어 동작하지 않는다. 그래서 앱 쪽 설정도 꺼져 있고,
링크를 열면 웹 페이지가 뜬 뒤 "BrewMate 앱에서 열기" 단추로 넘어간다. 그것도 정상 동작이다.

`.nojekyll` 이 저장소 루트에 있어야 GitHub Pages 가 `.` 로 시작하는 이 폴더를 서빙한다.
지우지 말 것.

## 채우는 순서

### 1. Android — `assetlinks.json`

`package_name` 은 확정한 패키지명. 지문은 **Play 앱 서명 키**의 SHA-256 이다
(업로드 키가 아니다 — Play 가 다시 서명하기 때문).

Play Console > 테스트 및 출시 > **앱 서명** > "앱 서명 키 인증서" 의 SHA-256 을 복사한다.
첫 업로드 전이라면 업로드 키의 지문으로 먼저 넣고, 업로드 후 앱 서명 키 지문을 **추가**한다
(배열이라 둘 다 넣어도 된다).

업로드 키 지문 보는 법:

```bash
keytool -list -v -keystore apps/mobile/release.keystore -alias brewmate | grep SHA256
```

### 2. iOS — `apple-app-site-association`

`appIDs` 는 `<Team ID>.<Bundle ID>` 형식이다.
Team ID 는 developer.apple.com 계정 화면 오른쪽 위 10자.

개발자 포털에서 그 App ID 에 **Associated Domains** 능력을 켜야 한다
(Identifiers > App IDs > 해당 ID > Capabilities).

> 확장자가 없고 `Content-Type: application/json` 으로 서빙돼야 한다.
> GitHub Pages 는 확장자 없는 파일을 `application/octet-stream` 으로 주는 경우가 있는데,
> iOS 14+ 는 Apple CDN 을 거쳐 받으므로 대체로 문제되지 않는다. 안 되면 커스텀 도메인을 쓴다.

### 3. 앱 설정 켜기

두 파일을 채워 배포한 뒤, 앱을 빌드할 때 환경변수를 준다.

```bash
SHARE_LINK_HOST=jasper8350.github.io npm run build:aab
```

Codemagic 이라면 환경변수 그룹 `brewmate_public` 에 `SHARE_LINK_HOST` 를 추가한다.
이 값이 있을 때만 `app.config.ts` 가 intent filter 와 associatedDomains 를 넣는다.

### 4. 확인

```bash
curl -s https://jasper8350.github.io/.well-known/assetlinks.json | head
# Android: 설치 후
adb shell pm get-app-links <패키지명>        # verified 로 나와야 한다
adb shell am start -a android.intent.action.VIEW -d "https://jasper8350.github.io/brewmate/r/#<코드>"
```

Google 의 검사 도구:
<https://developers.google.com/digital-asset-links/tools/generator>

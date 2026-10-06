# SDK-스텁-서명-배포

> 대분류: `09-Apple-플랫폼` / SDK 경량화와 배포 인증. TBD·버전검증·공증

### tapi
**개요:** Text-based API 파일 도구 모음의 입구이다. dylib 공개 API 목록을 관리한다.
**작동 방식:** 바이너리에서 공개 심볼만 뽑아 `.tbd` 텍스트 파일로 만든다. SDK 안에 가벼운 스텁만 넣어 용량을 줄인다. 링크 단계에서 실제 dylib 대신 이 파일을 참조한다.
```bash
tapi --help
```

### tapi-analyze
**개요:** 헤더에서 API 사용을 분석하는 TAPI 도우미이다.
**작동 방식:** C와 Objective-C 헤더를 읽고 어떤 심볼이 공개되어야 할지 추론한다. `.tbd` 생성 전 단계에서 검증한다.
```bash
tapi-analyze --help
```

### vtool
**개요:** Mach-O와 `.tbd`를 다루는 검증 도구이다.
**작동 방식:** dylib의 버전, 플랫폼, export를 보여주거나 변환한다. TAPI가 만든 스텁이 진짜 바이너리와 맞는지 대조한다.
```bash
vtool -show /usr/lib/libSystem.dylib 2>&1 | head -n 30
```

### notarytool
**개요:** Apple 공증을 요청하고 조회하는 공식 도구다.
**작동 방식:** Apple ID로 서버에 연결한다. 제출한 zip, dmg, pkg를 검사 대기열에 넣는다. 진행 상태를 폴링으로 확인하고 결과를 받는다. 통과 후에는 stapler로 티켓을 붙인다.
```bash
xcrun notarytool submit MyApp.zip --apple-id "me@example.com" --team-id ABCDE12345 --password "@keychain:NOTARY" --wait
```

### stapler
**개요:** 공증 티켓을 앱이나 dmg에 붙이는 도구다.
**작동 방식:** Apple 서버에서 받은 공증 티켓을 다운로드한다. 앱 번들, pkg, dmg 안에 티켓을 내장한다. 배포 직전 notarytool 통과 뒤에 실행한다.
```bash
xcrun stapler staple MyApp.dmg
```


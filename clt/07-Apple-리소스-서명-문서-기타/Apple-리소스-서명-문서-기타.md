# 7. Apple 리소스·서명·문서·기타

> 원문: `../CLT-tools-guide.md`에서 분리


Apple 전용 도구 21개. 리소스 포크, 파일 속성, 공증, 문서 생성처럼 macOS에서만 쓰는 작업을 맡는다.

### DeRez
**개요:** 컴파일된 리소스 파일에서 리소스 정의를 텍스트 형태로 뽑아내는 도구다.
**작동 방식:** 입력 파일의 리소스 포크와 데이터 포크를 읽고 리소스 맵을 해석한다. 타입, ID, 이름, 속성을 Rez가 다시 컴파일할 수 있는 문장으로 바꾼다. 출력은 표준 출력으로 나간다.
```bash
DeRez -only 'STR#' Carbon.rsrc > CarbonStrings.r
```

### GetFileInfo
**개요:** 파일의 Finder 정보와 타입, 크리에이터 코드, 생성일 등을 보여주는 조회 도구다.
**작동 방식:** 지정한 파일의 HFS 메타데이터를 읽는다. 타입 코드와 크리에이터 코드를 함께 표시한다. 잠금 여부, 보이지 않음 플래그도 보여준다. 읽기만 한다.
```bash
GetFileInfo ~/Documents/보고서.rtf
```

### ResMerger
**개요:** 리소스 포크 내용을 애플리케이션 패키지나 다른 파일에 합치는 도구다.
**작동 방식:** 소스 파일의 리소스를 읽어 대상 파일에 추가하거나 덮어쓴다. 리소스 맵 구조를 유지하면서 병합한다.
```bash
ResMerger -dst MyApp.app/Contents/Resources/MyApp.rsrc Source.rsrc
```

### Rez
**개요:** 텍스트로 쓴 리소스 정의를 컴파일해서 리소스 파일로 만드는 도구다. DeRez와 짝을 이룬다.
**작동 방식:** `.r` 파일과 헤더를 읽고 리소스 컴파일을 수행한다. 문자열, 메뉴, 아이콘 정의를 바이너리 리소스 형태로 바꾼다.
```bash
Rez -o MyStrings.rsrc MyStrings.r
```

### SetFile
**개요:** 파일의 타입, 크리에이터, Finder 플래그를 바꾸는 도구다. GetFileInfo와 짝을 이룬다.
**작동 방식:** 지정한 속성을 파일 메타데이터에 직접 쓴다. 타입 코드 변경, 보이지 않음 설정, 잠금 설정이 된다.
```bash
SetFile -t TEXT -c ttxt ~/Documents/메모.txt
```

### SplitForks
**개요:** 리소스 포크와 데이터 포크가 합쳐진 파일을 분리하는 도구다.
**작동 방식:** 입력 파일의 두 포크를 각각 별도 파일로 나눈다. AppleSingle이나 MacBinary 같은 보관 형태로 저장한다.
```bash
SplitForks OldApp.bin -outDir ~/Desktop/split
```

### cache-build-session
**개요:** Xcode 빌드 세션 캐시를 만들고 관리하는 내부 도구다.
**작동 방식:** 현재 SDK, 아키텍처, 빌드 설정을 읽어 세션 캐시 파일을 만든다. 같은 설정을 반복 빌드하면 캐시를 재사용한다.
```bash
cache-build-session -o /tmp/MyBuildSession.cache -sdk macosx
```

### ctags
**개요:** 소스 코드의 함수, 클래스, 매크로 위치를 모아 태그 파일로 만드는 도구다.
**작동 방식:** 지정한 소스 파일을 훑으며 정의 위치를 기록한다. 결과는 기본적으로 `tags` 파일에 쓴다. 에디터에서 정의로 바로 이동한다.
```bash
ctags -R Sources --languages=C,ObjectiveC
```

### gatherheaderdoc
**개요:** HeaderDoc 주석이 든 헤더들을 모아서 문서 세트로 묶는 스크립트다.
**작동 방식:** 지정한 디렉터리를 재귀적으로 돌며 헤더 파일을 찾는다. 각 파일을 headerdoc2html에 넘겨 HTML 조각을 만든다. 목차와 인덱스를 함께 생성한다.
```bash
gatherheaderdoc Sources/include docs/Headers -o docs/API
```

### hdxml2manxml
**개요:** HeaderDoc 중간 XML을 man 페이지용 XML로 바꾸는 변환 도구다.
**작동 방식:** 중간 XML을 입력으로 받는다. man 섹션, 이름, 설명 구조에 맞게 태그를 바꾼다. 출력은 xml2man이 처리할 수 있는 형태다.
```bash
hdxml2manxml MyDoc.hdx.xml > MyDoc.man.xml
```

### headerdoc2html
**개요:** 헤더 파일의 HeaderDoc 주석을 HTML 문서로 바꾸는 도구다.
**작동 방식:** 헤더의 특수 주석 표시를 파싱한다. 함수, 클래스, 매개변수 설명을 HTML 페이지로 렌더링한다.
```bash
headerdoc2html -o docs/API Sources/include/MyLib.h
```

### indent
**개요:** C와 Objective-C 소스의 들여쓰기를 자동 정리하는 도구다.
**작동 방식:** 소스 파일을 읽고 중괄호 깊이에 맞춰 공백을 다시 배치한다. 원본을 직접 고치므로 백업 후 실행이 좋다.
```bash
indent -kr -i4 Sources/legacy.c -o Sources/legacy.clean.c
```

### notarytool
**개요:** Apple 공증을 요청하고 조회하는 공식 도구다.
**작동 방식:** Apple ID로 서버에 연결한다. 제출한 zip, dmg, pkg를 검사 대기열에 넣는다. 진행 상태를 폴링으로 확인하고 결과를 받는다. 통과 후에는 stapler로 티켓을 붙인다.
```bash
xcrun notarytool submit MyApp.zip --apple-id "me@example.com" --team-id ABCDE12345 --password "@keychain:NOTARY" --wait
```

### pagestuff
**개요:** Mach-O 실행 파일의 논리 페이지 구성을 덤프하는 도구다.
**작동 방식:** 바이너리 헤더와 로드 명령을 읽는다. 각 페이지가 어떤 세그먼트에 속하는지 표로 보여준다.
```bash
pagestuff /bin/ls -p
```

### readtapi
**개요:** `.tapi`와 `.tbd` 파일을 읽어 내용을 보여주는 도구다.
**작동 방식:** TAPI 문서를 파싱해 아키텍처 목록을 뽑는다. 익스포트된 심볼과 허용된 클라이언트를 표시한다.
```bash
readtapi /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libSystem.tbd
```

### resolveLinks
**개요:** 경로 안의 심볼릭링크를 실제 경로로 풀어주는 유틸리티다. 빌드 스크립트에서 진짜 위치를 찾을 때 쓴다.
**작동 방식:** 입력 경로를 앞에서부터 한 조각씩 검사하면서 링크 여부를 확인한다. 링크를 만나면 그 자리에서 가리키는 대상을 치환한 뒤 다시 처음에 가까운 위치부터 검사를 이어간다. 이 과정을 링크가 하나도 남지 않을 때까지 반복한다. 절대 경로와 상대 경로를 구분해서 계산하므로 현재 디렉터리 위치가 결과에 영향을 준다. 출력은 표준 출력 한 줄이며 원본 파일은 바꾸지 않는다.

#### 동작 순서
1. 입력 경로를 받아 절대 경로 형태로 정규화한다. 상대 경로면 현재 작업 디렉터리를 앞에 붙인다.
2. 루트(`/`)부터 마지막 요소까지 왼쪽에서 오른쪽으로 한 단계씩 이동한다.
3. 각 단계에서 링크인지 확인한다. 일반 디렉터리면 다음 단계로 넘어간다.
4. 링크를 찾으면 대상 문자열을 읽는다. 대상이 상대 경로면 링크가 있던 디렉터리 기준으로 합치고, 절대 경로면 그대로 쓴다.
5. 남은 뒷부분 경로를 합친 새 경로를 만들고 2단계로 돌아가 재검사한다. 최대 반복 횟수를 넘기면 순환 링크로 보고 멈춘다.
6. 모든 조각에 링크가 없으면 정리한 최종 절대 경로를 출력한다.

```bash
resolveLinks /usr/local/bin/gcc
ls -l /usr/local/bin/gcc
resolveLinks ./상대경로/링크
```

### size
**개요:** Mach-O 바이너리의 세그먼트와 섹션 크기를 보여주는 도구다.
**작동 방식:** 오브젝트 파일 헤더를 읽고 섹션별 바이트 수를 합산한다. 영역을 구분해 표시한다.
```bash
size -m -x MyApp.app/Contents/MacOS/MyApp
```

### size-classic
**개요:** 예전 형식 그대로 크기를 보여주는 `size` 호환 도구다.
**작동 방식:** 기본 `size`와 같은 파서를 쓴다. 다만 출력 열 순서와 표시를 고전 BSD 형식에 맞춘다.
```bash
size-classic MyApp.o libHelper.a
```

### stapler
**개요:** 공증 티켓을 앱이나 dmg에 붙이는 도구다.
**작동 방식:** Apple 서버에서 받은 공증 티켓을 다운로드한다. 앱 번들, pkg, dmg 안에 티켓을 내장한다. 배포 직전 notarytool 통과 뒤에 실행한다.
```bash
xcrun stapler staple MyApp.dmg
```

### strings
**개요:** 바이너리 안에 들어 있는 읽을 수 있는 문자열을 뽑아내는 도구다.
**작동 방식:** 파일을 바이트 단위로 훑으며 연속된 출력 가능 문자를 모은다. 기본 최소 길이는 4이며 `-n`으로 조절한다.
```bash
strings -n 6 MyApp.app/Contents/MacOS/MyApp | grep -i version
```

### xml2man
**개요:** XML 문서를 man 페이지 소스로 바꾸는 도구다. hdxml2manxml 다음 단계에서 쓴다.
**작동 방식:** man용 XML 태그를 읽고 roff 매크로로 변환한다. 제목, 옵션, 예제 절을 man 규격 순서에 맞춘다.
```bash
xml2man MyDoc.man.xml > mydoc.1
```

# 4. Swift 툴체인

> 원문: `../CLT-tools-guide.md`에서 분리


Xcode Command Line Tools에 들어 있는 Swift 관련 도구 32개를 정리했다. 컴파일, 패키지 관리, 분석, 크로스 컴파일 설정까지 한 번에 훑을 수 있다.

### aarch64-swift-linux-musl-clang++.cfg
**개요:** ARM 64비트 Linux(musl)용 C++ 크로스 컴파일 설정 파일이다.
**작동 방식:** Swift 컴파일 파이프라인에서 C, C++ 상호 운용 코드가 필요할 때 clang++에게 전달할 플래그를 모아 둔다. 타깃 트리플, sysroot 경로, 링커 옵션, musl용 헤더와 라이브러리 경로가 들어 있다. swiftc가 이 파일을 읽으면 별도로 긴 옵션을 외우지 않아도 크로스 빌드가 된다.
```bash
swiftc -target aarch64-unknown-linux-musl main.swift -o hello-aarch64
```

### aarch64-swift-linux-musl-clang.cfg
**개요:** ARM 64비트 Linux(musl)용 C 컴파일 설정 파일이다.
**작동 방식:** Swift에서 C 코드를 함께 컴파일하거나 임포트할 때 이 설정이 쓰인다. clang에게 아키텍처, musl sysroot, include 경로를 알려준다. 패키지에 C 타깃이 섞여 있을 때 빌드 실패를 막아준다.
```bash
clang --config aarch64-swift-linux-musl-clang.cfg -c helper.c -o helper.o
```

### x86_64-swift-linux-musl-clang++.cfg
**개요:** 인텔 64비트 Linux(musl)용 C++ 크로스 컴파일 설정 파일이다.
**작동 방식:** 역할은 aarch64용과 같고 타깃 아키텍처만 x86_64이다. clang++에게 musl 헤더 위치, 정적 링킹 옵션, 링커 플래그를 넘긴다. Swift 패키지의 Cxx 타깃을 Linux용으로 같이 빌드할 때 참조된다.
```bash
swift build --swift-sdk x86_64-swift-linux-musl --build-path .build-linux
```

### x86_64-swift-linux-musl-clang.cfg
**개요:** 인텔 64비트 Linux(musl)용 C 컴파일 설정 파일이다.
**작동 방식:** C 소스를 x86_64 Linux 바이너리로 바꿀 때 필요한 기본 플래그를 담고 있다. sysroot, 타깃 트리플, musl 관련 라이브러리 경로가 핵심이다. SwiftPM이 C 타깃을 크로스 빌드할 때 내부적으로 이 파일을 쓴다.
```bash
clang --config x86_64-swift-linux-musl-clang.cfg -c helper.c -o helper.o
```

### sourcekit-lsp
**개요:** 에디터용 Swift 언어 서버이다. 자동 완성, 점프, 오류 표시를 맡는다.
**작동 방식:** 에디터가 LSP로 말을 걸면 코드를 분석해서 답을 준다. 컴파일러 프론트엔드와 SourceKit 엔진을 써서 심볼과 타입 정보를 뽑는다. 파일을 저장하지 않아도 실시간으로 진단한다.
```bash
sourcekit-lsp --help
```

### swift
**개요:** Swift 통합 실행기이다. REPL, 스크립트 실행, 버전 확인에 쓴다.
**작동 방식:** 입력된 코드를 swift-frontend로 넘기기 전에 인자를 정리하는 앞단이다. `swift run`, `swift test` 같은 하위 명령을 연결한다. 단일 파일은 바로 실행한다.
```bash
swift hello.swift
swift --version
```

### swift-api-digester
**개요:** API 변화 비교 도구이다. 라이브러리 공개 API가 깨졌는지 검사한다.
**작동 방식:** 기준 모듈 인터페이스와 새 빌드 결과를 비교한다. 삭제된 함수, 바뀐 시그니처, 추가된 제약을 찾아낸다. 컴파일 뒤 단계에서 ABI와 API 안정성을 점검한다.
```bash
swift-api-extract -o before.json MyLib.swift
swift-api-extract -o after.json MyLib.swift
swift-api-digester -diagnose-sdk --before before.json --after after.json
```

### swift-api-extract
**개요:** Swift 코드에서 공개 API 정보를 JSON으로 뽑는 도구이다.
**작동 방식:** 소스를 파싱해서 public과 open 심볼만 추린다. 함수 이름, 파라미터, 제네릭 조건까지 구조화한다. 이 JSON이 digester의 입력이 된다.
```bash
swift-api-extract MyLib.swift -o api.json
```

### swift-build
**개요:** SwiftPM의 실제 빌드 실행 파일이다. `swift build`가 내부적으로 부른다.
**작동 방식:** 패키지 매니페스트를 읽고 빌드 계획을 세운 뒤 swiftc와 clang을 순서대로 호출한다. 증분 빌드라 바뀐 파일만 다시 짓는다.
```bash
swift build -v
```

### swift-cache-tool
**개요:** 컴파일 캐시를 관리하는 도구이다. 재빌드를 빠르게 한다.
**작동 방식:** clang과 Swift 모듈 캐시에 쌓인 파일을 조회하거나 지운다. 빌드 결과를 직접 만들지는 않는다. 빌드가 이상하게 실패하면 캐시를 비우고 다시 시도한다.
```bash
swift-cache-tool --help
rm -rf ~/Library/Developer/Xcode/DerivedData/ModuleCache.noindex
swift build
```

### swift-demangle
**개요:** Swift의 난독화된 심볼 이름을 사람이 읽을 형태로 되돌린다.
**작동 방식:** 컴파일된 바이너리 속 이름을 원래 함수 모양으로 풀어준다. 크래시 로그나 `nm`, `otool` 출력 분석에 쓴다.
```bash
echo '$s4main5helloSSyF' | swift-demangle
```

### swift-driver
**개요:** 차세대 Swift 빌드 드라이버이다. 컴파일 작업을 계획하고 병렬 실행한다.
**작동 방식:** 어떤 파일을 언제, 어떤 순서로 컴파일할지 정한다. frontend 호출을 병렬로 돌려서 빌드가 빠르다. 보통 직접 부르지 않고 swiftc 뒤에서 일한다.
```bash
swift-driver --version
swiftc -driver-print-jobs main.swift -o main 2>&1 | head
```

### swift-experimental-sdk
**개요:** 실험적 Swift SDK를 다루는 도구이다.
**작동 방식:** SDK 번들을 내려받거나 로컬 경로에 등록한다. `swift build --swift-sdk`가 참조하는 SDK 목록을 관리한다. 컴파일 전에 타깃 플랫폼용 헤더와 라이브러리를 준비한다.
```bash
swift-experimental-sdk list
```

### swift-format
**개요:** 공식 Swift 코드 포매터이다.
**작동 방식:** 소스를 파싱한 뒤 정해진 스타일로 다시 출력한다. 컴파일 결과에는 영향을 주지 않는다. CI에서 `--lint` 모드로 검사한다.
```bash
swift-format format -i Sources/main.swift
swift-format lint Sources/main.swift
```

### swift-frontend
**개요:** Swift 컴파일러 본체이다. 파싱부터 코드 생성까지 다 한다.
**작동 방식:** swiftc와 driver가 넘긴 작업을 실제로 수행한다. 타입 검사, SIL 최적화, LLVM IR 생성, 오브젝트 출력이 차례로 일어난다.
```bash
swift-frontend -parse main.swift -typecheck 2>&1 | head
```

### swift-help
**개요:** Swift 도움말 출력기이다.
**작동 방식:** 각 하위 명령의 사용법을 보여준다. 별도 분석은 하지 않는다.
```bash
swift-help build
swift package --help
```

### swift-package
**개요:** SwiftPM 패키지 관리 명령의 본체이다.
**작동 방식:** `Package.swift`를 읽고 의존 그래프를 푼다. 필요한 경우 소스를 내려받는다. 그 뒤 swift-build에 실제 컴파일을 맡긴다.
```bash
swift package init --name Hello --type executable
swift package resolve
```

### swift-package-collection
**개요:** 패키지 모음 파일을 만들고 검증하는 도구이다.
**작동 방식:** 여러 패키지의 이름, 주소, 버전을 하나의 JSON 컬렉션으로 묶는다. 서명을 넣어 위변조를 막는다.
```bash
swift-package-collection generate --help
```

### swift-package-registry
**개요:** Swift 패키지 레지스트리와 통신하는 도구이다.
**작동 방식:** 패키지 아이디를 레지스트리 URL로 바꾸고 버전 목록을 받아온다. 깃 대신 중앙 서버에서 소스를 받는 방식이다.
```bash
swift package-registry --help
```

### swift-plugin-server
**개요:** SwiftPM 빌드 플러그인을 실행하는 서버이다.
**작동 방식:** 패키지 빌드 중에 플러그인 코드를 별도 프로세스로 띄운다. 코드 생성이나 린트 같은 작업을 본 빌드와 분리해서 안전하게 돌린다.
```bash
swift-plugin-server --help
```

### swift-run
**개요:** 빌드 뒤 실행을 맡는 내부 명령이다. `swift run`이 부른다.
**작동 방식:** swift-build로 만든 실행 파일을 찾아서 바로 실행한다. 인자를 그대로 전달한다.
```bash
swift run Hello -- --name 지민
```

### swift-sdk
**개요:** Swift SDK 목록과 설정을 관리하는 도구이다.
**작동 방식:** 설치된 SDK의 타깃, sysroot, 툴체인 경로를 보여주거나 추가한다. `swift build --swift-sdk`가 이 정보를 읽는다.
```bash
swift sdk list
```

### swift-stdlib-tool
**개요:** Swift 표준 라이브러리를 앱 번들에 복사하는 도구이다.
**작동 방식:** 빌드가 끝난 뒤 필요한 `libswift*.dylib`를 찾아 실행 파일 옆에 넣는다. 구형 macOS 지원을 위해 `@rpath`를 고친다.
```bash
swift-stdlib-tool --help
otool -L .build/debug/Hello | head
```

### swift-symbolgraph-extract
**개요:** 심볼 그래프를 뽑는 문서화 도구이다. DocC의 입력이 된다.
**작동 방식:** 컴파일된 모듈에서 심볼, 주석, 관계를 JSON으로 뽑는다. 상속과 프로토콜 채택까지 연결한다.
```bash
swift symbolgraph-extract -module-name MyLib -target x86_64-apple-macosx -o symbols.json
```

### swift-synthesize-interface
**개요:** 바이너리나 모듈에서 `.swiftinterface`를 복원하는 도구이다.
**작동 방식:** 컴파일된 swiftmodule을 읽어 사람이 보는 인터페이스 파일로 바꾼다. private 구현은 빼고 public 선언만 남긴다.
```bash
swiftc -emit-module MyLib.swift -o MyLib.swiftmodule
```

### swift-test
**개요:** 테스트 실행기 본체이다. `swift test`가 내부적으로 부른다.
**작동 방식:** XCTest와 Swift Testing 타깃을 찾아 빌드한 뒤 순서대로 돌린다. 병렬 실행과 필터를 지원한다.
```bash
swift test
swift test --filter HelloTests.testGreeting
```

### swiftc
**개요:** 대표 Swift 컴파일러 명령이다.
**작동 방식:** driver에게 작업을 맡기고 frontend가 실제 컴파일을 한다. 단일 파일부터 패키지 전체까지 다룬다.
```bash
swiftc main.swift -o hello
./hello 지민
```

### xcindex-test
**개요:** Xcode 인덱스 데이터를 검사하는 진단 도구이다.
**작동 방식:** SourceKit 인덱스 저장소에 쌓인 심볼 정보를 읽는다. 점프 정의나 자동 완성이 꼬였을 때 원인을 찾는다.
```bash
xcindex-test --help
```

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

### scalar
**개요:** Swift와 C++ 상호 운용 진단용 소규모 도구이다.
**작동 방식:** Swift 숫자 타입과 C++ 스칼라 타입 사이의 변환을 점검한다. 컴파일 파이프라인의 상호 운용 단계에서 타입 불일치를 미리 잡는다.
```bash
scalar --help
```

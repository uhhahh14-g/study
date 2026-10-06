# SDK-크로스컴파일

> 대분류: `06-Swift-패키지` / 타깃 플랫폼 준비. SDK 관리와 리눅스 크로스 설정

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

### swift-sdk
**개요:** Swift SDK 목록과 설정을 관리하는 도구이다.
**작동 방식:** 설치된 SDK의 타깃, sysroot, 툴체인 경로를 보여주거나 추가한다. `swift build --swift-sdk`가 이 정보를 읽는다.
```bash
swift sdk list
```

### swift-experimental-sdk
**개요:** 실험적 Swift SDK를 다루는 도구이다.
**작동 방식:** SDK 번들을 내려받거나 로컬 경로에 등록한다. `swift build --swift-sdk`가 참조하는 SDK 목록을 관리한다. 컴파일 전에 타깃 플랫폼용 헤더와 라이브러리를 준비한다.
```bash
swift-experimental-sdk list
```

### swift-stdlib-tool
**개요:** Swift 표준 라이브러리를 앱 번들에 복사하는 도구이다.
**작동 방식:** 빌드가 끝난 뒤 필요한 `libswift*.dylib`를 찾아 실행 파일 옆에 넣는다. 구형 macOS 지원을 위해 `@rpath`를 고친다.
```bash
swift-stdlib-tool --help
otool -L .build/debug/Hello | head
```


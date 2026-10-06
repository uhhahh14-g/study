# API-문서-인덱스

> 대분류: `06-Swift-패키지` / 공개 API 추출·호환성 검사·인덱싱

### swift-api-extract
**개요:** Swift 코드에서 공개 API 정보를 JSON으로 뽑는 도구이다.
**작동 방식:** 소스를 파싱해서 public과 open 심볼만 추린다. 함수 이름, 파라미터, 제네릭 조건까지 구조화한다. 이 JSON이 digester의 입력이 된다.
```bash
swift-api-extract MyLib.swift -o api.json
```

### swift-api-digester
**개요:** API 변화 비교 도구이다. 라이브러리 공개 API가 깨졌는지 검사한다.
**작동 방식:** 기준 모듈 인터페이스와 새 빌드 결과를 비교한다. 삭제된 함수, 바뀐 시그니처, 추가된 제약을 찾아낸다. 컴파일 뒤 단계에서 ABI와 API 안정성을 점검한다.
```bash
swift-api-extract -o before.json MyLib.swift
swift-api-extract -o after.json MyLib.swift
swift-api-digester -diagnose-sdk --before before.json --after after.json
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

### xcindex-test
**개요:** Xcode 인덱스 데이터를 검사하는 진단 도구이다.
**작동 방식:** SourceKit 인덱스 저장소에 쌓인 심볼 정보를 읽는다. 점프 정의나 자동 완성이 꼬였을 때 원인을 찾는다.
```bash
xcindex-test --help
```


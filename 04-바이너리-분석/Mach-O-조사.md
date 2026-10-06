# Mach-O-조사

> 대분류: `04-바이너리-분석` / Mach-O 구조·의존성·섹션 읽기 전용 분석

### otool
**개요:** macOS 대표 Mach-O 분석 도구다. 초보가 가장 먼저 만난다.
**작동 방식:** Mach-O 바이너리를 입력으로 읽어 로드 커맨드를 파싱한다. `-L`은 링크된 dylib 목록을, `-l`은 전체 로드 커맨드를, `-tV`는 역어셈블을 뽑는다. 의존성 확인부터 간단한 디스어셈블까지 텍스트로 보여준다.
```bash
otool -L /bin/ls
otool -l ./a.out | head -40
```

### otool-classic
**개요:** 구형 otool 구현이다. 호환 확인용이다.
**작동 방식:** 같은 Mach-O를 입력으로 읽는다. 구형 로드 커맨드 파서로 헤더를 해석한다. 최신 항목은 표시가 다를 수 있다. otool과 같은 형식 텍스트로 출력한다.
```bash
otool-classic -L /bin/ls
otool-classic -hv ./a.out
```

### llvm-otool
**개요:** otool의 LLVM 재구현이다. Mach-O 보기에 쓴다.
**작동 방식:** Mach-O 파일을 입력으로 읽는다. 로드 커맨드와 섹션을 otool과 같은 형식으로 파싱한다. `-L`은 의존 dylib, `-l`은 로드 커맨드 전체를 푼다. 결과를 otool 호환 텍스트로 출력한다.
```bash
llvm-otool -L /bin/ls
llvm-otool -l ./a.out | head -40
```

### llvm-objdump
**개요:** Mach-O 역어셈블과 헤더 덤프용 LLVM 도구다.
**작동 방식:** 바이너리 파일을 입력으로 읽어 Mach-O 로드 커맨드를 파싱한다. 텍스트 섹션 바이트를 디스어셈블러로 푼다. 심볼과 DWARF 줄 정보를 어셈블리 옆에 붙인다. 헤더, 섹션, 역어셈블 결과를 텍스트로 출력한다.
```bash
llvm-objdump -d --syms ./a.out | head -40
llvm-objdump -macho -private-headers /bin/ls | head -30
```

### objdump
**개요:** GNU 스타일 오브젝트 덤프 도구다. macOS CLT에도 들어 있다.
**작동 방식:** Mach-O나 오브젝트 파일을 입력으로 읽는다. 섹션 헤더와 역어셈블을 푼다. 텍스트 리포트로 출력한다.
```bash
objdump -h ./a.out | head -20
objdump -d --syms ./a.out | head -40
```

### dyld_info
**개요:** Mach-O의 dyld 바인딩 정보를 보여주는 전용 도구다.
**작동 방식:** Mach-O 실행 파일이나 dylib를 입력으로 읽는다. 로드 커맨드의 바인딩 정보를 파싱한다. rebase, bind, lazy bind, export trie 항목을 분리한다. 동적 링커가 시작 시 무엇을 고치는지 목록으로 출력한다.
```bash
dyld_info -bind /bin/ls | head -20
dyld_info -exports /usr/lib/libSystem.dylib | head -20
```

### pagestuff
**개요:** Mach-O 실행 파일의 논리 페이지 구성을 덤프하는 도구다.
**작동 방식:** 바이너리 헤더와 로드 명령을 읽는다. 각 페이지가 어떤 세그먼트에 속하는지 표로 보여준다.
```bash
pagestuff /bin/ls -p
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

### llvm-size
**개요:** 섹션별 크기를 보여주는 size 도구다.
**작동 방식:** Mach-O 파일을 입력으로 읽어 세그먼트와 섹션 헤더를 파싱한다. 영역 크기를 합산한다. Berkeley나 Darwin 포맷으로 정리한다. 바이너리 다이어트 전후 비교에 쓴다.
```bash
llvm-size -m ./a.out
llvm-size -l /bin/ls | head -20
```

### segedit
**개요:** Mach-O 세그먼트와 섹션 내용을 덤프, 추출하는 조사 도구다.
**작동 방식:** 링크 완료 후 분석 단계에서 쓴다. 입력은 실행 파일이나 dylib, 출력은 `__TEXT,__info_plist` 같은 섹션 덤프다. 바이너리를 수정하지 않고 읽기만 한다. 서명 전후 구조 확인할 때 편하다.
```bash
segedit MyApp -extract __TEXT __cstring /tmp/cstr.bin
xxd /tmp/cstr.bin | head
```

### unwinddump
**개요:** 언와인드 정보를 덤프하는 도구다. 예외와 백트레이스 분석용이다.
**작동 방식:** Mach-O 파일 속 compact unwind 테이블과 DWARF CFI를 입력으로 읽는다. 함수 주소 범위별 언와인드 인코딩을 파싱한다. 주소별 언와인드 규칙 목록으로 출력한다.
```bash
unwinddump ./a.out | head -40
unwinddump -eh-frame /bin/ls | head -20
```

### llvm-readtapi
**개요:** `.tbd` 파일과 dylib ABI를 읽고 비교하는 도구다.
**작동 방식:** TBD 텍스트 파일이나 dylib를 입력으로 읽는다. export 심볼, 아키텍처, 플랫폼, 버전을 파싱한다. 두 TBD 사이 추가, 삭제 심볼을 대조한다. 검증 결과와 정규화된 TBD를 출력한다.
```bash
llvm-readtapi compare libFoo.tbd libFoo_new.tbd
```

### readtapi
**개요:** `.tapi`와 `.tbd` 파일을 읽어 내용을 보여주는 도구다.
**작동 방식:** TAPI 문서를 파싱해 아키텍처 목록을 뽑는다. 익스포트된 심볼과 허용된 클라이언트를 표시한다.
```bash
readtapi /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib/libSystem.tbd
```

### strings
**개요:** 바이너리 안에 들어 있는 읽을 수 있는 문자열을 뽑아내는 도구다.
**작동 방식:** 파일을 바이트 단위로 훑으며 연속된 출력 가능 문자를 모은다. 기본 최소 길이는 4이며 `-n`으로 조절한다.
```bash
strings -n 6 MyApp.app/Contents/MacOS/MyApp | grep -i version
```


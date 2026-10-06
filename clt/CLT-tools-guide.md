# Xcode Command Line Tools 137개 도구 가이드

> 위치: `/Library/Developer/CommandLineTools/usr/bin` (총 137개)
> 각 도구의 개요, 작동 방식, macOS 예시를 정리했다.

## 1. 버전관리

### git
**개요**
분산 기록을 만들고 나누기 위한 기본 명령이다. 저장소 만들기, 변경 묶기, 가지 나누기, 올리기와 받기를 모두 맡는다.

**작동 방식**
먼저 작업 폴더에 있는 파일 상태를 모아서 개체 창고에 넣는다. 다음으로 바꾼 내용을 묶어서 갈래에 매단다. 올리기를 하면 묶음과 갈래 끝 위치를 함께 건네준다. 받기를 하면 상대 쪽 갈래 끝을 가져와 내 기록에 잇는다.

**예시**
```sh
git --version
git init /tmp/연습
cd /tmp/연습 && echo "안녕" > 읽어보기.txt && git add 읽어보기.txt && git commit -m "첫 묶음"
```

### git-receive-pack
**개요**
올리기를 받는 쪽에서 밀어넣기를 처리하는 받아들이기 전용 명령이다. 보통 직접 치지 않고 올리기 과정에서 저절로 불린다.

**작동 방식**
먼저 올리는 쪽에서 보낸 갈래 끝 위치와 묶음 꾸러미를 표준 입력으로 받는다. 다음으로 각 갈래가 빨리 감기로 이어지는지, 갈고리 검사를 통과하는지 차례로 따진다. 조건을 만족하면 묶음을 개체 창고에 풀고 갈래 끝을 새 위치로 옮긴다. 마지막으로 성공과 실패 목록을 꾸려 올리는 쪽에 돌려준다.

**예시**
```sh
git init --bare /tmp/받기연습.git
printf '0000' | git-receive-pack /tmp/받기연습.git
ssh 연습계정@연습호스트 "git-receive-pack '/저장소/모음.git'"
```

### git-shell
**개요**
바깥 접속자가 정해진 깃 일만 하도록 묶어 두는 제한 겉껍질이다. 넣기, 받기, 묶음 받기 같은 정해진 일 외에는 막는다.

**작동 방식**
먼저 접속자가 보내 온 명령 줄을 받아 허용 목록과 맞춰 본다. 목록에 있는 올리기 받기, 내려받기 보내기, 묶음 꺼내기에만 길을 열어준다. 대화식 끝말잇기나 파일 지우기 같은 다른 명령은 바로 끊는다. 마지막으로 허용된 명령을 해당 저장소 경로에 묶어 실행한다.

**예시**
```sh
which git-shell
git-shell -c help
chsh -s "$(which git-shell)" 연습계정
```

### git-upload-archive
**개요**
기록 묶음 꺼내기를 받는 쪽에서 내려보내기 요청을 처리하는 명령이다. 특정 시점의 묶음을 꾸려서 내보낼 때 쓴다.

**작동 방식**
먼저 내려받는 쪽에서 보낸 갈래 이름이나 묶음 번호와 꾸림 방식을 표준 입력으로 받는다. 다음으로 해당 시점의 나무 구조를 개체 창고에서 찾아 차례로 읽는다. 허용된 꾸림 방식인지, 내보내기 금지 글이 있는지 확인한다. 마지막으로 묶은 흐름을 표준 출력으로 내보내고 연결을 닫는다.

**예시**
```sh
git init /tmp/꾸림연습 && cd /tmp/꾸림연습 && touch 가.txt && git add 가.txt && git commit -m "기초"
git archive --list
printf '0000' | git-upload-archive /tmp/꾸림연습
```

### git-upload-pack
**개요**
내려받기와 가져오기를 보내는 쪽에서 요청을 처리하는 내보내기 전용 명령이다. 복제와 가져오기가 불리면 뒤에서 실제 일을 한다.

**작동 방식**
먼저 내려받는 쪽에서 가진 갈래 끝 목록을 받아 내가 가진 갈래 끝 목록과 맞춘다. 다음으로 상대가 모자란 묶음과 나무 조각을 가려내어 꾸러미로 묶는다. 얇게 묶기나 점진 묶기가 오면 그 방식에 맞추어 크기를 줄인다. 마지막으로 꾸러미와 갈래 끝 위치를 표준 출력으로 흘려보낸다.

**예시**
```sh
git init --bare /tmp/보내기연습.git
git-upload-pack /tmp/보내기연습.git
git clone --depth 1 file:///tmp/보내기연습.git /tmp/받기복제
```

## 2. 컴파일·빌드·링크 도구 30선

CLT의 컴파일과 링크 파이프라인을 담당하는 핵심 도구 모음. 전처리기, 컴파일러, 어셈블러, 링커, 아카이버, 바이너리 조작 순서로 이해하면 쉽다.

### ar
**개요:** 여러 오브젝트 파일(.o)을 하나의 정적 아카이브(.a)로 묶는 아카이버다.
**작동 방식:** 컴파일 파이프라인 중 링커 직전 단계에서 사용된다. 입력은 `cc`나 `clang`이 만든 .o 파일들이고, 출력은 정적 라이브러리인 .a 파일이다. 링커인 `ld`가 이 .a를 풀어서 최종 실행 파일에 연결한다. 실행 파일이 아니라 중간 산출물을 만드는 도구다.
```bash
clang -c add.c -o add.o
ar rcs libadd.a add.o
```

### as
**개요:** 어셈블리 코드(.s)를 기계어 오브젝트(.o)로 바꾸는 어셈블러다.
**작동 방식:** 파이프라인에서 전처리기, 컴파일러 다음에 위치한다. 입력은 `clang -S`가 만든 어셈블리 텍스트이고, 출력은 재배치 가능한 오브젝트 파일이다. 이 .o는 이후 `ld`가 링킹해서 실행 파일이 된다. 보통 직접 호출보다 `clang` 경유로 간접 실행된다.
```bash
clang -S hello.c -o hello.s
as hello.s -o hello.o
```

### asa
**개요:** 어셈블리 파이프라인을 보조하는 Apple 전용 어셈블러 프런트엔드다.
**작동 방식:** `as`와 같은 어셈블러 단계에 위치한다. 입력은 어셈블리 소스이고, 출력은 Mach-O 오브젝트다. 아키텍처 옵션 처리나 `clang` 드라이버와의 연동을 맡는다. 일반 사용자는 직접 쓸 일이 거의 없다.
```bash
asa -arch arm64 hello.s -o hello.o
```

### bitcode_strip
**개요:** Mach-O 바이너리에서 embedded bitcode를 제거하는 정리 도구다.
**작동 방식:** 링커 이후, 배포 직전 단계에서 동작한다. 입력은 bitcode가 포함된 앱 바이너리나 .a 파일이고, 출력은 bitcode 섹션이 제거된 바이너리다. 앱 사이즈를 줄이고 스토어 제출 조건을 맞출 때 쓴다. 코드 동작 자체는 바꾸지 않는다.
```bash
bitcode_strip -r MyApp -o MyApp.stripped
```

### c++
**개요:** C++용 컴파일러 드라이버로, 실체는 `clang++`에 대한 심링크다.
**작동 방식:** 파이프라인 전체를 총괄한다. 전처리, 컴파일, 어셈블, 링크를 순서대로 호출하고, 입력은 .cpp 파일, 출력은 실행 파일이나 .o다. C++ 표준 라이브러리인 libc++를 자동으로 연결하는 점이 `cc`와 다르다.
```bash
c++ -std=c++17 -O2 main.cpp -o main
./main
```

### c89
**개요:** C89 규격에 맞게 동작하는 C 컴파일러 호환 드라이버다.
**작동 방식:** `clang`을 C89 모드로 감싼 래퍼에 가깝다. 입력은 .c 소스, 출력은 실행 파일 또는 오브젝트다. 컴파일러 단계에서 언어 규격을 엄격히 제한한다. 오래된 코드의 호환성 확인용으로 쓴다.
```bash
c89 -pedantic -Wall legacy.c -o legacy
./legacy
```

### c99
**개요:** C99 규격 기준 C 컴파일러 호환 드라이버다.
**작동 방식:** 파이프라인 위치는 `c89`와 동일하고, 컴파일러 단계의 언어 모드만 다르다. 입력은 C99 문법을 쓴 .c 파일, 출력은 Mach-O 실행 파일이다. 가변 길이 배열이나 `//` 주석 등을 허용한다. 학습용이나 이식성 점검에 쓴다.
```bash
c99 -Wall loop.c -o loop
./loop
```

### cc
**개요:** 시스템 기본 C 컴파일러를 가리키는 대표 심링크다.
**작동 방식:** 전처리기, 컴파일러, 어셈블러, 링커를 순서대로 호출하는 드라이버다. 입력은 .c 파일, 출력은 a.out이나 지정한 실행 파일이다. macOS에서는 `clang`과 동일하게 동작한다. Makefile에서 `$(CC)`로 가장 많이 불린다.
```bash
cc -O2 hello.c -o hello
./hello
```

### clang
**개요:** CLT의 실제 C, Objective-C 컴파일러 본체다.
**작동 방식:** 컴파일 파이프라인의 중심이다. `cpp`로 전처리한 뒤 중간 표현(IR)으로 바꾸고, `as`로 어셈블한 뒤 `ld`에 링킹을 맡긴다. 입력은 .c, .m 파일, 출력은 .o나 실행 파일이다. 경고 메시지가 친절해서 초보에게 좋다.
```bash
clang -Wall -g hello.c -o hello
./hello
```

### clang++
**개요:** C++와 Objective-C++용 실제 컴파일러 본체다.
**작동 방식:** `clang`과 파이프라인 위치가 같고, C++ 프런트엔드와 libc++ 링크가 추가된다. 입력은 .cpp, .mm 파일이고, 출력은 실행 파일이나 dylib이다. `c++`, `g++`는 대부분 이 바이너리를 가리킨다.
```bash
clang++ -std=c++17 app.cpp -o app
./app
```

### clang-cache
**개요:** clang 모듈과 프리컴파일드 헤더를 캐싱하는 헬퍼다.
**작동 방식:** 컴파일러 바로 앞단에서 동작한다. 입력은 헤더 모듈 요청, 출력은 캐시된 PCM이나 PCH다. 같은 헤더를 매번 다시 파싱하지 않게 해서 빌드를 빠르게 한다. 사용자가 직접 치기보다 Xcode와 `clang`이 내부적으로 쓴다.
```bash
ls /var/folders/*/*/*/clang-cache 2>/dev/null || echo "cache dir check"
clang -v -c hello.c -o hello.o
```

### clang-stat-cache
**개요:** 파일 상태 조회 결과를 캐싱해 헤더 탐색을 빠르게 하는 헬퍼다.
**작동 방식:** 전처리기와 컴파일러 사이의 파일 탐색 단계에서 동작한다. 입력은 `#include` 경로 조회 요청, 출력은 캐시된 stat 결과다. 디스크 접근을 줄여 대규모 빌드 속도를 올린다. 직접 실행용 도구가 아니다.
```bash
clang -H -c hello.c -o hello.o 2>&1 | head
```

### cmpdylib
**개요:** 두 dylib의 호환성을 비교하는 점검 도구다.
**작동 방식:** 링커 이후, 빌드 검증 단계에서 쓴다. 입력은 신구 두 개의 .dylib 파일, 출력은 심볼 차이 리포트다. TAPI나 TBD 파일 생성 전 확인용으로 쓴다. 링크가 깨질 변경을 미리 잡는다.
```bash
cmpdylib libold.dylib libnew.dylib
```

### codesign_allocate
**개요:** Mach-O에 코드 서명 공간을 미리 확보하는 저수준 도구다.
**작동 방식:** 링커가 바이너리를 만든 직후, `codesign` 실행 전에 동작한다. 입력은 서명 없는 Mach-O, 출력은 서명용 여유 공간이 확보된 Mach-O다. 세그먼트 오프셋을 조정해서 서명이 들어갈 자리를 만든다. 보통 `ld`가 자동으로 호출한다.
```bash
codesign_allocate -i MyApp -o MyApp.signed_space
codesign -s - MyApp.signed_space
```

### codesign_allocate-p
**개요:** `codesign_allocate`의 페이지 단위 처리 변형이다.
**작동 방식:** 파이프라인 위치와 입출력 흐름은 `codesign_allocate`와 같다. 입력 Mach-O를 페이지 정렬 기준으로 다뤄서 서명 슬롯을 만든다. 큰 바이너리나 특정 정렬이 필요한 경우에 쓰인다. 일반 빌드에서는意識하지 않아도 된다.
```bash
codesign_allocate-p -i MyApp -o MyApp.paged
codesign -s - MyApp.paged
```

### cpp
**개요:** C 전처리기 단독 실행 도구다.
**작동 방식:** 파이프라인 맨 앞에 위치한다. 입력은 `#include`, `#define`이 포함된 .c 파일, 출력은 순수 C 코드 텍스트다. 이후 `clang` 본체가 이 결과를 컴파일한다. 매크로 확장 결과를 확인할 때 `-E` 대신 직접 쓸 수 있다.
```bash
cpp config.h -o config.i
clang config.i -c -o config.o
```

### ctf_insert
**개요:** 바이너리에 CTF(Compact Type Format) 디버그 정보를 삽입하는 도구다.
**작동 방식:** 컴파일과 링크가 끝난 뒤, 디버그 후처리 단계에서 동작한다. 입력은 DWARF가 있는 오브젝트나 dSYM 정보, 출력은 CTF 섹션이 추가된 바이너리다. `dtrace` 같은 도구가 타입을 읽을 때 쓴다. 일반 앱 개발에서는 거의 만날 일이 없다.
```bash
clang -g driver.c -o driver
ctf_insert -o driver.ctf driver
```

### g++
**개요:** C++ 컴파일러 호환 드라이버로, macOS에서는 `clang++`와 같다.
**작동 방식:** 전처리부터 링크까지 전체를 조율한다. 입력은 .cpp, 출력은 실행 파일이다. GNU 옵션을 받아주지만 내부 동작은 clang 기반이다. 리눅스용 Makefile을 그대로 돌릴 때 유용하다.
```bash
g++ -O2 -Wall main.cpp -o main
./main
```

### gcc
**개요:** C 컴파일러 호환 드라이버로, 실체는 `clang`이다.
**작동 방식:** 파이프라인 전체를 드라이브한다. 입력 .c를 전처리, 컴파일, 어셈블, 링크해서 Mach-O 실행 파일로 만든다. 진짜 GCC가 아니므로 GCC 전용 플래그는 일부 동작하지 않는다. `cc`와 거의 바꿔 쓸 수 있다.
```bash
gcc -O2 hello.c -o hello
./hello
```

### gnumake
**개요:** GNU Make를 명시적으로 가리키는 바이너리 이름이다.
**작동 방식:** 컴파일 파이프라인을 위에서 조율하는 빌드 러너다. 입력은 Makefile과 소스들, 출력은 `clang`, `ar`, `ld` 호출 결과물이다. 바뀐 파일만 골라서 다시 컴파일한다. `make`와 동일하고 이름 충돌 회피용으로 둔다.
```bash
gnumake -j4
./hello
```

### install_name_tool
**개요:** dylib의 install name과 의존 경로를 고치는 바이너리 편집기다.
**작동 방식:** 링크가 끝난 뒤, 배포 패키징 단계에서 쓴다. 입력은 이미 링크된 실행 파일이나 dylib, 출력은 내부 경로 테이블이 수정된 바이너리다. `@rpath`, `@loader_path`를 바꿔서 앱 번들 안의 라이브러리를 찾게 한다. 재컴파일 없이 경로만 고친다.
```bash
install_name_tool -id @rpath/libfoo.dylib libfoo.dylib
install_name_tool -change /usr/local/lib/libfoo.dylib @rpath/libfoo.dylib MyApp
```

### ld
**개요:** 오브젝트들을 묶어 실행 파일과 dylib를 만드는 링커다.
**작동 방식:** 파이프라인 마지막 연결 단계다. 입력은 여러 .o와 .a, .dylib, 출력은 Mach-O 실행 파일이다. 심볼 해결, 주소 재배치, `codesign_allocate` 호출까지 맡는다. 보통 `clang`이 뒤에서 호출하므로 직접 칠 일은 적다.
```bash
clang -c a.c b.c
ld a.o b.o -lSystem -o mytool
./mytool
```

### ld-classic
**개요:** 구세대 링커 구현으로, 호환성 유지용으로 남겨둔 버전이다.
**작동 방식:** 파이프라인 위치는 현행 `ld`와 동일하다. 입력 .o들을 받아 Mach-O를 만들고, 출력은 실행 파일이다. 새 링커에서 문제가 생길 때 폴백용으로 쓴다. 일반 신규 프로젝트에서는 쓸 필요가 없다.
```bash
clang -c hello.c -o hello.o
ld-classic hello.o -lSystem -o hello_classic
```

### libtool
**개요:** 정적 라이브러리와 동적 라이브러리를 만드는 Apple 전용 도구다.
**작동 방식:** 어셈블러와 링커 사이에 위치한다. 입력은 .o 파일들, 출력은 .a 또는 .dylib다. `ar`보다 Mach-O 특화 옵션이 많고 lipo용 fat 라이브러리도 만든다. Xcode의 라이브러리 타깃 뒤에서 자주 불린다.
```bash
clang -c a.c b.c
libtool -static a.o b.o -o libab.a
```

### lipo
**개요:** Universal 바이너리를 만들고 뜯어보는 fat 바이너리 관리자다.
**작동 방식:** 링크 이후 패키징 단계에서 동작한다. 입력은 arm64, x86_64용 Mach-O들, 출력은 하나로 합친 Universal 파일이다. 반대로 Universal에서 특정 아키텍처만 추출할 수도 있다. 배포용 바이너리 만들 때 필수다.
```bash
lipo -create arm_app x86_app -output UniversalApp
lipo -info UniversalApp
```

### lorder
**개요:** 아카이브 멤버의 의존 순서 목록을 뽑는 정렬 헬퍼다.
**작동 방식:** `ar`로 묶기 전이나 `ranlib`과 함께 쓴다. 입력은 .o와 .a 파일, 출력은 심볼 의존 순서가 적힌 텍스트다. 순서에 민감한 구형 정적 링크를 돕는다. 요즘 clang, ld 조합에서는 거의 필요 없다.
```bash
lorder a.o b.o libfoo.a
ar crs libsorted.a a.o b.o
```

### make
**개요:** Makefile 기준으로 빌드를 자동화하는 표준 빌드 러너다.
**작동 방식:** 파이프라인 최상위에서 `cc`, `ar`, `ld` 호출을 조율한다. 입력은 Makefile과 소스 트리, 출력은 실행 파일과 라이브러리다. 타임스탬프로 변경분만 다시 빌드한다. CLT 초보는 이 도구부터 익히면 된다.
```bash
make
./hello
```

### ranlib
**개요:** 정적 아카이브에 심볼 색인을付けて 링크 속도를 올리는 도구다.
**작동 방식:** `ar` 직후에 동작한다. 입력은 색인 없는 .a 파일, 출력은 색인이 추가된 .a다. 이후 `ld`가 원하는 .o를 빨리 찾는다. 요즘 `ar s`가 같은 일을 해서 단독 호출은 줄었다.
```bash
ar cr libfoo.a a.o b.o
ranlib libfoo.a
```

### segedit
**개요:** Mach-O 세그먼트와 섹션 내용을 덤프, 추출하는 조사 도구다.
**작동 방식:** 링크 완료 후 분석 단계에서 쓴다. 입력은 실행 파일이나 dylib, 출력은 `__TEXT,__info_plist` 같은 섹션 덤프다. 바이너리를 수정하지 않고 읽기만 한다. 서명 전후 구조 확인할 때 편하다.
```bash
segedit MyApp -extract __TEXT __cstring /tmp/cstr.bin
xxd /tmp/cstr.bin | head
```

### strip
**개요:** 바이너리에서 심볼과 디버그 정보를 걷어내 크기를 줄이는 도구다.
**작동 방식:** 링크 맨 마지막, 배포 직전에 쓴다. 입력은 심볼이 든 실행 파일, 출력은 가벼운 바이너리다. `-S`는 디버그만, `-x`는 로컬 심볼만 지운다. 디버깅이 끝난 릴리스 빌드에 쓴다.
```bash
clang -g hello.c -o hello
strip -S hello
ls -lh hello
```

## 3. 디버깅·바이너리 분석

Xcode Command Line Tools에 들어 있는 디버깅과 바이너리 분석 도구 27개를 정리한다. 크래시 로그 읽기, 심볼 확인, Mach-O 구조 보기가 중심이다.

### c++filt
**개요:** C++ 맹글링된 심볼 이름을 사람이 읽을 수 있게 되돌리는 필터다.
**작동 방식:** 표준 입력이나 인자로 맹글 이름을 받는다. Itanium C++ ABI 규칙에 따라 파싱해서 네임스페이스, 클래스, 인자 타입을 복원한다. 복원된 함수 형태를 표준 출력으로 내보낸다. nm나 otool 출력과 파이프로 함께 쓰기 좋다.
```bash
nm -gU /usr/lib/libc++.dylib | head -5 | c++filt
echo "_Z3fooi" | c++filt
```

### clang-format
**개요:** C, C++, Objective-C 코드를 정해진 스타일로 자동 정렬하는 서식 도구다.
**작동 방식:** 소스 파일을 입력으로 읽는다. Clang 파서가 토큰과 AST 구조를 파악해 줄바꿈과 들여쓰기 규칙을 적용한다. `.clang-format` 설정이 있으면 그 스타일을 우선한다. 결과를 표준 출력이나 제자리 수정으로 내보낸다.
```bash
clang-format -i main.m
clang-format --style=LLVM main.cpp | head -20
```

### clang-format-diff.py
**개요:** diff 결과 중 바뀐 줄에만 clang-format을 적용하는 파이썬 스크립트다.
**작동 방식:** unified diff를 입력으로 받는다. 바뀐 줄 번호 범위를 파싱해 clang-format에 전달한다. 해당 범위만 다시 서식을 맞추고 나머지는 손대지 않는다. 큰 코드베이스에서 부분 정리에 쓴다.
```bash
git diff -U0 HEAD^ | clang-format-diff.py -p1 -i
```

### clangd
**개요:** 에디터용 C/C++ 언어 서버다. 자동 완성, 정의 이동, 오류 검사를 맡는다.
**작동 방식:** 소스 파일과 `compile_commands.json`을 입력으로 읽는다. Clang 프론트엔드로 AST와 인덱스를 만들고 변경분을 증분 분석한다. LSP 메시지로 에디터와 진단, 완성 후보, 호버 정보를 주고받는다. 백그라운드 인덱싱으로 프로젝트 전체 탐색을 빠르게 한다.
```bash
clangd --check=main.cpp
ls .clangd compile_commands.json
```

### crashlog
**개요:** macOS 크래시 로그(.crash, .ips)를 심볼화해서 읽기 좋게 바꾸는 도구다.
**작동 방식:** 크래시 로그 파일과 앱 바이너리, dSYM을 입력으로 받는다. 로그 속 로드 주소와 백트레이스 주소를 파싱한다. DWARF와 심볼 테이블로 주소를 파일명, 함수명, 줄 번호로 바꾼다. 심볼화된 리포트를 표준 출력으로 보여준다.
```bash
xcrun crashlog ~/Library/Logs/DiagnosticReports/MyApp-2026-10-06.ips
```

### dsymutil
**개요:** 흩어진 디버그 정보를 모아 `.dSYM` 번들을 만드는 도구다.
**작동 방식:** 빌드된 Mach-O 바이너리와 오브젝트 파일 속 DWARF를 입력으로 읽는다. 중복 디버그 정보를 링크하듯 합치고 압축한다. `MyApp.app.dSYM` 번들을 출력한다. 이 번들이 있어야 나중에 크래시 로그 심볼화가 된다.
```bash
dsymutil build/MyApp -o build/MyApp.dSYM
dwarfdump --uuid build/MyApp.dSYM
```

### dwarfdump
**개요:** Mach-O나 dSYM 속 DWARF 디버그 정보를 덤프하는 고전 도구다.
**작동 방식:** 바이너리나 dSYM 경로를 입력으로 받는다. `__DWARF` 세그먼트와 DIE 트리를 파싱한다. 컴파일 유닛, 함수, 변수, 줄 번호표를 사람이 읽을 형태로 푼다. 특정 DIE나 주소만 골라 출력할 수도 있다.
```bash
dwarfdump --debug-info build/MyApp.dSYM | head -40
```

### dyld_info
**개요:** Mach-O의 dyld 바인딩 정보를 보여주는 전용 도구다.
**작동 방식:** Mach-O 실행 파일이나 dylib를 입력으로 읽는다. 로드 커맨드의 바인딩 정보를 파싱한다. rebase, bind, lazy bind, export trie 항목을 분리한다. 동적 링커가 시작 시 무엇을 고치는지 목록으로 출력한다.
```bash
dyld_info -bind /bin/ls | head -20
dyld_info -exports /usr/lib/libSystem.dylib | head -20
```

### gcov
**개요:** gcda, gcno 파일로 코드 커버리지를 보여주는 GNU 방식 도구다.
**작동 방식:** `--coverage`로 빌드한 뒤 실행해서 생긴 카운터 파일을 입력으로 읽는다. 소스 줄과 실행 횟수를 대조한다. 실행된 줄과 안 탄 분기를 표시한다. 텍스트 리포트로 출력한다.
```bash
clang --coverage main.c -o main && ./main
gcov main.c && head -20 main.c.gcov
```

### lldb
**개요:** macOS 기본 디버거다. 중단점, 스텝 실행, 메모리 조사를 맡는다.
**작동 방식:** 실행 파일과 dSYM을 입력으로 읽어 심볼과 DWARF를 로드한다. Mach-O를 메모리에 올리고 중단점을 주소에 심는다. 스레드 정지 시 레지스터와 스택을 읽는다. `p`, `bt` 같은 명령 결과를 콘솔에 보여준다.
```bash
lldb build/MyApp
```

### lldb-dap
**개요:** LLDB를 Debug Adapter Protocol로 감싼 서버다. VS Code 연동용이다.
**작동 방식:** 에디터가 보낸 DAP JSON 메시지(launch, breakpoints)를 입력으로 받는다. 내부적으로 LLDB 엔진을 그대로 구동한다. 중단, 스택, 변수 상태를 DAP 응답으로 바꾼다. 표준 입출력으로 에디터와 통신한다.
```bash
lldb-dap --port 4711
```

### llvm-cov
**개요:** LLVM 소스 기반 커버리지 리포트 도구다. gcov보다 정확하다.
**작동 방식:** 계측 빌드에서 나온 프로파일을 합친 뒤 입력으로 받는다. 바이너리의 커버리지 맵과 카운터를 소스 줄에 매핑한다. 함수, 브랜치, 리전 단위로 집계한다. 텍스트, HTML 리포트로 출력한다.
```bash
clang -fprofile-instr-generate -fcoverage-mapping main.c -o main && ./main
xcrun llvm-profdata merge -sparse default.profraw -o default.profdata
xcrun llvm-cov report ./main -instr-profile=default.profdata
```

### llvm-cxxfilt
**개요:** c++filt의 LLVM 버전이다. 동작은 거의 같다.
**작동 방식:** 맹글 심볼을 인자나 표준 입력으로 받는다. LLVM Demangle 라이브러리로 심볼을 푼다. 복원된 함수 시그니처를 줄 단위로 출력한다. 스크립트에서 심볼 일괄 변환에 쓴다.
```bash
echo "_ZN3foo3barEi" | llvm-cxxfilt
nm -gU ./a.out | llvm-cxxfilt | head -10
```

### llvm-dwarfdump
**개요:** dwarfdump의 LLVM 버전이다. 출력이 더 자세하다.
**작동 방식:** Mach-O, dSYM, 오브젝트 파일을 입력으로 읽는다. DWARF 파서로 DIE, 줄 번호표, 가속 테이블을 푼다. 주소와 DIE 관계를 검증한다. 구조화 텍스트로 출력한다.
```bash
llvm-dwarfdump --debug-info build/MyApp.dSYM | head -40
llvm-dwarfdump --verify ./a.out
```

### llvm-nm
**개요:** LLVM 심볼 테이블 보기 도구다. nm과 호환된다.
**작동 방식:** Mach-O 파일 속 심볼 테이블 항목을 입력으로 읽는다. 심볼 값, 타입, 이름을 분류한다. C++ 심볼은 필요 시 디맹글한다. 정렬된 목록을 표준 출력으로 낸다.
```bash
llvm-nm -gU /bin/ls | head -20
llvm-nm --demangle ./a.out | head -20
```

### llvm-objdump
**개요:** Mach-O 역어셈블과 헤더 덤프용 LLVM 도구다.
**작동 방식:** 바이너리 파일을 입력으로 읽어 Mach-O 로드 커맨드를 파싱한다. 텍스트 섹션 바이트를 디스어셈블러로 푼다. 심볼과 DWARF 줄 정보를 어셈블리 옆에 붙인다. 헤더, 섹션, 역어셈블 결과를 텍스트로 출력한다.
```bash
llvm-objdump -d --syms ./a.out | head -40
llvm-objdump -macho -private-headers /bin/ls | head -30
```

### llvm-otool
**개요:** otool의 LLVM 재구현이다. Mach-O 보기에 쓴다.
**작동 방식:** Mach-O 파일을 입력으로 읽는다. 로드 커맨드와 섹션을 otool과 같은 형식으로 파싱한다. `-L`은 의존 dylib, `-l`은 로드 커맨드 전체를 푼다. 결과를 otool 호환 텍스트로 출력한다.
```bash
llvm-otool -L /bin/ls
llvm-otool -l ./a.out | head -40
```

### llvm-profdata
**개요:** LLVM 프로파일 데이터를 합치고 변환하는 도구다.
**작동 방식:** 실행 시 생긴 여러 프로파일 파일을 입력으로 읽는다. 카운터를 함수별로 합치고 인덱싱한다. 바이너리 프로파일로 출력한다. 이 파일이 llvm-cov 입력이 된다.
```bash
xcrun llvm-profdata merge -sparse *.profraw -o merged.profdata
xcrun llvm-profdata show merged.profdata | head -20
```

### llvm-readtapi
**개요:** `.tbd` 파일과 dylib ABI를 읽고 비교하는 도구다.
**작동 방식:** TBD 텍스트 파일이나 dylib를 입력으로 읽는다. export 심볼, 아키텍처, 플랫폼, 버전을 파싱한다. 두 TBD 사이 추가, 삭제 심볼을 대조한다. 검증 결과와 정규화된 TBD를 출력한다.
```bash
llvm-readtapi compare libFoo.tbd libFoo_new.tbd
```

### llvm-size
**개요:** 섹션별 크기를 보여주는 size 도구다.
**작동 방식:** Mach-O 파일을 입력으로 읽어 세그먼트와 섹션 헤더를 파싱한다. 영역 크기를 합산한다. Berkeley나 Darwin 포맷으로 정리한다. 바이너리 다이어트 전후 비교에 쓴다.
```bash
llvm-size -m ./a.out
llvm-size -l /bin/ls | head -20
```

### nm
**개요:** 바이너리 심볼 테이블을 보여주는 기본 도구다.
**작동 방식:** Mach-O 파일의 심볼 테이블을 입력으로 읽는다. 각 항목의 주소, 타입, 이름을 해석한다. 외부 심볼과 로컬 심볼을 구분한다. 목록으로 출력한다.
```bash
nm -gU ./a.out | head -20
nm -demangle ./a.out | head -20
```

### nm-classic
**개요:** 구형 nm 구현이다. 새 nm과 결과가 다를 때 대조용이다.
**작동 방식:** 같은 Mach-O 심볼 테이블을 입력으로 읽는다. 구형 파서 로직으로 심볼을 해석한다. 최신 포맷 일부는 다르게 표시될 수 있다. 텍스트 목록으로 출력한다.
```bash
nm-classic -gU ./a.out | head -20
```

### nmedit
**개요:** dylib 심볼 가시성을 고치는 도구다. export 목록 편집용이다.
**작동 방식:** dylib와 심볼 리스트 파일을 입력으로 받는다. 외부 정의, 미정의 심볼 플래그를 고친다. 수정된 바이너리를 제자리나 새 파일로 출력한다.
```bash
nmedit -s export_list.txt build/libFoo.dylib -o build/libFoo_stripped.dylib
```

### objdump
**개요:** GNU 스타일 오브젝트 덤프 도구다. macOS CLT에도 들어 있다.
**작동 방식:** Mach-O나 오브젝트 파일을 입력으로 읽는다. 섹션 헤더와 역어셈블을 푼다. 텍스트 리포트로 출력한다.
```bash
objdump -h ./a.out | head -20
objdump -d --syms ./a.out | head -40
```

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

### unwinddump
**개요:** 언와인드 정보를 덤프하는 도구다. 예외와 백트레이스 분석용이다.
**작동 방식:** Mach-O 파일 속 compact unwind 테이블과 DWARF CFI를 입력으로 읽는다. 함수 주소 범위별 언와인드 인코딩을 파싱한다. 주소별 언와인드 규칙 목록으로 출력한다.
```bash
unwinddump ./a.out | head -40
unwinddump -eh-frame /bin/ls | head -20
```

## 4. Swift 툴체인

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

## 5. Python 개발 도구

Xcode CLT에는 Python 3.9 계열 도구가 함께 들어 있다. 이 Python은 Xcode와 시스템 스크립트를 돌리기 위한 용도에 가깝다. 개발 공부를 하거나 프로젝트를 만든다면 python.org나 Homebrew, pyenv로 별도 설치를 권장한다.

### 2to3
**개요:** Python 2로 쓴 코드를 Python 3 문법으로 바꿔 주는 자동 변환기다.
**작동 방식:** 입력 파일을 읽고 파서로 문법 트리를 만든다. 그 다음 fixer 목록을 순서대로 적용한다. 각 fixer가 옛 패턴을 찾아 새 패턴으로 바꾼다. 결과는 화면에 보여 주거나 `-w`를 주면 원본에 덮어쓴다.
**예시:**
```bash
2to3 -w old_script.py
```

### 2to3-3.9
**개요:** `2to3`의 Python 3.9 고정판이다.
**작동 방식:** CLT 안의 Python 3.9 인터프리터를 기준으로 동작한다. 버전 없는 `2to3`는 기본값을 따라가지만 이 명령은 3.9용 변환 규칙에 고정된다.
**예시:**
```bash
2to3-3.9 -n -W --add-suffix=.bak legacy_tool.py
```

### pip3
**개요:** Python 패키지 설치기다.
**작동 방식:** CLT 기본 Python 3에 딸린 pip가 실행된다. `sys.path` 기준으로 설치 위치를 정하고 의존성을 함께 받아 버전을 맞춘다. CLT 내장 Python이라 가상 환경과 함께 쓰는 편이 안전하다.
**예시:**
```bash
pip3 install --user requests
```

### pip3.9
**개요:** Python 3.9 전용 pip다.
**작동 방식:** 반드시 Python 3.9 인터프리터와 한 쌍으로 움직인다. 설치 경로도 3.9 기준이다.
**예시:**
```bash
pip3.9 install --user flask
```

### pydoc3
**개요:** 모듈과 함수의 도움말을 보여 주는 문서 보기 도구다.
**작동 방식:** 모듈 이름을 받으면 `sys.path`에서 해당 모듈을 찾아 독스트링과 함수 목록을 뽑는다. 터미널에 페이지 형태로 출력한다.
**예시:**
```bash
pydoc3 os.path
```

### pydoc3.9
**개요:** Python 3.9 전용 문서 보기 도구다.
**작동 방식:** Python 3.9 기준으로 모듈을 찾고 문서를 만든다. 버전별 함수 차이를 확인할 때 도움이 된다.
**예시:**
```bash
pydoc3.9 pathlib
```

### python3
**개요:** Python 코드를 실행하는 본체다.
**작동 방식:** CLT에 들어 있는 Python 3을 불러온다. 스크립트를 주면 위에서 아래로 실행한다. `import`를 만나면 모듈을 찾는다. 인수가 없으면 대화형 프롬프트가 열린다.
**예시:**
```bash
python3 hello.py
```

### python3.9
**개요:** 버전을 3.9로 못박은 Python 실행 파일이다.
**작동 방식:** CLT 안의 Python 3.9를 직접 가리킨다. 기본값이 바뀌어도 3.9로 돈다.
**예시:**
```bash
python3.9 --version
python3.9 hello.py
```

## 6. 문법·파서·매크로·IDL

파서와 매크로는 코드를 만드는 코드다. lex가 문장을 토큰으로 자르고, yacc이 토큰을 문법에 맞춰 해석하고, cc가 결과를 실행 파일로 묶는다.

### bison
개요: GNU 파서 생성기. yacc과 호환되며 문법 파일(.y)에서 LALR 파서를 만든다.
작동 방식: .y 파일에 토큰 선언과 문법 규칙, 액션 코드를 적는다. bison을 실행하면 y.tab.c와 y.tab.h가 생긴다. 이 C 파일을 flex가 만든 렉서와 함께 cc로 컴파일한다.
```bash
bison -d calc.y && flex calc.l && cc -o calc y.tab.c lex.yy.c -ll
```

### bm4
개요: Berkeley m4 계열 매크로 전처리기.
작동 방식: 텍스트에 섞인 매크로 정의를 읽고 치환 규칙을 적용한다. 입력 파일을 스캔하면서 매크로를 확장한다. 결과가 새 텍스트로 출력된다.
```bash
bm4 define(`VERSION', `1.0')VERSION < input.txt > output.txt
```

### byacc
개요: Berkeley Yacc. 오리지널 yacc을 BSD 계열에서 다듬은 파서 생성기다.
작동 방식: .y 문법 파일을 입력으로 받아 LALR 파서 C 코드를 만든다. flex나 lex 출력과 링크해서 쓴다. 생성된 y.tab.c를 cc로 컴파일하면 완성된다.
```bash
byacc -d grammar.y && cc -o parser y.tab.c lex.yy.c
```

### flex
개요: 고속 렉서 생성기. 정규식으로 토큰 규칙을 적으면 스캐너 C 코드를 만든다.
작동 방식: .l 파일에 패턴과 액션을 적는다. flex를 돌리면 lex.yy.c가 생성된다. 이 파일은 입력 문자열을 토큰 단위로 잘라 파서로 넘긴다.
```bash
flex count.l && cc -o counter lex.yy.c -ll
```

### flex++
개요: flex의 C++ 래퍼. C++ 스캐너 클래스를 만들 때 쓴다.
작동 방식: 기본 동작은 flex와 같고 C++ 코드를 생성한다. FlexLexer 클래스를 상속받아 yylex를 메서드로 제공한다. 빌드할 때는 c++ 컴파일러로 링크한다.
```bash
flex++ scanner.l && c++ -o scanner lex.yy.cc -lfl
```

### gm4
개요: GNU m4. POSIX 표준 매크로 프로세서로 autoconf의 핵심 엔진이다.
작동 방식: m4 파일을 읽어 내장 매크로를 확장한다. 텍스트에 상관없이 순수 치환을 하므로 어떤 언어에도 쓸 수 있다. 재귀 확장과 인용 부호 변경으로 복잡한 템플릿을 만든다.
```bash
gm4 template.m4 > result.h
```

### gperf
개요: 완벽 해시 함수 생성기. 키워드 집합에 맞는 충돌 없는 해시 함수를 만든다.
작동 방식: 키워드 목록 파일을 입력으로 받는다. 각 키워드를 상수 시간에 찾을 수 있는 해시 함수와 테이블을 C 코드로 출력한다. 컴파일러의 예약어 검색에 쓴다.
```bash
gperf keywords.gperf > keywords_hash.c
```

### lex
개요: 오리지널 렉서 생성기. flex의 뿌리가 되는 유닉스 표준 도구다.
작동 방식: .l 파일의 정규식 규칙을 읽어 lex.yy.c를 만든다. flex보다 기능은 적지만 거의 모든 유닉스에 있다. yacc과 짝을 이뤄 쓴다.
```bash
lex tokens.l && yacc -d grammar.y && cc -o lang y.tab.c lex.yy.c -ll -ly
```

### m4
개요: 전통 유닉스 매크로 프로세서.
작동 방식: 입력 텍스트를 읽으며 매크로를 찾아 바꾼다. divert와 undivert로 출력을 버퍼에 나눠 담을 수 있다. 디버깅할 때는 `-d`로 확장 과정을 추적한다.
```bash
m4 -DDEBUG=1 config.m4 > config.h
```

### mig
개요: Mach Interface Generator. macOS 커널과 주고받는 IPC 인터페이스 코드를 만든다.
작동 방식: .defs 파일에 RPC 함수와 메시지 타입을 정의한다. mig를 실행하면 user 측 스텁과 server 측 스텁 C 코드가 나온다. user 코드는 메시지를 보내고, server 코드는 받은 메시지를 함수 호출로 푼다.
```bash
mig mach_test.defs -user mach_testUser.c -server mach_testServer.c -header mach_test.h
```

### rpcgen
개요: ONC RPC 스텁 생성기. 네트워크 함수 호출 코드를 자동으로 만든다.
작동 방식: .x 파일에 프로그램 번호와 함수 시그니처를 IDL로 적는다. rpcgen이 클라이언트 스텁, 서버 스텁, XDR 직렬화 코드, 헤더를 뽑아낸다. 클라이언트는 로컬 함수처럼 호출하고 실은 네트워크로 통신한다.
```bash
rpcgen -a -o calc calc.x
```

### unifdef
개요: 조건부 컴파일 제거기. `#if`, `#ifdef` 블록을 조건값에 따라 정리한다.
작동 방식: 소스 파일과 `-D`나 `-U`로 준 심볼 값을 입력으로 받는다. 참이 확정된 분기는 남기고 거짓 분기는 지운다. 판단이 안 되는 분기는 그대로 둔다.
```bash
unifdef -DOS_MAC=1 -DOS_LINUX=0 config.h > config_mac.h
```

### unifdefall
개요: unifdef의 일괄 처리용 스크립트. 여러 파일에 같은 조건을 한 번에 적용한다.
작동 방식: 내부적으로 unifdef를 반복 호출하는 래퍼다. 디렉터리를 돌며 조건 심볼 목록을 일괄 적용한다.
```bash
unifdefall -DOS_MAC=1 -DOS_LINUX=0 *.c *.h
```

### yacc
개요: 오리지널 유닉스 파서 생성기. bison과 byacc의 기준이 되는 도구다.
작동 방식: .y 문법에서 LALR 테이블과 y.tab.c를 만든다. lex가 토큰을 공급하고 yacc이 문법 트리를 조립한다. 마지막에 cc로 묶으면 작은 언어나 계산기가 완성된다. 전형적 파이프라인은 `lex → yacc → cc` 세 단계다.
```bash
yacc -d calc.y && lex calc.l && cc -o calc y.tab.c lex.yy.c -ly -ll
```

## 7. Apple 리소스·서명·문서·기타

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

# 6. 문법·파서·매크로·IDL

> 원문: `../CLT-tools-guide.md`에서 분리


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

# IPC-RPC

> 대분류: `08-파서-코드생성` / 통신 스텁 생성. Mach 메시지와 네트워크 호출

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


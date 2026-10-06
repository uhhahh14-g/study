# DWARF

> 대분류: `04-바이너리-분석` / DWARF 디버그 정보 덤프와 검증

### dwarfdump
**개요:** Mach-O나 dSYM 속 DWARF 디버그 정보를 덤프하는 고전 도구다.
**작동 방식:** 바이너리나 dSYM 경로를 입력으로 받는다. `__DWARF` 세그먼트와 DIE 트리를 파싱한다. 컴파일 유닛, 함수, 변수, 줄 번호표를 사람이 읽을 형태로 푼다. 특정 DIE나 주소만 골라 출력할 수도 있다.
```bash
dwarfdump --debug-info build/MyApp.dSYM | head -40
```

### llvm-dwarfdump
**개요:** dwarfdump의 LLVM 버전이다. 출력이 더 자세하다.
**작동 방식:** Mach-O, dSYM, 오브젝트 파일을 입력으로 읽는다. DWARF 파서로 DIE, 줄 번호표, 가속 테이블을 푼다. 주소와 DIE 관계를 검증한다. 구조화 텍스트로 출력한다.
```bash
llvm-dwarfdump --debug-info build/MyApp.dSYM | head -40
llvm-dwarfdump --verify ./a.out
```


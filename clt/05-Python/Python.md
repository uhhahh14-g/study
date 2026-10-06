# 5. Python 개발 도구

> 원문: `../CLT-tools-guide.md`에서 분리


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

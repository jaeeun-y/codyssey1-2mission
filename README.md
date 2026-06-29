
# 🛠️ Docker Agent Environment Setup & Monitoring Guide

`my-agent-test` 도커 컨테이너 환경에서 애플리케이션을 구동하기 위한 필수 조건을 정리한 가이드입니다.

---

## 1. 필수 환경변수 및 보안 키 설정 (Environment Variables)

애플리케이션 구동 전에 아래 환경변수들이 반드시 프로세스 세션에 등록되어야 합니다.

| 환경변수명 | 설정 값 / 경로 | 설명 |
| :--- | :--- | :--- |
| **`AGENT_HOME`** | `"$HOME/agent_home"` | 에이전트의 루트 디렉터리 경로 |
| **`AGENT_PORT`** | `15034` | 통신에 사용할 애플리케이션 고정 포트 |
| **`AGENT_UPLOAD_DIR`**| `"$AGENT_HOME/upload_files"` | 업로드 파일이 임시 저장되는 필수 디렉터리 (생성 필요) |
| **`AGENT_KEY_PATH`** | `"$AGENT_HOME/api_keys"` | `secret.key` 보안 파일이 위치하는 경로 (생성 필요) |
| **`AGENT_LOG_DIR`** | `"$AGENT_HOME/logs"` | 앱 실행 로그(`system.log`)가 기록될 디렉터리 (쓰기 권한 필요) |

  
### 🔑 필수 보안 키(Secret Key) 생성 명령어
시스템 검증에 사용되는 API 키 파일을 해당 경로에 사전 생성해야 합니다.


```bash
# 디렉터리 생성 및 키 텍스트 주입
mkdir -p $AGENT_HOME/api_keys
echo "agent_api_key_test" > $AGENT_HOME/api_keys/secret.key

```
  
---
  
## 2. 가상환경 및 도커 컨테이너 진입 (Docker Access)
  
권한이 제한된 `student` 유저로 컨테이너 내부 Bash 쉘에 진입하는 단계입니다.
  
```bash
# 1. 도커 컨테이너 기동
docker run -d --name my-agent-test ubuntu:22.04

# 2. student 계정으로 컨테이너 내부 진입
docker exec -it -u student my-agent-test /bin/bash

```
  
### 💡 `docker exec` 핵심 옵션 상세 설명
  
* **`-i` (`--interactive`):** 컨테이너와 상호작용할 수 있도록 표준 입력(stdin)을 열어두어 키보드 입력을 가능하게 합니다.
* **`-t` (`--tty`):** 가상 터미널을 할당하여, 리눅스 프롬프트(`student@...`)가 화면에 정상 출력되도록 합니다.
* **`-u` (`--user`):** 특정 유저 권한을 지정하여 접속합니다. (여기서는 `student` 계정)
  
---

## 3. 애플리케이션 제어 (Process Control)

리눅스 프로세스는 **처음 실행될 때의 환경변수만 기억**하므로, 환경변수를 수정했다면 반드시 기존 앱을 완전히 종료(`kill`)한 후 재실행해야 합니다.

### 🏃‍♂️ 애플리케이션 백그라운드 실행

```bash
./agent-leak-app-x86 > $AGENT_LOG_DIR/system.log 2>&1 &

```

### 🛑 기존 애플리케이션 강제 종료

```bash
kill -9 $(pgrep -f agent-leak-app-x86)

```

### 💡 프로세스 제어 옵션 상세 설명

* **`pgrep -f [프로세스명]`:** `-f` (Full command line) 옵션을 사용하여 프로세스 이름뿐만 아니라 전체 실행 경로를 매칭해 정확한 PID를 찾아냅니다.
* **`kill -9`:** 운영체제 레벨에서 예외 없이 즉각 프로세스를 강제 소멸시키는 최고 등급 시그널(`SIGKILL`)입니다.

---

## 🔍 4. 실시간 자원 모니터링 (Resource Monitoring)

장애 상황(OOM, CPU Spike, Deadlock)을 상시 관측하기 위해 5초 간격으로 자원 변화 추이를 누적 출력하는 무한 루프 스크립트입니다.

```bash
while true; do
  ps -o pid,%cpu,%mem,rss -p $(pgrep -f agent-leak-app-x86)
  sleep 5
  echo "-------------------"
done

```

### 💡 `ps` 모니터링 옵션 및 칼럼 상세 설명

* **`-p [PID]`:** 특정 PID를 타겟으로 지정하여 원하는 프로세스 정보만 필터링 조회합니다.
* **`-o [Format]`:** 기본 표준 출력을 무시하고, 사용자가 지정한 맞춤형 칼럼만 순서대로 배치합니다.

### 👀 ps 출력 칼럼 의미

* **`pid`**: 운영체제가 부여한 프로세스 고유 ID
* **`%cpu`**: 프로세스의 실시간 CPU 사용량 비율 (%)
* **`%mem`**: 전체 물리 RAM 대비 프로세스 점유 비율 (%)
* **`rss` (Resident Set Size)**: 프로세스가 가상 메모리가 아닌 **실제 물리 RAM 공간에 상주시키고 있는 실제 메모리 크기 (KB 단위)**

```

```

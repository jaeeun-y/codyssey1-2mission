
# Docker Agent Environment Setup & Monitoring Guide
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

  

```bash
# 디렉터리 생성 및 키 텍스트 주입
mkdir -p $AGENT_HOME/api_keys
echo "agent_api_key_test" > $AGENT_HOME/api_keys/secret.key

```
  
---
  
## 2. 가상환경 및 도커 컨테이너 진입 (Docker Access)


## 3. 애플리케이션 제어 (Process Control)

리눅스 프로세스는 **처음 실행될 때의 환경변수만 기억**하므로, 환경변수를 수정했다면 반드시 기존 앱을 완전히 종료(`kill`)한 후 재실행해야 합니다.

### 애플리케이션 백그라운드 실행

```bash
./agent-leak-app-arm64 > $AGENT_LOG_DIR/system.log 2>&1 &

```

### 기존 애플리케이션 강제 종료

```bash
kill -9 $(pgrep -f agent-leak-app-arm64)

```

---

## 4. 실시간 자원 모니터링 (Resource Monitoring)
 

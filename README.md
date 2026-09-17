
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


```

sudo sh -c "cat << 'EOF' >> /home/agent-admin/.bashrc

export AGENT_HOME=/home/agent-admin/agent-app
export AGENT_PORT=15034
export AGENT_UPLOAD_DIR=\$AGENT_HOME/upload_files
export AGENT_KEY_PATH=\$AGENT_HOME/api_keys/secret.key
export AGENT_LOG_DIR=/var/log/agent-app
EOF"

```

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

nohup $AGENT_HOME/bin/agent-leak-app-arm64 > $AGENT_LOG_DIR/app_boot.log 2>&1 &

```

```
cat /var/log/agent-app/app_boot.log
```

### 기존 애플리케이션 강제 종료

```bash
kill -9 $(pgrep -f agent-leak-app-arm64)

```

---

## 4. 실시간 자원 모니터링 (Resource Monitoring)

```
cat << 'EOF' > monitor.sh
#!/bin/bash

# ── 설정 ─────────────────────────────────────────

# 비-root 계정 권한 문제 방지를 위해 홈 디렉터리 하위 경로 사용
LOG_DIR="$HOME/agent_logs"
LOG_FILE="$LOG_DIR/monitor.log"
APP_NAME="agent-leak-app-x86"  # x86 바이너리로 수정
APP_PORT=15034

CPU_THRESHOLD=80
MEM_THRESHOLD=80
DISK_THRESHOLD=80

MAX_LOG_SIZE=$((10 * 1024 * 1024))   # 10MB
MAX_BACKUPS=10

# ── 사전 준비 ─────────────────────────────────────

mkdir -p "$LOG_DIR"
touch "$LOG_FILE" 2>/dev/null

log() {
   echo "$1" | tee -a "$LOG_FILE"
}

fail() {
   echo "[ERROR] $1" | tee -a "$LOG_FILE"
}

# ── 검사 함수 ─────────────────────────────────────

check_process() {
   local pid
   pid=$(pgrep -f "$APP_NAME" | head -n 1)
   if [ -z "$pid" ]; then
       fail "Process $APP_NAME is not running"
       return 1
   fi
   echo "$pid"
}

get_cpu_usage() {
   top -bn1 | awk '/Cpu\(s\)/ {printf "%d", $2 + $4}'
}

get_mem_usage() {
   free | awk '/Mem:/ {printf "%d", $3/$2 * 100}'
}

get_disk_usage() {
   df / | awk 'NR==2 {print $5}' | tr -d '%'
}

check_threshold() {
   local label="$1" value="$2" threshold="$3"
   if [ -n "$value" ] && [ "$value" -gt "$threshold" ]; then
       log "[ALERT] ${label} 경고: ${value}% (임계치 ${threshold}% 초과)"
   fi
}

# ── 실시간 지속 모니터링 루프 ───────────────────────

main() {
   echo "=================================================="
   echo "  Agent Leak App 실시간 모니터링 시작 (Ctrl+C 종료)"
   echo "=================================================="

   while true; do
       local pid cpu mem disk timestamp
       
       pid=$(check_process)
       
       if [ -n "$pid" ]; then
           cpu=$(get_cpu_usage)
           mem=$(get_mem_usage)
           disk=$(get_disk_usage)

           timestamp=$(date "+%Y-%m-%d %H:%M:%S")
           log "[$timestamp] PID:${pid} | CPU:${cpu}% | MEM:${mem}% | DISK:${disk}%"

           check_threshold "CPU" "$cpu" "$CPU_THRESHOLD"
           check_threshold "MEM" "$mem" "$MEM_THRESHOLD"
           check_threshold "DISK" "$disk" "$DISK_THRESHOLD"
       fi

       sleep 2
       echo "--------------------------------------------------"
   done
}

main
EOF

chmod +x monitor.sh
```

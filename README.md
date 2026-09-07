
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

LOG_DIR="/var/log/agent-app"
LOG_FILE="/var/log/agent-app/monitor.log"
APP_NAME="agent-leak-app-arm64"
APP_PORT=15034

CPU_THRESHOLD=80
MEM_THRESHOLD=80
DISK_THRESHOLD=80

MAX_LOG_SIZE=$((10 * 1024 * 1024))   # 10MB
MAX_BACKUPS=10

# ── 로그 함수 ─────────────────────────────────────

log() {
    echo "$1" >> "$LOG_FILE"
}
 
fail() {
    echo "[ERROR] $1"
    log "[ERROR] $1"
    exit 1
}
 
# ── 사전 준비 ─────────────────────────────────────

mkdir -p "$LOG_DIR"
touch "$LOG_FILE"
 
# ── 1. 프로세스 실행 확인 ──────────────────────────

check_process() {
    if ! pgrep -f "$APP_NAME" > /dev/null; then
        fail "Process $APP_NAME is not running"
    fi
    pgrep -f "$APP_NAME" | head -n 1
}
 
# ── 2. 포트 리스닝 확인 ────────────────────────────

check_port() {
    if ! ss -tln | grep -q ":${APP_PORT} "; then
        fail "Port $APP_PORT is not listening"
    fi
}
 
# ── 3. 방화벽 활성 확인 (경고만, 종료하지 않음) ──────

check_firewall() {
    if command -v ufw > /dev/null 2>&1; then
        if command -v systemctl > /dev/null 2>&1; then
            if ! systemctl is-active --quiet ufw; then
                log "[WARNING] UFW firewall is not active"
            fi
}
 
# ── 4. 리소스 사용량 측정 ──────────────────────────

get_cpu_usage() {
    top -bn1 | awk '/Cpu\(s\)/ {printf "%d", $2 + $4}' # $2|user 사용률, $4|system 사용률
}
 
get_mem_usage() {
    free | awk '/Mem:/ {printf "%d", $3/$2 * 100}' # $3|used, $2|total (사용량/전체 비율)
}
 
get_disk_usage() {
    df / | awk 'NR==2 {print $5}' | tr -d '%' # df의 Use% 컬럼
}
 
# ── 5. 임계값 초과 경고 ────────────────────────────

check_threshold() {
    local label="$1" value="$2" threshold="$3"
    if [ "$value" -gt "$threshold" ]; then
        log "${label} 경고: ${value}%"
    fi
}
 
# ── 6. 로그 로테이션 (10MB 초과 시 최대 10개 백업) ───

rotate_log_if_needed() {
    [ -f "$LOG_FILE" ] || return
    local size
    size=$(stat -c%s "$LOG_FILE")
    [ "$size" -gt "$MAX_LOG_SIZE" ] || return
 
    rm -f "${LOG_FILE}.${MAX_BACKUPS}"
    for ((i = MAX_BACKUPS - 1; i >= 1; i--)); do
        [ -f "${LOG_FILE}.${i}" ] && mv "${LOG_FILE}.${i}" "${LOG_FILE}.$((i + 1))"
    done
    mv "$LOG_FILE" "${LOG_FILE}.1"
    touch "$LOG_FILE"
}
 
# ── 메인 실행 흐름 ──────────────────────────────────

main() {
    local pid cpu mem disk timestamp
 
    pid=$(check_process)
    check_port
    check_firewall
 
    cpu=$(get_cpu_usage)
    mem=$(get_mem_usage)
    disk=$(get_disk_usage)
 
    check_threshold "CPU" "$cpu" "$CPU_THRESHOLD"
    check_threshold "MEM" "$mem" "$MEM_THRESHOLD"
    check_threshold "DISK" "$disk" "$DISK_THRESHOLD"
 
    timestamp=$(date "+%Y-%m-%d %H:%M:%S")
    log "[$timestamp] PID:${pid} CPU:${cpu}% MEM:${mem}% DISK_USED:${disk}%"
 
    rotate_log_if_needed
}
 
main
exit 0
```

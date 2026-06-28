# 핵심 정리

monitor.sh 결과(메모리 상승 수치)  
종료 직전/직후 실행 로그("Memory limit exceeded…", "SELF-TERMINATED…" 등)  
MEMORY_LIMIT 변경 전후 비교(최소 2회 실행)  

  
monitor.sh를 활용하여, 대상 프로세스(agent-leak-app)의 물리 메모리 사용량이 시간 경과에 따라 증가하는 패턴을 관측한다.  
프로세스가 예고 없이 중단되었을 때 프로그램 실행 로그를 분석하여 메모리 임계치 초과로 인해 애플리케이션의 메모리 보호 정책(MemoryGuard)에 따라 강제 종료되었음을 나타내는 핵심 로그를 식별한다.  
환경변수(MEMORY_LIMIT)를 조정하여 프로그램이 더 오래 생존하는 것을 확인하고, 그 결과를 리포트에 Before & After로 기록한다.  

---

## 📁 발생 현상
장애가 어떻게 관측되었는지 서술

```bash

<<export MEMORY_LIMIT=50>>
# 메모리 보호 임계치 초과 시, MemoryGuard 작동


12:50:51,455 [INFO] [MemoryWorker] Current Heap: 25MB  
12:50:54,511 [INFO] [MemoryWorker] Current Heap: 50MB  

12:50:54,511 [CRITICAL] [MemoryGuard] Memory limit exceeded (50MB >= 50MB) / (Recommend Over 256MB)  
12:50:54,511 [CRITICAL] [MemoryGuard] Self-terminating process 1014 to prevent system instability.  
>>> [SYSTEM] SELF-TERMINATED (Memory Limit Exceeded) <<<  
Killed

```

---

### 📁 재현 경로 및 증거
로그/명령어 출력/스크린 샷 등 객관적 증거

monitor.sh 관제 로그 데이터

```

while true
do
    ps -o pid,%cpu,%mem,rss -p $(pgrep -f agent-leak-app-x86)
#pgrep으로 pid번호를 알아낸 뒤 ps가 pid를 이용해서 CPU/MEM점유율, 실제 사용 메모리량 RSS를 보여줌.
    sleep 5
    echo "-------------------"
end

```

---

### 📁 근본 원인
장애의 기술적 원인 분석

---

### 📁 조치 내용
환경변수 조정 등 임시 조치와 그 결과

<<export MEMORY_LIMIT=256>>

```
12:54:24,819 [INFO] [MemoryWorker] Current Heap: 25MB
12:54:27,876 [INFO] [MemoryWorker] Current Heap: 50MB
12:54:30,933 [INFO] [MemoryWorker] Current Heap: 75MB
.
.
.
12:54:52,333 [INFO] [MemoryWorker] Current Heap: 250MB  
12:54:55,389 [INFO] [MemoryWorker] Current Heap: 275MB  
    
12:54:55,389 [CRITICAL] [MemoryGuard] Memory limit exceeded (275MB >= 256MB) / (Recommend Over 256MB)  
12:54:55,389 [CRITICAL] [MemoryGuard] Self-terminating process 1018 to prevent system instability.  
  
>>> [SYSTEM] SELF-TERMINATED (Memory Limit Exceeded) <<<  
Killed
```

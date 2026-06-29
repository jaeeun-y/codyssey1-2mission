
### 발생 현상
장애가 어떻게 관측되었는지 서술

tail -f $AGENT_LOG_DIR/system.log

<img width="869" height="311" alt="Screenshot 2026-06-29 at 7 41 30 PM" src="https://github.com/user-attachments/assets/f93b9653-ae08-446c-a667-14c2dcb86642" />

<img width="530" height="134" alt="Screenshot 2026-06-29 at 7 44 27 PM" src="https://github.com/user-attachments/assets/f8c43039-d8e5-4eec-a7f3-7ecb21c4814b" />


### 근본 원인
장애의 기술적 원인 분석

### 조치 내용
환경변수 조정 등 임시 조치와 그 결과


<<1>>
export MEMORY_LIMIT=500
export CPU_MAX_OCCUPY=50
export MULTI_THREAD_ENABLE=true
  
export DEADLOCK_TRIGGER=true  # ★ 이 값이 데드락 방지 로직을 활성화


<<2>>
export MULTI_THREAD_ENABLE=false # 싱글 스레드 모드로 전환

```

 PID %CPU %MEM   RSS
     47  0.2  0.0  2144
     48  1.9  1.8 300168
-------------------
    PID %CPU %MEM   RSS
     47  0.2  0.0  2144
     48  2.1  2.1 351376
-------------------
    PID %CPU %MEM   RSS
     47  0.2  0.0  2144
     48  2.1  2.4 402584
-------------------
    PID %CPU %MEM   RSS
     47  0.1  0.0  2048
     48  2.1  2.5 421324
-------------------
    PID %CPU %MEM   RSS
     47  0.1  0.0  2048
     48  2.1  2.8 472532
-------------------
    PID %CPU %MEM   RSS
     47  0.1  0.0  2048
     48  2.1  0.0 12476
-------------------
    PID %CPU %MEM   RSS
     47  0.1  0.0  2048
     48  2.1  0.0 12476
^C
[3] + Stopped

```

# codyssey1-2mission


AGENT_HOME	필수 환경변수 설정  
```
% export AGENT_HOME="HOME/agent_home"
```

AGENT_PORT	15034 (고정)  
```
% export AGENT_PORT=15034 # 통신에 사용할 고정 포트
```

AGENT_UPLOAD_DIR	$AGENT_HOME/upload_files (디렉터리 존재 필수)  
```
% export AGENT_UPLOAD_DIR="AGENT_HOME/upload_files" # 업로드 파일이 임시 저장되는 필수 디렉터리
```

AGENT_KEY_PATH	$AGENT_HOME/api_keys (경로 존재 필수)  
```
% export AGENT_KEY_PATH="AGENT_HOME/api_keys"  # secret.key 파일이 위치하는 보안 경로
```

AGENT_LOG_DIR	로그 디렉터리 (존재 + 쓰기 권한)  
```
% export AGENT_LOG_DIR="AGENT_HOME/logs" 앱 실행 로그가 기록될 디렉터리
```

MEMORY_LIMIT	정수, 50 ~ 512 범위 (단위: MB)  
```
export MEMORY_LIMIT=128
```

CPU_MAX_OCCUPY	정수, 10 ~ 100 범위 (단위: %)  
```
export CPU_MAX_OCCUPY=60
```

MULTI_THREAD_ENABLE	true/false (1/0, yes/no 허용)  
```
export MULTI_THREAD_ENABLE=true
```

secret.key 파일	$AGENT_HOME/api_keys/secret.key 존재, 내용: agent_api_key_test  
```
% echo "agent_api_key_test" > AGENT_HOME/api_keys/secret.key  # 시스템 검증에 사용되는 필수 API 키 텍스트 생성
```

네트워크	0.0.0.0:15034 바인딩 가능


---



```
% mkdir -p AGENT_HOME/upload_files  
% mkdir -p AGENT_HOME/api_keys  
% mkdir -p AGENT_HOME/logs   
```


```
 % chmod +x monitor.sh
 % ./monitor.sh &
[1] 13773
 % =========================================
 관제 스크립트가 시작되었습니다. (2초 주기)
 로그 저장 위치: /logs/monitor.log
```

# codyssey1-2mission



실행 계정	root가 아닌 일반 사용자  
AGENT_HOME	필수 환경변수 설정  
AGENT_PORT	15034 (고정)  
AGENT_UPLOAD_DIR	$AGENT_HOME/upload_files (디렉터리 존재 필수)  
AGENT_KEY_PATH	$AGENT_HOME/api_keys (경로 존재 필수)  
AGENT_LOG_DIR	로그 디렉터리 (존재 + 쓰기 권한)  
MEMORY_LIMIT	정수, 50~512 범위 (단위: MB)  
CPU_MAX_OCCUPY	정수, 10~100 범위 (단위: %)  
MULTI_THREAD_ENABLE	true/false (1/0, yes/no 허용)  
secret.key 파일	$AGENT_HOME/api_keys/secret.key 존재, 내용: agent_api_key_test  
네트워크	0.0.0.0:15034 바인딩 가능  


## AGENT_HOME	필수 환경변수 설정   
= export AGENT_HOME="$HOME/agent_home"

  
  
## AGENT_PORT	15034 (고정)   
= export AGENT_PORT=15034   
# 통신에 사용할 고정 포트

  
  
## AGENT_UPLOAD_DIR	$AGENT_HOME/upload_files (디렉터리 존재 필수)  
= export AGENT_UPLOAD_DIR="$AGENT_HOME/upload_files"   
# 업로드 파일이 임시 저장되는 필수 디렉터리

  
  
## AGENT_KEY_PATH	$AGENT_HOME/api_keys (경로 존재 필수)  
= export AGENT_KEY_PATH="$AGENT_HOME/api_keys"    
# secret.key 파일이 위치하는 보안 경로

  
  
## AGENT_LOG_DIR	로그 디렉터리 (존재 + 쓰기 권한)  
= export AGENT_LOG_DIR="$AGENT_HOME/logs"   
# 앱 실행 로그가 기록될 디렉터리
   

  
## secret.key 파일	$AGENT_HOME/api_keys/secret.key 존재, 내용: agent_api_key_test  
= echo "agent_api_key_test" > $AGENT_HOME/api_keys/secret.key    
# 시스템 검증에 사용되는 필수 API 키 텍스트 생성





---


  
```
#사용한 가상환경

docker start my-agent-test

#권한이 제한된 student 유저로 컨테이너 내부 bash쉘에 진입
docker exec -it -u student my-agent-test /bin/bash

```
-i: 컨테이너와 상호작용할 수 있도록 표준 입력(stdin, 키보드입력)을 열어둠.
-t: 가상 터미널을 할당. 리눅스 프롬포트( 'student@...')가 화면에 출력


```
#무한 루프 모니터링 명령어
  
while true;
 do
  ps -o pid,%cpu,%mem,rss -p $(pgrep -f agent-leak-app-x86);
  sleep 5; echo "-------------------";
 done

```
-p [pid]: 특정 pid를 타겟으로 정보를 조회.
-o: 기본 표준 출력 무시, 사용자가 보고 싶은 칼럼만 순서대로 배치.


```
#앱 실행
  
./agent-leak-app-x86 > $AGENT_LOG_DIR/system.log 2>&1 &

```
  
```
#리눅스 프로세스는 처음 실행될 때의 환경변수를 기억해서 환경변수 변경 시 기존 앱 종료 필수
  
kill -9 $(pgrep -f agent-leak-app-x86

```
-pgrep -f [프로세스명]: -f(full command line) 옵션을 사용하여 이름, 실행 경로를 매치해 올바른 PID를 찾아냄.
-9: 시스템 레벨에서 즉각 프로세스 강제 소멸

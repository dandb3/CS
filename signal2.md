signal 관련 함수들 2탄
* kill 함수
	```C
	#include <signal.h>
	
	int	kill(pid_t pid, int sig);
	```
	* sig 시그널을 보내는 함수.
	* pid > 0 : sig 시그널을 pid로 보낸다.
	* pid == 0 : 현재 프로세스가 속한 프로세스 그룹의 모든 프로세스에게 sig 시그널을 보낸다.
	* pid == -1 : 1번 프로세스(init)를 제외한, 현재 프로세스가 시그널을 보낼 권한이 있는 모든 프로세스에 sig 시그널을 보낸다. (자신 제외)
	* pid < -1 : pid의 절댓값을 가지는 프로세스 그룹의 모든 프로세스에게 sig 시그널을 보낸다.
	* sig == 0 : 시그널이 보내지지는 않지만, 존재와 권한 체크는 수행된다. 호출자가 시그널을 보낼 수 있게 허용된 process ID나 process group ID의 존재 여부를 체크하는데에 사용된다.
	* 성공시 : 0리턴, 실패시 : -1리턴, errno값이 세팅된다.
* raise 함수
	```C
	#include <signal.h>
	
	int	raise(int sig);
	```
	* 자기 자신에게 sig 시그널을 보낸다.
	* 성공 : 0 리턴, 실패 : nonzero 리턴
* getpid, getppid 함수
	```C
	#include <unistd.h>
	
	pid_t	getpid(void);
	pid_t	getppid(void);
	```
	* getpid
		* 호출 프로세스의 PID값을 리턴해준다.
		* (unique한 temporary filename을 만들기 위해 보통 사용된다.)
	* getppid
		* 호출 프로세스의 부모 프로세스의 PID값을 리턴해준다.
		* fork를 통해 생성된 경우는 fork를 호출한 프로세스의 ID가 리턴
		* fork를 호출한 프로세스가 terminated라면, 바뀐 부모 프로세스의 ID가 리턴된다. (init이나 "subreaper" 프로세스)
		* subreaper 프로세스???
			* 지금까지는 자식 프로세스가 종료되기 전에 부모 프로세스가 종료되면 고아 프로세스가 되어 init 프로세스가 그 부모의 역할을 해 준다고 알고 있었다.
			* 하지만 prctl 함수의 PR_SET_CHILD_SUBREAPER 를 이용하면 init이 아닌 고아 프로세스의 가장 가까운 살아있는 조상이 subreaper가 되어 init 함수가 했었던 역할을 수행한다.
			* 계층 구조를 가지고 있는 프로그램의 경우 사용하면 좋다.
* sleep 함수
	```C
	#include <unistd.h>
	
	unsigned int	sleep(unsigned int seconds);
	```
	* sleep 함수는 실제 seconds 
* pause 함수
	```C
	#include <unistd.h>
	
	int	pause(void);
	```
	* pause 함수는 프로세스를 terminate 시키거나 signal-catching function을 호출하는 시그널을 받기 전까지 호출 프로세스(혹은 스레드)를 sleep 시킨다.
	* signal을 받아서 signal-catching function이 작동하고 리턴했을 경우에만 pause 함수가 리턴된다.
	* 리턴 값 : -1, errno : EINTR(signal was caught and the signal-catching function returned)

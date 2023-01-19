1. 시그널 처리 관련 함수들
	* signal
	  ```C
	  #include <signal.h>
	  void (*signal(int signum, void (*handler)(int)))(int)
	  // int를 인자로 받는 void 함수를 리턴한다.
	  ```
	  int signum : 시그널 번호
	  void (*handler)(int) 시그널을 처리할 핸들
	  반환 : void *()(int) 이전에 설정된 시그널 핸들러
		* 시그널 처리의 3가지 방법 -> signal 함수의 2번째 인자에 들어갈 부분
			1. 기존 방법으로 처리 (SIG_DFL)
			2. 무시 (SIG_IGN)
			3. 프로그램에서 직접 처리 (함수 이름)
		* 예시
	  ```C
	  #include <stdio.h>
	  #include <unistd.h>
	  #include <signal.h>
	  
	  void (*old_fun)( int);
	  
	  void sigint_handler( int signo)
	  {
   	  	printf( "Ctrl-C 키를 눌루셨죠!!\n");
   	  	printf( "또 누르시면 종료됩니다.\n");
   	  	signal( SIGINT, old_fun);   // 또는 signal( SIGINT, SIG_DFL);
	  }
	  
	  int main( void)
	  {
	     old_fun = signal( SIGINT, sigint_handler);
	     while(1 ){
	        printf( "badayak.com\n");
	        sleep( 1);
	     }
	  }
	  ```
	  	* 보면 알겠지만 처음 old_fun = signal(SIGINT, sigint_handler); 부분에서는 시그널을 sigint_handler을 이용해서 처리하겠다 라고 선언하면서 이전 처리 방법을 old_fun에 대입한다.
	  	* 그래서 그 이후에 SIGINT가 발생하면 sigint_handler가 실행이 되고, 그 내부에 있는 signal(SIGINT, old_fun);(signal(SIGINT, SIG_DFL)과 동일) 함수가 실행이 되어 이후에 시그널에 대해서는 기존의 방법이 실행되는 것이다.
	* sigfillset, sigemptyset
		```C
		int sigfillset(sigset_t *set);
		int sigemptyset(sigset_t *set);
		```
		* 신호를 간편하게 다루기 위해서 신호를 집합으로 표시하게 되었다.
		* sigset_t : 신호들의 집합
		* sigfillset : sigset_t라는 집합에서 모든 시그널을 추가
		* sigemptyset : sigset_t라는 집합에서 시그널을 모두 지워줌
		* 성공 시 0, 실패 시 -1 반환
	* sigaddset, sigdelset
		```C
		int sigaddset(sigset_t *set, int signum);
		int sigdelset(sigset_t *set, int signum);
		```
		* 이름만 보면 알 수 있듯
		* sigaddset : set에 signum 추가
		* sigdelset : set에 signum 제거
		* 성공 시 0, 실패 시 -1 반환
	* sigprocmask
		```C
		int sigprocmask(int how, const sigset_t *set, sigset_t oldset);
		```
		* 시그널을 블록시킬지 말지를 결정
		* how : 어떻게 시그널을 제어할지 말해준다. 3가지 동작이 있음 : 
			1. SIG_BLOCK : 기존에 블록된 시그널이 있다면 두 번째 인자는 set의 시그널을 추가하라는 의미
			2. SIG_UNBLOCK : 기존의 블록된 시그널에서 set의 시그널을 제거
			3. SIG_SETMASK : 기존의 블록된 시그널을 전부 제거시키고 새로운 set의 시그널들을 블록시킴
		* set : how가 동작하는 방식에 따라 기존의 block된 시그널에 대해 set을 추가시킬지, 제거시킬지, 아니면 전부 set으로 설정할지를 의미. -> set : 이번에 설정할 시그널
		* oldset : 이 전에 블록된 시그널들을 저장
3. 

1. 시그널 처리 관련 함수들
	* signal
	  ```C
	  #include <signal.h>
	  void (*signal(int signum, void (*handler)(int)))(int)
	  // int를 인자로 받는 void 함수를 리턴한다.
	  ```
	  int signum : 시그널 번호
	  void (\*handler)(int) 시그널을 처리할 핸들
	  반환 : void \*()(int) 이전에 설정된 시그널 핸들러
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
	* sigaction 함수
		```C
		#include <signal.h>
		int	sigaction(int signo, const struct sigaction *act, struct sigaction *oact);
		```
		* signal 함수가 불안정적이기 때문에 이를 대체하기 위해 나온 함수이다.
		* 만약 ```act != NULL```이라면 signum번호를 가지는 시그널에 대해서 act 함수가 설치된다.
		* 만약 ```oact != NULL```이라면 이전의 액션은 oact에 저장된다.
		* 성공 시 0, 실패하면 -1 리턴
	* sigaction 구조체
		```C
		struct sigaction
		{
			void		(*sa_handler)(int); // 시그널을 처리하기 위한 핸들러.
											// SIG_DFL, SIG_IGN 또는 핸들러 함수
			void		(*sa_sigaction)(int, siginfo_t *, void *); // 밑의 sa_flags가 SA_SIGINFO일 때
																   // sa_handler 대신에 동작하는 핸들러
			sigset_t	sa_mask; // 시그널을 처리하는 동안 블록화할 시그널 집합의 마스크
			int			sa_flags; // 아래 설명 참고
			void		(*sa_restorer)(void); // 사용하면 안 된다.
		}
		```
		* 몇몇 architecture의 경우 sa_handler와 sa_sigaction 둘이 union으로 묶여있는 경우도 존재하므로 둘 중 하나만 정의해서 사용하는 것이 좋다.
		  ```C
		  union {
		  	void	(*sa_handler)(int);
		  	void	(*sa_sigaction)(int, siginfo_t *, void *);
		  }			_n;
		  ```
		  의 꼴로 사용되는 경우가 있다.
		* 
		
	* 시그널 집합
		* 시그널 집합은 시그널을 비트 마스크로 표현함.
		* 시그널 하나가 비트 하나를 가리킨다. (1이면 시그널이 설정된 것이고, 0이면 시그널이 설정되지 않음)
		* 시그널 집합 : sigset_t 구조체
			```C
			typedef struct {
				unsigned int __sigbits[4];
			}	sigset_t;
			```
	* sigfillset, sigemptyset
		```C
		int sigfillset(sigset_t *set);
		int sigemptyset(sigset_t *set);
		```
		* 신호를 간편하게 다루기 위해서 신호를 집합으로 표시하게 되었다.
		* set : 시그널의 집합
		* sigfillset : set에서 모든 시그널을 추가 (모든 비트를 1로 바꿔줌)
		* sigemptyset : set에서 시그널을 모두 지워줌 (모든 비트를 0으로 바꿔줌)
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
	* sigismember
		```C
		#include <signal.h>
		
		int	sigismember(sigset_t *set, int signo);
		```
		* signo로 지정한 시그널이 set에 포함되어 있으면 1을, 포함되어 있지 않으면 0을 리턴
	* sigprocmask
		```C
		int sigprocmask(int how, const sigset_t *restrict set, sigset_t *restrict oset);
		// restrict : 프로그래머에 의해 의도된 포인터 선언방법.
		// 특정 메모리 영역에 접근할 수 있는 포인터가 단 하나임을 보장하는 키워드
		// 컴파일러에게 미리 알려주어 더 나은 최적화를 하게 해준다.
		```
		* 시그널 집합을 사용하는 이유가 이런 sigprocmask 같은 함수를 사용하여 편리하게 관리하기 위해서이다.
		* 시그널을 블록시킬지 말지를 결정 (예외 : SIGKILL, SIGSTOP은 블록이 안됨, 얘네들은 handler도 설정 불가)
		* how : 어떻게 시그널을 제어할지 말해준다. 3가지 동작이 있음 : 
			1. SIG_BLOCK : set에 지정한 시그널 집합을 시그널 마스크에 추가한다.
			2. SIG_UNBLOCK : set에 지정한 시그널 집합을 시그널 마스크에서 제거한다.
			3. SIG_SETMASK : set에 지정한 시그널 집합으로 현재 시그널 마스크를 대체한다.
		* set : 블록하거나 해제할 시그널 집합. NULL이라면 현재 블록된 시그널들을 oset에 전달하고 끝난다.
		* oset : NULL이 아니라면 바꾸기 전에 블록된 시그널들을 저장
		* 시그널이 블록되고 나면 해당 시그널은 전달되지 않고, 블록이 풀리게 되면 그제서야 프로세스에 전달된다.
3. 

1. Zombie Process?
실행이 종료되었지만 아직 삭제되지 않은 프로세스.
보통 프로세스는 exit 시스템 함수를 호출해서 프로세스를 종료시키고, 자신의 모든 자원을 해제하게 된다.
하지만 프로세스의 exit status와 PID는 여전히 커널의 task struct에 유지된다.
->이유: 부모 프로세스는 wait 시스템 함수를 통해 자식 프로세스의 종료 상태에 대한 정보를 가져올 수 있다.

-커널에서 사용하고 있는 task_struct 구조체
```C
struct task_struct 
{
    /*
     * offsets of these are hardcoded elsewhere - touch with care
     */
    volatile long state;    /* -1 unrunnable, 0 runnable, >0 stopped */
    unsigned long flags;    /* per process flags, defined below */
    int sigpending;
    mm_segment_t addr_limit;    /* thread address space:
                        0-0xBFFFFFFF for user-thead
                        0-0xFFFFFFFF for kernel-thread
                     */
    ....
    long counter;
    long nice;

    int exit_code, exit_signal;

	struct task_struct *next_task, *prev_task;
    ....
    pid_t pid;
    pid_t session;
}
```
프로세스 별로 task_struct가 존재하며, struct task_struct *next_task, *prev_task;를 통해 이전 프로세스, 이후 프로세스를 접근할 수 있다.(doubly linked list)
int exit_code, pid_t pid 멤버 변수를 통해 부모 프로세스는 자식 프로세스의 종료값을 얻어올 수 있다.

부모 프로세스는 wait 시스템 함수를 통해 자식 프로세스의 종료 상태를 얻어올 수 있다고 하였는데, 만약 wait 함수를 사용하지 않는다면 task_struct 구조체가 메모리에 그대로 남아있기 때문에 메모리의 낭비가 발생하게 된다.
또한 PID의 값 또한 회수되지 않았기에 다른 프로세스의 실행에 방해가 될 수도 있다.(PID값은 한정되어있기 때문)

2. 더 자세하게?
task_struct는 doubly linked list를 이루고 있다.
그러므로 회수되지 않은 자식 프로세스는 커널의 자원을 소모하게 되고, 커널이 가질 수 있는 task_struct의 list의 크기는 제한되어 있으므로 시스템 성능에 영향을 끼치게 된다.(너무 많은 PID의 할당으로 다음 프로세스 생성 불가능 문제인듯?)
또한 list의 길이가 더 길어지게 되므로 리스트를 순환하는데에 시간이 오래 걸리게 되어 프로세스 스케줄링에도 악영향을 끼치게 된다.

실제로 /proc/PID 디렉토리에 가서 프로세스 상태를 확인해 보면 모든 proc파일의 크기가 0인데, 프로세스 이미지 자체는 남아있지 않다는 것을 알 수 있다. 또한 모든 파일(STDIN, STDOUT, STDERR, 다른 fd 등) 역시 닫혀있음을 확인할 수 있다.


## 3 Project 2: User Programs

 이제 user program을 실행할 수 있는 system 부분에 대한 작업을 시작하자. 기본 코드는 이미 user program을 실행할 수 있지만, I/O interactivity가 불가능함. 이 플젝에서는 system call을 써서 interact하도록 해야해.

 `userprog` directory에서 작업할건데, Pintos 의 거의 모든 부분과 interact할거야. [잘 짜란 얘기겠지]

 이건 플젝1코드 필요없음. alarm clock은 플젝3이랑 4에서 필요할 수 있지만 필수는 아니야.



### 3.1 Backgroud

 지금까지 짠 Pintos 코드는 다 OS kernel부분이였어. test code가 다 kernel에서 시스템이 특권을 줘서 실행된거야. 하지만 user program을 실행하면 더 이상 kernel에서 도는게 아니야. 이 플젝에서는 결과가 중요함.

 한 번에 하나 이상의 process를 실행할 수 있도록 허용해야해. 각 프로세스에는 하나의 thread가 있어. (multi-threaded process안할겨. 지원안해). user program은 전체 machine을 가지고 있다고 생각하고 쓰여질거야. 그래서 여러 process를 실행할 때, 이런 illusion을 유지하려면 memory, scheduling이나 다른 상태를 잘 관리해야해.

 이전 플젝은 직접 kernel에서 컴파일했으니까 특정 함수가 필요했었어. 하지만 지금은 user program을 실행해서 OS를 검사할거야. 이게 훨씬 자유로워. user program interface가 여기 설명된 스펙만 충족하면 되긴 하는데, 제한 조건 아래에서 kernel code를 조건 아래에서 다시 쓰고싶으면 rewrite해.



#### 3.1.1 Source Files

 보자. ㅋ. `userprog` 엔 파일이 별로 없어.

`process.c`

`process.h`

ELF binaries를 load하고 실행해. [Executable and Linkable Format이래]

`pagedir.c`

`pagedir.h`

 80x86 하드웨어 page table의 간단한 관리자임. 수정하진 않는데, 일부 함수를 호출할 수 있어. 자세한건 Section 4.1.2.3 [Page Tables]보셈.

`syscall.c`

`syscall.h`

user process가 kernel에 access하려고 하면 system call호출한다. system call handler의 skelton code야. 지금은 메세지를 print하고 user process를 종료하는데, 이 플젝의 part2에서 system call에 필요한 코드를 많이 추가해야 할거야. [너가 ㅋ]

`exception.c`

`exception.h`

 user program이 권한 밖, 금지된 작업 실행하면 "exception"이나 "fault"로 kernel에 trap됨. exception다룬단 얘기임. 지금은 message print하고 process종료하는게 다임. 전부는 아닌데 플젝2에서 `page_fault()` 를 수정해야 할 거야. 

`gdt.c`

`gdt.h`

 80x86은 segmeted architecture야. GDT(global descriptor table)은 segment를 설명하는 table임. 이 파일들은 GDT설정하는데, 이건 수정할 필요가 없어. GDT가 어떻게 작동하는지 궁금하면 읽어보던가.

`tss.c`

`tss.h`

 TSS(Task-State Segment) 는 80x86 architecture의 task switching에 사용됨. Pintos는 Linux처럼 user process가 interrupt handler를 시작할 때 stack을 switch 하기 위해서 사용됨. 이거 수정할 필요 없어. TSS가 어케 되는지 궁금하면 읽어보던가.



#### 3.1.2 Using the File System

file system code와 interface해야 하는데. `filesys` directory에 `filesys.h`랑 `file.h`인터페이스를 살펴봐야해. 이 플젝은 file system code수정할 필요 없어. 일단 다음과 같은 한계가 플젝 4전까지 있을거야

- 내부 동기화 없어. 한 번에 하나의 프로세스만 file system code를 실행하도록 synchronize해야해.
- 파일 크기는 생성시 고정이야. 파일 수도 제한됨
- 파일 데이터는 single extent로 할당됨. 그니까 한 파일이면 disk의 연속 sector차지.
- subdirectory 없어
- 파일 이름 14character 제한
- system crash로 disk손상될 수 있는데, 복구 도구 없어.
- **중요함** `filesys_remove()` 는 unix-like semactics야. 파일이 열려있는데 삭제되면, 마지막 thread가 닫을 때 까지 open된 상태인거니까 access할 수 있단 뜻임. 자세한 건 [Removing an Open File] 봐

 file system partition이 있는 simulate된 disk를 만들 수 있어야해. `pintos-mkdisk` 프로그램은 이 기능을 제공해. `userprog/build` directory에서 `pintos-mkdisk filesys.dsk --filesys-size=2` 해. 이 명령은 `filesys.dsk`로 이름이 지어진 2MB 의 Pintos file system partition을 포함하는 simulated disk를 만듬. kernel command line에서  `$pintos -f -q`를 하면 file system partition에 `-f -q`옵션이 전달되는데, `-f`는 포맷지정, `-q` 는 포맷 지정되면 Pintos 종료하게 해줌. 

 simulated 된 file system을 안<->밖 복사할 수 있어야 해. `-p` ("put")이거랑 `-g` ("get") option이 도와줄거임. `file`을 Pintos file sysetm에 복사하려면 `$pintos -p file -- -q`쓰셈. "newname" 으로 하려면 `-a newname` 넣어. `$pintos -p file -a newname -- -q`이렇게. VM에서 밖으로 하는 방법은 비슷한데 `-p`대신 `-g`쓰렴.

 다음은 file system partition이 있는 disk를 만들고, file system을 포맷하고, echo program을 새 disk에 복사한 다음, argument x 를 전달하는 방법이다. 이미 build했고 현재 directory를 `userprog/build` 라고 가정하자

```
$pintos-mkdisk filesys.dsk --filesys-size=2
$pintos -f -q
$pintos -p ../../examples/echo -a echo -- -q
$pintos -q run 'echo x'
```

마지막 세 단계는 하나의 명령으로 결합할 수 있어

```
$pintos-mkdisk filesys.dsk --filsys-size=2
$pintos -p ../../examples/echo -a echo -- -f -q run 'echo x'
```

나중에 사용하거나 검사하려는 목적이 아니라서 file system disk를 보관하지 않으려면 한 번에 할 수 있다.

```
$pintos --filesys-size=2 -p ../../examples/echo -a echo -- -f -q run 'echo x'
```

`$pintos -q rm file` 로 file system에서 파일 삭제할 수 있어. `$ls`  나 `$cat file` 도 있어.



#### 3.1.3 How User Programs Work

 Pintos는 메모리에 맞고, 너가 구현한 system call만 쓰는 일반적인 C program을 실행할 수 있어. `malloc()` 구현 못함. 부동 소수점 연산을 사용하는 프로그램도 못써.

 `src/examples` directory에 몇 가지 샘플 user program이 있어. `Makefile`은 예제 컴파일하고, 너가 편집하면 너가 만든 파일도 컴파일할 수 있어. 이 중 일부는 플젝3 혹은 플젝4가 구현된 이후에만 작동해.

 Pintos는 `userprog/process.c` 에 제공된 loader로 ELF executable을 load할 수 있어. ELF는 많은 OS에서 쓰는 object file이야. 80x86 ELF executable를 출력하는 컴파일러와 링커를 이용해서 Pintos용 프로그램을 만들 수 있어. (컴파일러와 링커는 pintos에서 제공함)

 simulated된 file system에 test program을 복사할 때 까지 Pintos는 작업을 할 수 없어. 다양한 프로그램을 file system에 복사할 때 까지 재밌는 일을 할 수 없을거야. clean reference file system disk를 만들고, `filesys.dsk` 를 지울 때 마다 복사해서 useful state가 될 때를 넘어서, 디버깅중에 때때로 발생해. [????????? 뭐라는겨]



#### 3.1.4 Virtual Memory Layout

 Pintos에서 Virtual memory는 두 가지로 나뉘는데, kernel virtual memory랑 user virtual memory다. user virtual memory는 virtual address 0에서 `PHYS_BASE`까지고, 저건 `threads/vaddr.h` 에 정의되어 있어. 기본값은 0xc0000000 (3GB)야. kernel virtual memory는 virtual address의 나머지 부분을 차지하고 `PHYS_BASE` 부터 위쪽이야. 4GB임.

 user virtual memory는 process마다 있어. kernel 이 한 process에서 다른 process로 switch하면, user virtual address space를 process의 page directory base register를 바꾸는걸로 switch하는거야. (`userprog/pagedir.c` 안에 있는 `pagedir_activate()` 를 봐) `struct thread` 는 process의 page table의 pointer를 포함하고 있어.

kernel virtual memory는 global임. 실행중인 user process나 kernel thread에 상관 없이 항상 같은  방식으로 mapping된다. Pintos에서 kernel virtual memery는 `PHYS_BASE` 로 시작해서 1대 1로 mapping됨. 이건 virtual address`PHYS_BASE`에 access하는게 physical address 0에 access하는거야. 그리고 virtual address `PHYS_BASE + 0x1234`가 physical address에서는 0x1234인거지. 컴퓨터 실제 메모리 크기까지 계속 access함.

 user program은 각각의 user virtual memory에만 access할 수 있어. kernel memory에 access하려고 하면 `userprog/exception.c` 에서 `page_fault()` 를 불러서 page fault발생하고 process종료함. kernel thread는 kernel virtual memory랑 user process가 돌고 있으면 실행중인 user virtual memory까지 access할 수 있어. 하지만, kernel에서도 mapping 안된 user virtual address에 access하려고 하면 page fault발생해.



##### 3.1.4.1 Typical Memory Layout

 실제로 user virtual memory는 다음과 같이 배치됨

![스크린샷 2017-03-26 19.19.15](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-03-26 19.19.15.png)

 이 플젝에서 user stack의 크기는 고정되어 있지만, 플젝3에서는 확장이 허용됨. 전통적으로 초기화 안된 data segment의 크기는 system call로 조정할 수 있지만, 구현할 필요는 없어.

 Pintos 에서 code segment는 user virtual address 0x08084000부터 시작해. address space의 bottom에서 약 128MB 위에 있지. 큰 의미는 없어.

 linker는 다양한 프로그램 segment의 이름과 위치를 알려주는 "linker script"의 지시에 따라 memory안의 user program의 layout을 설정해. `info ld` 를 통해 access 할 수 있어. 자세한 건 "Scripts"를 봐.

 특정 실행 파일의 layout을 보려면 `-p` 옵션과 함께 `objdump` 나 `i386-elf-objdump` 를 실행해.



#### 3.1.5 Accessing User Memory

 system call의 일부로, kernel은 종종 user program이 제공하는 pointer를 통해 memory에 access해야 해. user 가 null pointer, mapping 안된 virtual memory를 가리키거나, kernel virtual address space(`PHYS_BASE` 위쪽에 있는)를 가리키지 않도록 kernel이 주의해야 한다. 이러한 잘못된 pointer는 kernel이나 다른 실행중인 process에 해를 끼치지 않고 종료되어야해. 

 적어도 두 개의 적절한 방법이 있어. 첫번째는 user가 준 pointer의 유효성을 확인한 뒤에, 그걸 dereference하는 방법이야. 이 방법을 선택하면 `userprog/pagedir.c` 와 `threads/vaddr.h` 에 있는 함수를 봐. 이게 user memory access를 handle하는게 가장 쉬운 방법이야. 

 두번째 방법은, user pointer가 `PHYS_BASE` 아래만 가리키도록 체크하고 dereference하는 방법이야. invalid user pointer가 `userprog/exception.c` 에서 `page_fault()` 를 사용해서 "page_fault"를 발생시키는거야. 이 기술은 processor의 MMU를 사용하기에 더 빨라서 실제 kernel에 이용되기도 해. 

 두 경우 모두 resource가 leak 되지 않도록 해야해. 예를 들어, system call이 acquired lock이나 `malloc()` 으로  allocate된 상태의 memory를 획득했다고 해보자. 이후에 invalid user pointer가 발생하면 release lock하거나 page memory를 free시켜야해. 만약 너가 user pointer를 dereferencing하기 전에 확인하도록 했다면 이런건 간단해야해. invalid pointer가 page fault를 발생시면 더 어려워. 왜냐면 memory access에서 error code를 return 방법이 없거든. 그래서 후자를 이용하려는 사람들을 위해 다음과 같은 code를 줄게.

```c
/* Reads a byte at user virtual address UADDR.
   UADDR must be below PHYS_BASE.
   Returns the byte value if successful, -1 if a segfault
   occurred. */
static int
get_user (const uint8_t *uaddr)
{
  int result;
  asm ("movl $1f, %0; movzbl %1, %0; 1:"
       : "=&a" (result) : "m" (*uaddr));
  return result;
}
/* Writes BYTE to user address UDST.
   UDST must be below PHYS_BASE.
   Returns true if successful, false if a segfault occurred. */
static bool
put_user (uint8_t *udst, uint8_t byte)
{
  int error_code;
  asm ("movl $1f, %0; movb %b2, %1; 1:"
       : "=&a" (error_code), "=m" (*udst) : "q" (byte));
  return error_code != -1;
}
```

 이 기능은 user address가 이미 `PHYS_BASE`보다 낮은걸로 확인되었단 가정 하에 실행한거야. `page_fault()`를 수정해서 kernel의 page fault 가 일어나면 `eax`를 `0xffffffff` 로 설정하고 이전 값을 `eip` 로 복사했다고 가정함. [잘 이해가 안되는데 원문 읽으셈.]



### 3.2 Suggested Order of Implementation

 다음 사항들은 parallel하게 일어날 수 있으니까 먼저 구현해.

- Argument passing. (Section 3.3.3 [Argument Passing] 보셈). 모든 user program은 argument passing이 될 때 까지  page fault를 표시할거야.

  지금은 단순히 `setup_stack()`에서  `*esp = PHYS_BASE;` 를 `*esp = PHYS_BASE - 12;` 로 바꿀 수 있어. 이건 test program이 argument를 검사하지 않아. 이름이 (null)로 print되더라도 말이지.

  argument passing이 구현될 때 까지, passing command-line argument없이 program을 실행해야 할거여. argument pass하려고 하면 program 이름에 포함되서 실패할 수도 있어.

- user memory access. (Section 3.1.5 [Accessing User Memory]를 봐) 모든 system call은 user memory를 읽어야 해. user memory에 write할 필요가 있는 system call은 거의 없어.

- system call infrastructure. (Section 3.3.4 [System Calls]봐). user stack에서 system call number를 읽고, 이를 기반으로 handler에 dispatch할 수 있는 코드를 구현하자.

- system call **exit**. user program이 정상적으로 **exit**을 불러서 끝내는거야. 심지어 한 program 이 `main()`에서 return 되더라도 **exit**을 간접적으로 불러야지. (`lib/user/entry.c`에서 `_start()`를 보렴.)

- system console인 fd 1에 write할 수 있는 **write** system call. 우리가 실행할 모든 test program은 console에 wrtie할 거야. ( user process version의 `printf()`가 이런 식으로 구현됨). 그래서 **write**가 가능할 때까지 모두 오작동할거야. 

- 지금부턴, `process_wait()` 를 infinite loop로 바꿔야 해. 원래는 즉시 return을 불렀는데, Pintos 는 실제로 process가 실행되기 전에 꺼져. 올바르게 구현하렴. [ㅋ]

위를 구현한 뒤에는 user process가 최소한으로 작동해야해. 최소한 console에 wrtie하고 exit을 올바르게 해야지. 이 다음부터 구현을 제대로 해서 일부 테스트를 통과할 수 있어. 



### 3.3 Requirements

#### 3.3.1 Design Document

[버려]

#### 3.3.2 Process Termination Messages

 exit하거나 다른 이유로 user process가 종료될 때 마다 process의 이름과 exit code를 다음과 같이 `printf ("%s: exit(%d)\n", …);` 로 프린트해야해. `process_execute()`에서 전달받은 프로세스의 full name을 print해야해. 이건 user process가 종료된게 아니라 kernel thread나 system call로 halt된거면 출력하지 마. process가 load fail되면 message 해도 되고 안해도 됨. 니 맘임.

이 외에도  Pintos가 이미 print하지 않은 다른 메세지는 print하지마. 점수 스크립트에 혼동줌.



#### 3.3.3 Argument Passing

`process_execute()`는 argument를 new process로 전달하는거 지원 안해. `prcess_execute()`를 확장해서 단순히 빈 공간으로 단어를 나눠서 program file name이랑 그 argument로 나눠야 해. 첫 단어는 program name이고 두번째 단어가 첫 argument고 계속 이렇게. `process_execute("grep foo bar")` 이렇게 있으면 `grep`이 실행되고 `foo` 와 `bar` 가 argument로 전달.

 command line에서 multiple spaces는 single space랑 같음. 그니까 space바 많이 쳐도 똑같게 해야해. 길이에 적당한 제한을 줄 수 있어. 예를 들어, 단일 페이지에 들어갈 수 있는 정도로 제한할 수 있어. 대충 4kb정도 됨. (pintos utility가 kernel에 전달할 수 있는 최대 128bytes command-line argument는 제한하면 안된다.)

 원하는 방식으로 argument를 parse할 수 있어. `strtok_r()`을 보려면 `lib/string.h`안에 있는 프로토타입을 보고 `lib/string.c`에 주석을 이해하면 됨.. `$man strtok_r`을 실행해봐.

 자세한건 Section 3.5.1 [Program Startup Details]보고.



#### 3.3.4 System Calls

`userprog/syscll.c` 에 system call handler를 구현해야해. 일단 지금 스켈레톤으로 구현된 거는 process를 종료해서 system call을 handle함. system call number를 검색한 다음에 그에 맞는 행동을 해야함. 

 다음에 나오는 system call을 구현해. `lib/user/syscall.h` 에 포함된 user program에 의해 표시된 프로토 타입이야. 각 system call number에 대한 system call은 `lib/syscall-nr.h` 에 정의되어 있어. 

<u>System Call</u> : void **halt** (void)

 `shutdown_power_off()` 를 불러서 Pintos 를 끈다. `devices/shutdown.h` 에 있어. deadlock 상황으로 정보를 잃어버릴 수 있어서 거의 사용하지 마렴. [이거 스탠포드 버전에만 있음. `threads/init.h` 에 있는 `power_off()` 를 쓰자]

<u>System Call</u> : void **exit**  (int **status**)

 current user program을 종료하고 kernel에 status를 return해준다. process의 부모가 **wait**하고 있다면, 이 **exit**의 argument인 **status**가 return될 상태임. 일반적으로 status 0은 성공이고 0이 아니면 error야.

<u>System Call</u> : pid_t **exec** (const char * **cmd_line**)

 cmd_line에 주어진 이름의 파일을 실행하고 주어진 argument를 전달한 다음, 새 process's program id(pid)를 return한다. 어떤 이유로든 load나 실행할 수 없는 경우 pid-1을 return해야해. 그래서 child process가 성공적으로 load되고 실행가능한지 알 때 까지 exec는 return할 수 없어. 이를 위해서 적절한 synchronization을 사용해야 한다.

<u>System Call</u> : int **wait** (pid_t **pid**)

 child process pid를 기다리고 child의 exit status를 가져온다. 

 **pid**가 여전히 살아있다면, 종료될 때 까지 wait한다. **pid**가 exit을 받으면 status를 return한다. pid가 `exit()`을 call하지 않았는데, kernel에 의해 종료되면 wait(pid)는 -1을 return해야 한다. parent process가 child processes를 기다리는건 완벽하게 합법이지만, parent가 **wait**이어도 child의 exit status를 검사하거나 kernel에 의해 child가 종료되었는지 확인해야해. 

다음 조건 중 하나라도 해당되면 **wait**는 즉시 -1을 return해야 한다. 

- **pid**는 직접 calling process의 child를 참조하지 않아. **pid**는 calling process가 exec이 성공적으로 되어서 return한 pid일 때만 calling process의 direct child임.

  child는 상속되지 않음(not inherited). A가 B를 생성하고, B가 C를 생성해도, A는 C를 wait하지 않아. B가 죽어도 말이지. A의 `wait(C)`는 실패해야해. 비슷하게 고아 process는 parent process가 먼저 종료되어도 새 parent를 할당받지 않음.

- wait를 호출한 process는 이미 pid에 wait를 호출했어. process는 주어진 child 를 최대 한 번 기다릴 수 있단 말이야.

process는 얼마든지 child를 생성할 수 있고, 어떤 순서로든지 wait할 수 있어. 심지어 모든 child를 기다리지 않고 종료될 수도 있어. 너는 모든 경우의 수를 생각해서 디자인해야해. `struct thread`를 포함한 모든 process의 resources는 wait와 parent process의 존재와 상관없이 free되어야함.

initial process가 끝날 때 까지 Pintos가 종료시키면 안됨. 제공된 Pintos code는 `process_wait()`( `userprog/process.c`안에 있음)를 부르려고 시도할거야. 이 코드는 `threads/init.c` 에 있는 `main()`에서 call함. `process_wait()`를 먼저 구현한 다음에(함수 맨 위에 주석으로 써둠) **wait** system call을 구현하는걸 추천해. 이 system call을 구현하려면 나머지 어느 것보다 더 많은 작업이 필요해.

<u>System Call</u> : bool **create** (const char * **file**, unsigned **initial_size**)

 **initial_size** 크기의 **file**이라고 이름붙은 새 file을 만든다. 성공하면 true를 return하고, 아니면 false return. 만들기만 하고 열지는 않는다. 열려면 open이 필요함.

<u>System Call</u> : bool **remove** (const char * **file**)

 **file**이라고 이름붙여진 file을 삭제한다. 성공하면 true, 아니면 false를 return. 열려있든 닫혀있는 상관없이 제거될 수 있으며, 열려있는 상태로 제거되도 파일 안닫음.

<u>System Call</u> : int **open** (const char * **file**)

**file**이라는 file열어. "file descriptor"이라는 (fd) 음이 아닌 정수를 return해. 못열면 -1 return하고.

0과 1은 console을 위한 거야. fd 0 (`STDIN_FILENO`)은 standard input이고, fd 1(`STDOUT_FILENO`)는 standard output임. **open** system call은 이 파일 descriptor를 return하지 않아. 

각 process에는 독립적인 file descriptors가 있어. child process에게 상속 안됨.

다른 process든, 같은 process든 한 file이 두 번 열리면, 각각의 open은 새로운 file descriptor를 return해. **close**는 file position을 공유하지 않아. [뭔소리야...]

<u>System Call</u> : int **filesize** (int **fd**)

 **fd**로 open된 파일의 크기를 byte로 return함.

<u>System Call</u> : int **read** (int **fd**, void * **buffer**, unsigned **size**)

 **fd**로 열린 파일의 size 를 byte로 buffer에 읽어들인다. 실제로 읽힌 bytes를 return함. (파일 끝에서 0). 파일을 읽을 수 없으면 -1 return(파일 끝 이외 조건으로). fd 0은 `input_getc()` 를 사용해서 키보드에서 읽는다. 

<u>System Call</u> : int **write**  (int **fd**, const void * **buffer**, unsigned **size**)

buffer로 열린 파일 **fd**로 size bytes를 쓴다. 실제로 적힌 byte를 return한다. 일부 byte를 쓸 수 없는 경우엔 **size**보다 작을 수 있어.

파일의 끝을 지나면 정상적으로 확장되지만, 기본 file system에는 구현이 안되어있음. 가능한 많은 byte를 쓰고 쓰여진 만큼 return하거나 쓸 수 없으면 0을 return함.

fd 1은 console에 기록한다. 수백byte보다 크지 않는 한 `putbuf()`를 불러 한 번의 buffer로 작성해야해. larger buffer는 break야기함. 아니면 다른 process의 test output이 채점 script와 사람이 읽는거에 혼동을 줄 수 있어.

<u>System Call</u> : void **seek** (int **fd**, unsigned **position**)

 열린 파일 fd에서 읽거나 쓰게 될 다음 byte를 파일의 시작 부분부터 byte단위로 바꾼다. (그니까 position 0은 file의 시작부분임) 

파일의 current end를 지나쳐도 에러가 아니야. 나중에 파일의 끝을 나타내는 0byte를 얻는다. [네?] 이후의 쓰기는 파일을 확장하여 작성되지 않은 공간을 0으로 채운다. 그런데 플젝4가 완료될 때 까지 pintos의 파일 길이는 고정되어 있으니까 파일 끝 부분을 기록하면 오류나옴. 이런 semantic은 file system에서 구현되며 특별한 노력이 필요하지 않음.

<u>System Call</u> : unsigned **tell** (int **fd**)

 열려있는 파일 fd의 읽거나 쓸 next byte를 return함. 시작부터 byte단위로. 

<u>System Call</u> : void **close** (int **fd**)

 fd닫음. process를 종료하면 각 process가 종료되면서 파일도 닫혀야 한다. 

system call을 구현하려면 user virtual address space에서 데이터를 읽고 쓰는 방법을 제공해야해. system call number는 user's virtual address space에 있는 user's stack위에 있어서, 이 기능이 필요한거야. 까다로울 수 있어. user가 invalid pointer, kernel memory를 가리키는 pointer, block된 region을 가리키면 어케해. user process를 종료해야함. 이 code를 쓰고 테스트하는걸 추천해. Section 3.1.5 [Accessing User Memory]봐.

 여러 user process가 system call을 동시에 수행할 수 있도록 system call을 synchronize해야해. `filesys` directory에 제공된 file system code를 여러 thread에서 동시에 호출하는건 안전하지 않아. system call은 file system code를 critical section으로 설정해놔야해. `process_execute()` 도 access file한다는걸 잊지마. 지금은 `filesys` directory를 수정하지 않는게 좋아.

`lib/user/syscall.c` 에서 각 system call에 대한 user-level function을 제공해. 적절한 경우 system call을 호출하고 system call의 return값을 return함.

이 부분을 끝내면, Pintos는 bulletproof해야해.[안전해야 한다는 뜻인듯. 방탄.] user program은 OS가 crash되거나, panic, fail하도록 놔두면 안됨. user program이 OS를 정지시킬수 있는 유일한 방법은 **halt** system call을 부르는 거야.

system call에 invalid argument가 전달되면, (값을 return 하는 경우)error value를 return하고, 아니면 process를 종료해야해.

자세한건 Section 3.5.2 [System Call Details]봐



#### 3.3.5 Denying Writes to Executables

실행 파일로 사용중인 파일에 대한 쓰기를 거부하도록 code를 추가해야해. 플젝3에서 매우 중요한데, 지금은 별 상관 없을수도 있어. 

`file_deny_write()`를 사용해서 열린 파일에 대한 쓰기를 방지할 수 있어. 파일에서 `file_allow_wrtie()`를 호출하면 다시 가능함. (하지만 다른 opener가 deny하면 불가능). 파일을 닫으면 쓰기도 다시 활성화됨. 따라서 process가 실행중일 때만 이게 유지되어야함.



## 3.4 FAQ

코드 얼마나 써야함?

```
threads/thread.c		| 13
threads/thread.h		| 26
userprog/exception.c	| 8
userprog/process.c		| 247 
userprog/syscall.c		| 468
userprog/syscall.h		| 1
```

[나머지 생략]



## 3.5 80x86 Calling Convention

calling convention은 이렇게 작동함

1. caller는 각 함수의 argument를 어셈블리 언어 **PUSH**를 이용해서 하나씩 stack에 push한다. arguments는 right-to-left순서로 push됨. 

   stack은 downward로 자란다. push마다 stack pointer가 감소하는겨. `*--sp = value`처럼

2. caller는 다음에 실행할 주소를 push하고(그니까 return address) callee의 처음 instruction으로 jump함. **CALL**이 둘 다 함.

3. callee가 실행됨. callee가 control을 가지면, stack pointer는 return address를 가리킨다. 첫번째 argument가 그 return address바로 위에 있고, 두번째 argument가 그 첫번째 argument바로 위에 있도록 말이다. 

4. callee가 return value를 가지면, register **EAX**에 저장한다.

5. callee가 stack 에서 return address를 pop하면서 return하고, 그 주소로 80x86의 **RET** instruction을 이용해서 jump한다.

6. caller가 stack에서 argument를 pop한다.



#### 3.5.1 Program Startup Details

Pintos에서 `lib/user/entry.c`에 있는 `_start()`로 user program이 지정되어 있어. `main()` 이 return하면 `main()` 을 wrap하는 `exit()` 함수가 그걸 받아서 끝냄.

```c
void
_start (int argc, char *argv[])
{
  exit (main (argc, argv));
}
```

 kernel은 intial funcion을 위해서 argument를 stack에 넣어야 해. user program이 시작되기 전에 말이지.  argument 는 normal calling convention과 같은 방법으로 전달됨. [바로 위에 있던 3.5 처럼 말이지.]

 `/bin/ls -l foo bar` 를 어떻게 다루는지 보자. 첫번째로 커맨드를 나눠야해. `/bin/ls`, `-l`, `foo`, `bar`이렇게. 이걸 stack의 top에 넣자. 순서는 안중요해. 왜냐면 pointer로 refernce될거니까. [포인터랑 순서랑…뭔상관이지?]

 그리고 나서, right-to-left order로 stack에 push하는데, 각 string다음에 null pointer sentinel을 붙여서 넣어야 해. 이게 **argv**의 element임. null pointer sentinel은 **argv[argc]**가 null pointer인걸 보장해줌. [읭? 해석 잘못된듯함.] 그리고 순서가 argv[0]이 가장 낮은 virtual address인걸 보장하고. word-aligned access는 unligned access보다 빠름. 그래서 stack pointer을 첫 push전에 4의 배수로 자르는거임. 

 그리고 **argv**(argv[0]의 주소임)랑 **argc**를 순서에 맞게 push하자. 마지막으로 fake "return address" 를 push함: entry function을 절대 return 안함, stack frame은 반드시 다른 stack frame과 구조가 같아야해. 

 아래 table이 user program 시작 전의 관련 register의 상태를 보여주고 있음. `PHYS_BASE` 가 0xc0000000이라고 가정하자.

![스크린샷 2017-04-03 14.55.02](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-04-03 14.55.02.png)

![스크린샷 2017-04-03 14.31.44](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-04-03 14.31.44.png)

여기서 stack pointer는 0xbffffcc로 초기화될거야. 

보다시피 user virtual address의 top에서 시작해야 하고, 그건 virtual address인 `PHYS_BASE`바로 밑이야. (`threads/vaddr.h`에 정의되어 있어.)

`hex_dump()`라는 함수가 `<stdio.h>` 에 있는데, 잘 쓰렴. 다음처럼 나와.

[`hex_dump((uintptr_t) (PHYS_BASE - 150), (void **) (PHYS_BASE - 150), 150, true);` 난 이렇게 썼음. 이거 스택 다 채우고 다음줄에 붙여넣으면 밑에처럼 나옴. `PHYS_BASE` 는 직접 `0xc0000000`을 넣든가 마음대로]

![스크린샷 2017-04-03 14.34.40](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-04-03 14.34.40.png)



#### 3.5.2 System Call Details

 OS는 프로그램 코드에서 발행하는 이벤드인 software exception을 처리할 수 있다. page fault나 division by zero같은  error말이다. 아니면 request services("system calls")같은 것도.

 80x86 architecture에서는 `int` 명령으로 대부분의 system call을 실행한다. pintos에서는 `int $0x30`으로 system call을 만든다. system call number나 추가적인 argument는 interrupt되기 전에 normal한 방법으로 stack에 push될 것으로 예상된다.

 따라서 `syscall_handler()` 가 control을 얻으면, caller의 stack pointer인 32bit word로system call number, 첫 argument는 32bit word의 바로 다음 높은 address에 있다. caller's stack pointer는 `syscall_handler()`에 의해 access가능하고, 이건 `struct intr_frame` 가 넘겨준`esp`라는 member임. (`struct intr_frame`은 kernel stack에 있다.)

 80x86구조에서는 return value를 register **EAX**에 저장한다. system call은 `struct intr_frame`의 member인 `eax`에 저장함. 그러니까 return value를 저장하고 싶으면 이걸 수정하면 되는거야.

 각 system call arguement는 integer나 pointer로 되어서 4bytes의 크기로 stack에 있음. 그니까 stack 거슬러 올라가다 보면 각 system call에 필요한 argument가 순서대로 들어와 있을거야. 





[번역 종료 너무 많다]

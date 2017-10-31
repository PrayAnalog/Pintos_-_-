## 4 Project 3:Virtual Memory

 이제 핀토스의 inner working에 익숙해졌을겨. 너의 OS는 multiple thread의 execution을 적절한 synchronization으로 handle할 수 있을거고, multiple user porgram을 한 시점에 load 할 수 있을거야. 그러나! 프로그램의 숫자와 크기는 machine 의 main memory size의 한계가 있어. 이번엔 이 한계를 없애보자! [유리천장따윈 부숴버려!...]

 이 부분은 마지막에 했던거에 추가적으로 붙을거야. 플젝2의 테스트도 플젝3에서 작동해야함. 플젝3 시작하기 전에 플젝2 버그 다 잡으렴. 왜냐면 플젝3에서 결국 똑같은 버그 나올거거든. [ㅋ]

 플젝2에서 했던 거랑 같은 방식으로 Pintos disk랑 file system처리할거야. Section 3.1.2 [Using the File System]봐.



### 4.1 Background

#### 4.1.1 Source Files

 `vm` directory에서 이번 플젝이 진행됩니다. 그런데 `vm` 안에는 `Makefile` 밖에 없음. `userprog` 안에 있는것들을 `-DVM` 으로 실행한다는 것만 다름. 너가 작성한 모든 코드는 새 파일에 들어있거나, 이전 프로젝트에 도입된 파일들에 있어. [있다는거야 있어야 한다는거야 뭐라는거야]

`devices/block.h`

`devices/block.c` 

​ block device에 access해서 sector-based read와 write를 지원해. 이 interface를 이용해서 한 block device인 swap partition에 access할거야. 

[우린 block에다 안하고 disk이용할거임 ㅋ]



#### 4.1.2 Memory Tereminology

[메모리 용어? 언어?]

 virtual memory에 대해 계속 생각해야해.

##### 4.1.2.1 Pages

 virtual page라고도 불리는 page는 연속적인 virtual memory 4,906 bytes(page size)임. page는 page-aligned되어 있어야 하는데, 뭔말이냐면, virtual address는 page size로 나눠져야 한단 말임. 그니까 virtual address는 page size의 배수라고. 32 virtual address는 20bit의 page number와 12bit의 page offset(그냥 offset)으로 나눠짐. 이렇게 말야.

![스크린샷 2017-04-11 12.07.48](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-04-11 12.07.48.png)

 각 process는 independent set 의 user (virtual) pages가 있어. 이 페이지는 virtual address PHYS_BASE아래에 있는 page들이야. Pintos는 PHYS_BASE가 0xc0000000(3GB)쓰지. kernel (virtual) pages는 당연히 이 위쪽에 있고, global임. kernel은 물론 user과 kernel page에 다 접근할건데, user process는 자신의 user pages에만 접근해야 함. Section 3.1.4 [Virtual Memory Layout] 봐.

 Pintos는 virtual addresses에서 작업하기 위해서 몇가지 function제공해. Section A.6 [Virtual Addresses]보셈.



##### 4.1.2.2 Frames

 Frame혹은 physical frame이라고도 하는데, physical memory에 continuous region이야. [한 page안에서는 continuous한 physical address를 갖는다는 말임. ] page처럼 frame도 page-size랑 같은 size고 page-aligned되어 있어야 해. [page-aligned는 physical memory가 page-size의 배수란 뜻이겠지!]. 그래서 32bit physical address는 20bit frame number과 12bit frame offset으로 이뤄짐.

![스크린샷 2017-04-11 12.14.32](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-04-11 12.14.32.png)

 80x86 architecture에서는 physical address로 바로 memory access하는거 지원 안함. Pintos는 kernel virtual memory를 physical memory로 mapping하는 걸로 이 문제를 해결한다. kernel virtual memory의 첫번째 page는 physcial memory의 첫번째 frame으로 mapping되고 두 번째 page는 두 번째 frame 이렇게 쭉 mapping됨. [user virtual memory가 아니라 kernel virtual memory라는거 명심해라.] 따라서 이렇게 kernel virtual memory로 frame에 access 할 수 있음.

 Pintos는 kernel virtual addresses와 physical addresses를 서로 translate할 수 있는 function을 제공해. Section A.6 [Virtual Addresses]보렴.



##### 4.1.2.3 Page Tables.

 Pintos에서, page table은 CPU가 virtual address를 physical address로 translate하는데 쓰는 data structure임. 그니까 page에서 frame으로 바꿔준다고. page table format은 80x86 architecture에 의해 결정됨. Pintos는 `pagedir.c` 에서 page table 관리하는 code를 제공해. (Section a.7 [Page Table]을 보렴)

 밑에 그림은 page와 frame관계를 보여줌. 왼쪽에 virtual address는 page number와 offset을 가지고 있어. pagee table 는 page number를 frame number로 translate해주고, 수정되지 않은 똑같은 offset을 붙여서서 오른쪽처럼 physical address로 합쳐줌. 

![스크린샷 2017-04-11 13.17.53](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-04-11 13.17.53.png)



##### 4.1.2.4 Swap Slots

 swap slot은 continuous하다, swap partition에 있는 page-size크기만한 disk space임. hardware가 정한 한계는 page나 frame 보다는 느슨하게 잡아놨지만, swap slot은 page-aligned해야 함. 왜냐면 밑에 부분에는 하는게 없거든.



#### 4.1.3 Resource Management Overview

 디자인하고 코딩하려면 다음 data structure들이 필요할겨.

<u>Supplemental page table</u>

 page talble을 보완해서 page fault handling을 enable하게 해야함. Section 4.1.4 [Managing the Supplemental Page Table]보셈.

<u>Frame table</u>

 eviction[축출] policy를 효율적으로 구현할 수 있도록 도와줌. Section 4.1.5 [Managing the Frame Table]보셈.

<u>Swap table</u>

 swap slots의 사용량을 track함. Section 4.1.6 [Managing the Swap Table]봐.

<u>Table of file mappings</u>

 processes는 virtual memory space에 file을 mapping할 수 있어. 어떤 page에 어떤 file이 mapping 되었는지 track하려면 table이 필요할거야.

 완전히 다른 4개의 data structure로 구현할 필요는 없긴 함. 전체나 부분적으로 합쳐서 구현하는게 편리할 수도 있어. [니 맘대로 해]

 각각의 data structure에서 element들이 어떤걸 포함할지 결정해야함. local일지(per process) global일지 (applying to the whole system) 등등.

 단순하게 하기 위해서, page구조가 아닌 data structure에 이런걸 store할 수도 있어. 하지만 이 방법을 쓰려면 그 pointer가 계속 유효하게 남아있는지 확실해야 쓸 수 있겠지. 

 array, list, bitmap, hash table같은 data structure을 쓸 수 있겠네. array는 간단하지만, 공간이 부족하면 memory를 낭비함. list도 간단하지만, 길어지면 특정 position으로 가려면 계속 읽고 next해야해서 시간걸림. array랑 list둘 다 resize가 가능하지만, list가 중간에 insertion이랑 deletion하는거 더 효과적임. 

 Pintos 는 `lib/kernel/bitmap.c` 이랑 `lib/kernel/bitmap.c`에서 bitmap data structure도 지원함. bitmap은 bit의 array인데, 각각은 true나 false값을 갖는다. bitmap은 일반적으로 (동일한) resources set에서 사용량을 track하는데 사용됨. resource **n**이 사용되면 bitmap의 bit **n**이 true야. Pintos bitmap의 크기는 고정되어있지만, 크기 조정을 할 수 있도록 구현을 바꿀 수 있어.

 Pintos는 hash table data structure도 있어. (Section A.8 [Hash Table]보셈). Pintos hash table은 wide range of table size에도 효과적으로 insertion과 deletion지원함.

 더 복잡한 data structure가 성능이나 더 좋을수도 있긴 한데, 구현을 불필요하게 복잡하게 할 수도 있으니까, 안넣는걸 추천함. (예를 들어 balanced binary tree같은거 비추.)



#### 4.1.4 Managing the Supplemental Page Table

 <u>supplemental page table</u> 은 각 page에 추가적인 data를 가진 page table이야. 이게 왜 필요하냐면, page table의 format때문에 한계가 있기 때문이지. page table이랑 헷갈리지 않도록 supplemental이란 단어를 붙일거야. 

 supplemental page table은 적어도 두 가지 목적으로 쓰여. 가장 중요한건, page fault 상태에서 그니까, kernel이 virtual page를 보고 fault했단 말은, supplemental page table에서 어떤 데이터가 있어야 하는지 찾아야 한단 말이야. 두번째로, process가 종료되면 kernel은 supplemental page table을 참조해서 어떤 resource를 free해야할지 결정하는거고. 

 너가 원하는대로 supplemental page table을 구성할 수 있어. 적어도 두 가지 기본적인 방식이 있는데, segments와 pages야. [네? 뭐라구요? in terms of segments or in terms of pages라는데 뭐라는거지…] 선택적으로, 넌 page table에 추가적으로 구성요소를 넣어서 그 자체로 supplemental page table로 쓸 수 있어. 이러한 방식을 쓰려면 `pagedir.c`에서 Pintos의 page table구현해 놓은걸 수정해야해. 그런데 이건 어려우니까 너가 갓갓이라면 하고, 갓갓이 아니라면 기본적인 방식으로 하자. Section A.7.4.2 [Page Table Entry Format]을 보렴.

 supplemental page table의 가장 중요한 사용법은 page fault handler야. 플젝2에서 page fault는 kernel이든 user program이든 항상 bug로 취급해서 `exit(-1);` 했잖아. 플젝3에서는 더 이상 이러면 안됨. page fault는 아마 file을 가져오거나 swap할 때 생길거야. 이러한 경우를 세련되게 handle해야함. page fault handler는, `userprog/exception.c` 안에 있는 `page_fault()` 를 수정하면 되고, 일단 roughly하게 다음 4가지를 만족시켜야해. 

1. supplemental page table에서 fault된 page를 찾자. memory reference가 유효하면, supplemental page table의 entry를 사용해서 page에 들어가거나 file system이나 swap slot이나 all-zero page일 수도 있는 data를 찾자. sharing을 구현하면, page data는 이미 page frame에 있겠지만, page table에는 없는 경우도 있어. 

   access가 invalid한 경우가 있는데, user process가 address에 어떠한 data가 있을거라고 기대를 안하는 곳에 supplemental page table이 가리키고 있는데 여기에 access할 때, [첫번째 경우는 이해를 못하겠다. 원서 읽자]또 kernel virtual memory에 page가 있는 경우도 안되고, read-only page에 write하려고 access하는 경우도 안됨. 이러한 경우에는 process가 종료되고 모든 resource가 free되어야 해. 

2. page에 저장할 frame을 얻자. Section 4.1.5 [Managing the Frame Table]을 보렴.

   sharing을 구현하면, 너가 필요한 데이터가 이미 frame에 있을 수 있어. 이 경우에는 그 frame을 찾을 수 있어야 해.

3. file system에서 data를 읽어오거나 swap하거나, zero화 하는걸로 frame에 data를 가져오자. 

   sharing을 구현하면, 필요한 page가 이미 frame에 있을 수 있는데 이 경우에서 더 필요한 과정은 없긴 함.

4. faulting한 virtual address를 physical page로 지정하자. `userprog/pagedir.c` 에 있는 함수를 쓰도록 하렴. 



#### 4.1.5 Managing the Frame Table

 frame table에는 user page가 들어있는 각각의 frame을 가리키는 entry를 포함해. frame table안에 있는 각 entry는 page를 포함하고 있다면 그 page의 pointer를 가지고 있고, 너가 원하면 data의 pointer일수도 있지. frame table은 Pintos에 free한 frame이 없을 때, 희생할 frame골라서 축출시키는 정책을 효율적으로 구현할 수 있도록 도와주는애야. 

 user page에 사용된 frame은 `palloc_get_page(PAL_USER)` 를 사용해서 "user pool" 에 포함되도록 해야해. "kernel pool"에 allocate되지 않도록 `PAL_USER`를 꼭 써야해. 아니면 test막 이상하게 된다. ([Why PAL_USER?]를 보렴) `palloc.c`를 수정해서 frame table 을 구현하면, 두 pool이 반드시 구별되도록 유지하면서 수정해야해. 

 frame table에서 가장 중요한 작업은 unused frame을 얻는거야. 이건 frame이 free하면 쉬운 작업이지. 하지만 아무것도 free하지 않다면, 어떤 page를 희생양으로 삼아서 축출시켜야 해. 그래야 frame도 비니까. 

 swap slot을 allocating하지 않고는 frame을 축출할 수 없지만, swap이 full이라면 kernel이 panic될거야. 실제 OS는 이러한 상황을 복구하거나 예방하기 위해서 광범위한 정책을 사용하지만, 이 플젝에서는 안할거임.

 eviction 은 roughly하게 다음과 같은 과정을 거쳐.

1. 너의 page replacement algorithm을 사용해서 evict할 frame을 고르자. 아래에 설명된 page table의 "accessed"와  "dirty" bit가 쉽게 결정해줄거야.

2. 참조하는 모든 page tables의 frame의 reference를 지워버리자. 

   sharing을 구현하지 않는 한, 주어진 시간에는 하나의 page만 하나의 frame을 참조해야해. 

3. 필요하다면, file system이나 swap에 page를 write하자. 

 evicted frame은 다른 page를 store할 때 쓸 수 있어.



##### 4.1.5.1 Accessed and Dirty Bits

80x86 hardware는 각 page의 PTE(page table entry)에 있는 한 쌍의 bit를 통해 page replacement algorithm을 도와준다. page에 read 나 write를 하면 CPU는 accessed bit를 1로 하고, write의 경우에는 dirty bit도 1로 설정해준다. CPU는 이걸 다시 0으로 바꾸지는 않고, OS가 함. 

 같은 frame을 참조하는 서로 다른 두 개 혹은 그 이상의 page가 있다면 넌 그걸 알아야 해. aliased frame에 access되면, accessed 랑 dirty bit는 하나의 page table entry에만(access한 page에만) update되어야 해. 다른 page table entry에는 update되면 안됨. 

 Pintos에선, 모든 user virtual page는 kernel virtual page에 aliased되어 있어. 이러한 aliase를 어떤 방법으로든 manage해야해. 예를 들어, 너의 코드는 accessed&dirty bits를 둘 다 check하고 update해야해. 또는, kernel은 user virtual address를 통해서면 user data에 access 하는 방식으로 이 문제를 피할 수 있어.

 다른 aliase는 sharing for extra credit의 구현이나 네 코드의 bug로 발생할 수 있어. ([VM Extra Credit]을 보렴)

 Section A.7.3 [Page Table Accessed and Dirty Bits]를 보렴.



#### 4.1.6  Managing the Swap Table

 swap table은 사용중이거나 free된 swap slot을 track한다. evicting page의 frame을 swap partition으로 넘기고 unused swap slot을 고를 수 있어야 해. 그리고 page가 read back되거나 process가 swap된 page가 종료되면 swap slot을 free해야해. 

 `BLOCK_SWAP` 이란 block device를 swap할 때 사용할 수 있으며, `struct block` 을 `block_get_role()` 로 호출해서 얻을 수 있어. `vm/build` directory에서, `$pintos-mkdisk swap.dsk --swap-size=n` 으로 `swap.dsk` 라는 이름의 n MB의 swap partition을 만들 수 있어. 그 다음엔,  `swap.dsk`가 자동으로 extra disk로 추가되어서 pintos가 실행됨. 아니면 `--swap-size=n` 을 사용해서 임시 n MB의 swap disk를 사용하도록 명령어 추가할 수도 있고.

 swap slot은 lazily하게 allocate되어야 해. 즉, 실제로 eviction이 필요할 때만 slot을 allocate해야 한다고. executable로 reading data page하거나 writing하기 위해서 process의 시작과 동시에 swap하는건 lazy한게 아니야. swap slot은 특정 page를 저장하도록 예약되어서도 안됨. 

 frame으로 다시 read back할 땐 swap slot을 free해야해.



#### 4.1.7 Managing Memory Mapped Files

 file system은 일반적으로 read나 write system call로 access됨. secondary interface는 **mmap** system call을 사용해서 file을  virtual page에 "map"하는거야. 이러면 프로그램은 file data에서 직접 memory instruction을 사용할 수 있어.

 file 'foo' 가 0x1000 bytes (4 kB, or one page) 라고 하자. 'foo' 가 주소 0x5000에서 시작해서 memory에 mapping되면 0x5000 부터 0x5fff에 모든 memory access가 'foo'의 해당 바이트에 엑세스함. 

 밑에는 **mmap** 을 사용해서 console에 file을 print하는 프로그램이야. 파일 열고, virtual address 0x10000000 에 mapping한 뒤에, mapped data를 console에 write하고, file을 unmap함.

```c
#include <stdio.h>
#include <syscall.h>
int main(int argc UNUSED, char *argv[])
{
  void * data = (void *) 0x10000000;	/* Address at which to map */
  
  int fd = open(argv[1]);				/* Open file. */
  mapid_t map = mmap(fd, data);			/* Map file */
  write(1, data, filesize(fd));			/* Write file to console */
  munmap(map);							/* Unmap file (optional). */
  return 0;
}
```

 전체 오류 처리 기능을 갖춘 비슷한 프로그램이 `mmcp` 의 두 번째 예제 인 `mcp.c` 를 포함하는`examples` 디렉토리에 `mcat.c` 로 포함되어 있습니다.

 제출 한 내용은 메모리 매핑 파일에서 사용되는 메모리를 추적 할 수 있어야합니다. 매핑 된 영역의 페이지 오류를 올바르게 처리하고 매핑 된 파일이 프로세스 내의 다른 세그먼트와 겹치지 않도록해야합니다.



### 4.2 Suggested Order of Implementation

 이 순서대로 구현하는걸 추천해요!

1. Frame table (Section 4.1.5 [Managing the Frame Table]을 보렴), `process.c`를 바꿔서 너의 frame table allocator로 실행되도록 바꾸자. 

   swap은 아직 구현하지마. 아직은 frame이 부족하면 allocator가 실패하거나 kernel이 패닉할거야.

   이 단계가 끝나면 kernel은 여전히 모든 플젝2 테스트 케이스 통과해야해. [플젝2에서는 swap이 필요한 경우가 하나도 없음!!]

2. Supplemental page table과 page fault handler (Section 4.1.4 [Managing the Supplemental Page Table]). executable을 load하고 stack을 setting할 때 supplemental page table에 필요한 정보를 record하려면 `process.c` 를 바꿔야해. page fault handler에서 loading code와 data segments를 구현하고. 일단 유효한 access만 고려해서 코딩하자.

   이 과정이 끝나면, kernel은 플젝2의 모든 기능적인 테스트 케이스만 통과할거야, 견고성 테스트만...

 여기부턴, stack growth와 mapped files[이건 3-2에서], 그리고 process exit에서 page reclamation을 동시에 구현할 수 있어. 

 다음엔 eviction을 구현하는건데 (Section 4.1.5 [Managing the Frame Table]을 보렴), 처음에는 evict를 randomly하게 하자. 이 시점에선, 어떻게 accessed&dirty bits를 관리하고 user와 kernel의 page를 aliasing할지 고려해야함. Synchronization또한 고려해야지. 예를 들어, process A가 page fault했는데, process 가 evicting한 frame이면 어떻게 처리해야 할지. 마지막으로 clock algorithm과 같은 ecivtion 정책을 구현해야함. 



### 4.3 Requirements

 이게 좀 열려있는 방식이라…하지만 OS가 다루어야할 기본적인 기능은 다뤄야 하겠지? page fault 처리, swap partition 구성, paging 구현 방식 등을 자유롭게 선택할 수 있어. 

#### 4.3.1 Design Document

[넘겨 ㅋ]

#### 4.3.2 Paging

 executable이 load된 segment에 대해 paging을 구현하자. 이러한 page들은 load를 lazily하게 해야함, 즉 kernel이 page fault를 할 때에만. [넹?] eviction엔, page가 load이후에 수정되면 (dirty bit로 표시된)swap에 write되어야 해. 수정되지 않은 page라면(read-only page를 포함해서), swap에 절대 write해서는 안되는데, executable로 다시 read back될 수 있기 때문이야. 

 LRU(least recently used)와 비슷하게 global page replacement algorithm을 구현해야해. 너의  algorithm은 적어도 "second chance"나 "clock" algorithm의 simple한 변형을 수행해야해. 

 그리고 parallelism을 허용해야함. 한 page fault가 I/O를 요구하면, fault되지 않은 process는 계속 실행되어야 하고, I/O를 필요로 하지 않는 다른 page fault는 완료될 수 있어야 해. 이건 synchronization 노력이 필요할거야. 

 `userprog/process.c`안에 `load_segment()` 인 program loader의 core를 수정해야해. loop를 `page_read_bytes` 가 excutable에서 read bytes를 받고 `page_zero_bytes` 는 읽은 byte 다음에 0으로 초기화 할 bytes를 받음. 이 둘의 합은 항상 `PGSIZE` (4096으로 설정되어있음) 이어야 해. page handling은 다음 변수의 값에 따라 달라지는데 :

- `page_read_bytes` 가 `PGSIZE`라면, page는 첫 access시에 underlying file에 의해 page가 요구되어야 해.
- `page_zero_bytes` 가 `PGSIZE` 라면, page가 disk로부터 읽어올 byte가 전혀 없단 뜻이야. 넌 첫 page fault에서 이 새 page를 모드 0으로 만들면서 handle해야함.
- 위의 두 가지 경우가 아리나면, page의 시작부분은 underlying file에서 read되고 나머지 부분은 0으로 하면 된다. 



#### 4.3.3 Stack Growth

플젝2에서는 한 page만 쓰고 그걸로 제한을 뒀는데, 이젠 stack이 현재 크기보다 커지면 추가적으로 page를 할당해야해. 

 stack access인 경우에만 추가 page를 allocate해. stack access를 다른 access와 구별해야 한단 말이지.

 일반적으로 실제 OS는 언제든지 stack에서 data를 push하는 방식으로 process를 interrupt할 수 있어서, user program은 stack pointer아래의 stack에 write를 하려고 하면 버그가 발생해. 그러나 80x86 architecture에서 **PUSH** instruction은 stack pointer를 조정하기 전에 access 권한을 확인하므로 stack pointer아래 4byte에서 page fault가 발생할 수 있어. (아니면, **PUSH**는 간단한 방식으로 다시 시작할 수 없어.) 비슷하게, **PUSHA** instruction은 32bytes를 push하므로 stack pointer아래 32byte에서 오류가 발생할 수 있어. 

 user program의 stack pointer를 알 수 있어야해. user program에 의해서 system call이나 page fault로 `syscall_handler()` 나 `page_fault()` 에 전달된 `struct intr_frame` 의 `esp` member로 알 수 있지. user pointer에 access하기 전에 확인하려면, 처리해야 할 유일한 경우야. [네?]반면에, page fault로 invalid memory access를 detect하는거에 의존하고 있다면, kernel에서 page fault가 발생하면 이 경우도 detect해서 handle해줘야 한다. exception이 발생해서 user mode에서 kernel mode로 switch되면 processor는 오직 stack pointer만 저장하니까, `struct intr_frame`에서  `esp`가 `page_fault()` 에 user stack pointer가 아닌 undefined value를 전달해줌. user mode에서 kernel mode로 처음에 전환될 때, `esp` 를 `struct thread` 에 저장하는 등의 다른 방법을 준비해야 함. 

 대부분의 OS처럼 stack size에 절대적인 limit가 있어야겠지? 일부 OS는 이런 limitation을 user가 조정할 수 있게 하는데 (많은 unix system에서 ulimit 명령을 사용해서). 많은 GNU/Linux system에서는 이런 limit을 8MB로 두고 있어.

 첫 stack page를 allocate하는건 lazily하게 할 필요는 없어. load time에 command line arguments로 allocate해서 초기화할 수 있으며, fault가 될 때 까지 기다릴 필요가 없어.

 모든 stack pages는 eviction의 희생양이 되어야 해. evicted된 stack page는 swap에 write되어야 함.



#### 4.3.4 Memory Mapped Files

memory mapped file을 구현하자, 다음 system call도 포함해서.

<u>System Call</u> : mapid_t **mmap** (int **fd**, void * **addr**)







#### 4.3.5 Accessing User Memory

 system call을 handle하는 동안 user memory에 access하도록 코드를 수정해야해(Section 3.1.5 [Accessing User Memory]를 보자). user process가 현재 file이나 swap space에 있는 page의 content를 access하는 것처럼, non-resident page를 참조하는 주소를 system call로 전달할 수도 있어. 게다가, kernel이 prevent하지 않는다면, page가 kernel code에 의해 access되는 동안에도 frame에서 evicted될 수 있어. kernel code가 non-resident user page에 access 하면 page fault가 발생해. 

 user memory에 access 하는 동안 kernel 은 이러한 page fault를 처리할 준비가 되어 있거나, 아예 발생하지 못하게 해야해. kernel은 이러한 fault를 handle 하기 위해 확보해야 하는 resource를 보유하고 있는 동안, 이러한 page fault를 방지해야 함. Pintos에서는, file system과 swap space를 포함한 device(s)를 컨트롤할 수 있는 device driver(s)가 요구되어지는 lock이 있어. [뭐라는거양아ㅏㅠㅏ] 예를 들어, device driver가 `file_read`로 pass되어진 user buffer에 access 할 때 page fault가 발생하는걸 허용해서는 안된다. 이러한 fault를 처리하는 동안 driver를 invoke할 수 없기 때문이야.

 이러한 page fault를 방지하려면 access occurs하는 code와 page eviction code가 서로 협력해야함. 예를 들어, frame에 포함된 page를 제거하지 않아도 되도록 frame table을 확장할 수 있어야 해. (이걸 "pinning" 혹은 "locking" the page in its frame이라고 함) Pinning은 page를 evict할 때 너의 page replacement algorithm의 선택을 제한하기 때문에, 필요한 만큼만 page를 pin해야 하고, 필요하지 않으면 pinning page하는걸 피해야해. 



### 4.4 FAQ

```
userprog/exception	| ?
vm/frame.c			| 162
vm/frame.h			| 23
vm/page.c			| 297
vm/page.h			| 50
vm/swap.c			| 85
vm/swap.h			| 11
```

[이건 본문 직접 읽으렴 ㅋ]



### A.6 Virtual Addresses

![스크린샷 2017-04-23 17.05.58](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-04-23 17.05.58.png)

<u>Macro</u> : **PGSHIFT**

<u>Macro</u> : **PGBITS**

 bit index(0) 과 각각 address의 offset부분의 비트(12) 수

<u>Macro</u> : **PGMASK**

 page offset내의 bit mask 1로 설정되어 있고, 나머지는 0으로 되어 있음. (`0xfff`)

<u>Macro</u> : **PGSIZE**

 page size in bytes (4,096)

<u>Function</u> : unsigned **pg_ofs** (const void * **va**)

 virtual address **va**의 page offset를 return

<u>Function</u> : uintptr_t **pg_no** (const void * **va**)

 virtual address **va**의 page number를 return

<u>Function</u> : void * **pg_round_down** (const void * **va**)

 virtual page의 시작점을 return한다. 그러니까 **va**의 offset을 0으로 한 거겠지?

<u>Function</u> : void * **pg_round_up** (const void * **va**)

 **va**를 반올림해서 가장 가까운 page의 경계를 return해준다. 

<u>Macro</u> : **PHYS_BASE**

 kernel virtual memory의 기본 주소. 기본값이 3GB(`0xc0000000`) 바꿀 수 있음. user virtual memory는 0부터 **PHYS_BASE** 이다. 

<u>Function</u> : bool **is_user_vaddr** (const void * **va**)

<u>Function</u> : bool **is_kernel_vaddr** (const void * **va**)

 각각 **va**가 user이나 kernel virtual address면 true, 아니면 false를 return한다.

<u>Function</u> : void * **ptov** (uinptr_t **pa**)

 physical address **pa** 에 대응하는 kernel virtual address를 return한다. **pa** 는 0부터 physical memory의 bytes 사이.

<u>Function</u> : uintptr_t **vtop** (void * **va**)

 **va**에 대응되는 physical address를 return할거야. **va**는 kernel virtual address여야 함.



### A.7 Page Table

[번역 귀찮아…원서 보자]
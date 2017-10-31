# 한다 번역 Pintos 한국어로

피드백 여기로 : prayanalog@gmail.com

[ ] <- 안에 있는 말들은 챕터 이름을 나타내는게 아니라면 내 마음대로 씨부린것임

함수 이름이나 명령어, directory는 대부분 `이 박스 안에` 넣었음.

불확실하거나 번역이 이상하거나 내가 이해하지 못한건 [?????????? ~~]로 물음표 10개 넣어둠.

이 문서는 매우 언프로페셔널 함. 그리고 말투도 이상함. 번역도 이상할 수 있음. 

오타 매우 많음. function을 functoin으로 쓴다던가 하는거 매우 많음. 피드백주면 고칠게여.

[TOC]

## 1 Introduction

 어서 오셈. Pintos는 80x86 architecture을 위한 간단한 OS 프레임웍이야. Kernel threads, user program 로딩과 실행, 그리고 file system을 지원해주는데, 엄청 간단하게만 구현되어있어. 이 Pintos 플젝으로, 너랑 너의 플메는 위에 세 가지 분야를 다 제대로 짜야 할거야. 그리고 virtual memory도 구현해야해.

 Pintos는 이론적으로 일반적인 IBM-compatible PC에서 돌아가(뭔 뜻인지는 알 필요 없을거야). 안타깝게도 CS330 학생들한테 이 컴터를 제공하는건 불가능해. 그래서 simulator위에서 할거야. **Bochs**랑 **QEMU** 시뮬을 쓸거야. Pintos는 VMware Player에서도 테스트되었어.

 이 플젝은 어려워. CS330은 시간 많이 걸리기로 악명이 높지. 그래서 많은 자료를 줘서 업무량을 최대한 줄여줄게. 물론 해야 할 일은 짱 많음. 과제의 불필요한 overhead를 줄이는 방법이 생각나면 조교님이나 교수님께 제안하자. 

 이 1 챕터에서는 Pintos를 어케 실행해야 할지 설명해놨어. 플젝 시작하기 전에 다 읽고 이해하고 시작해.



### 1.1 Getting Started

 시작하려면 Pintos가 build될 수 있는 기계에 접속해야해. 그냥 CS330에서 제공하는 서버로 들어가자.서버에 접속하는 방법은 조교님이 klms에 올려주셨음. 

그리고 PATH를 설정할건데, 만약 팀이 1000번이고 a플메라면 접속을 

`$ssh -p 10XXX 1000a@143.xxx.xxx.xxx` 로 서버에 접속했을거야.

그럼 PATH 설정은

```
$PATH=$PATH:/home/1000a/pintos/src/utils
```

로 해주면 된다. 정확히는 `utils/` 의 directory로 해주면 되는거야. 나중에 build하면 저기에 pintos 실행 파일이 있거든. 

[서버 접속 예시를 들어주면 위험할 수 있으니 klms에 올려주신 접속 방법을 참고하자. 봐도 모르겠다면 빠른 도움 요청 혹은 드랍…을...]

[나머지는 볼 필요 없다]

#### 1.1.1 Source Tree Overview

 접속을 했다면 밑에 방법으로 Pintos를 설치하자. klms에 올려주신 방법으로 설치해도 무관하다.

```
$wget http://www.stanford.edu/class/cs140/projects/pintos/pintos.tar.gz
$tar xzf pintos.tar.gz
```

이제 어떤 것들이 있는지 `$cd pintos/src` 로 가서 살펴보자.

**`threads/`**

프로젝트 1에서 시작해서 수정할 kernel의 기본 소스 코드다.

**`userprog/`**

프로젝트 2에서 시작할 user program loader의 소스 코드다.

**`vm/`**

거의 비어있는 directory다. 프로젝트 3에서 virtual memory를 구현할거다.

**`filesys/`**

fily system의 기본 소스 코드이다. 프로젝트 2에서 사용하지만, 프로젝트 4까지는 수정 안해.

**`devices/`**

키보드, 타이머, 디스크 등 I/O 장치의 interfacing을 위한 소스 코드다. 프로젝트 1에서 타이머 구현을 수정할거야. 그 때가 끝이고 다른 때는 안건드림.

**`lib/`**

표준 C 라이브러리의 subset이야. Pintos kernel(프로젝트 2 초반)이랑 user program에서 컴파일될거야. 이 directory안에 있는 파일들은 `#include <…>` 표기로 사용할 수 있어.

**`lib/kernel/`**

Pintos kernel에만 포함된 C 라이브러리야. bitmap, doubly linked list나 hash table같이 kernel code에서 맘대로 쓸 수 있는 data type들이 있어. 위와 마찬가지로 `#include <…>` 표기로 사용할 수 있어.

**`lib/user/`**

Pintos user program에만 포함된 C 라이브러리야. 위와 마찬가지로 `#include <…>` 표기로 사용할 수 있어.

**`tests/`**

각 프로젝트를 테스트해. 수정해서 사용할 수 있긴 한데, 조교님들이 채점할 땐 원본으로 바꿔서 채점하실거임.

**`examples/`**

프로젝트 2를 시작할 때 user program의 예제들이야

**`misc/`**

**`utils/`**

조교님이 제공해주는 서버말고 개인적으로 machine돌려서 할 거면 필요할거야. 걍 무시하자



#### 1.1.2 Building Pintos

 첫 프로젝트를 위해 build하자. 일단 `os-pintos/pintos/src/threads/` 로 들어가자. 그리고 `$make` 쳐보자. `build/` 라는 directory가 생기고 `Makefile` 이랑 몇 개의 subdrictory가 생길거야. 30초 안에 안되면 문제가 있는거야. [다른 machine에서 돌린건 직접 원서를 보거나 문서를 찾아보자]

`build/` 안에 있는 파일들을 보자



**`Makefile`**

`pintos/src/Makefile.build` 의 복사본이야. kernel을 build하는 방법을 설명해놨어. 자세한 내용은 [Adding Source Files]를 참고해.



**`kernel.o`**

전체 kernel에 대한 object file임. 개별 kernel 소스 file에 컴파일된 single object file이야. 디버그 정보가 포함되어 있어서, GDB(Section E.5 [GDB] 보셈)나 **backtrace**(Section E.4 [Backtraces] 보셈)를 실행할 수 있어.



**`kernel.bin`**

kernel의 메모리 이미지인데, Pintos kernel을 실행하기 위해 메모리에 load된 정확한 바이트들이야. 위의 kernel.o에서 디버그 정보를 제거한 거라 많은 공간을 절약할 수 있어. 그래서 kernel loader의 디자인에 의해서 512kB 크기 제한을 뒀지 [뭔소린지 모르면 넘어가자]



**`loader.bin`**

kernel loader의 메모리 이미지인데, disk에서 kernel로 kernel을 읽어서 시작하는 assembly 언어로 작성된 작은 코드야. 512 bytes로 고정됨.



`build/` 의 하위 directory에는 컴파일러가 생성한 object파일('.o')이랑 dependency파일('.d') 있지? dependency파일은 header파일이 바뀌면 어떤걸 re컴파일 해야 하는지 결정하는 것들이야.



#### 1.1.3 Running Pintos

 조교님들이 Pintos 편하게 하라고[아니 이래도 어려운건 마찬가지임] 서버 제공 다 해주심. 일단 가장 쉬운 걸 방식은 `pintos argument …` 로 Pintos를 실행하는 방법이 있어. 각 argument는 Pintos kernel이 처리할 수 있도록 전달될거야.

 우선 `$cd build/` 로 'build' directory에 들어가보자. 

```
$pintos run alarm-multiple
```

라고 해보면, pintos 뒤의 `run alarm-multiple` 부분이 argument로 Pintos kernel에 전달되는거야. 여기서 `run` 은 kernel에서 test를 실행하자는 말이고, `alarm-multiple` 은 어떤 test인지 알려주는 거지.

[ `-bash: pintos: command not found` 같은 말이 나오면 PATH설정을 안한 거야. [1.1 시작하자] 부분을 다시 보렴 ]

저 위에 실행으로 Bochs를 실행하고 `bochsrc.txt` 라는 파일을 만들었을거야. BIOS 페이지처럼 나와서 메세지가 쭈르륵 나올거야.[라고 써있지만 `Error opening terminal: xterm-256color.` 같은거 보이면서 아무 일도 일어나지 않았으면 "http://stackoverflow.com/questions/6804208/nano-error-error-opening-terminal-xterm-256color" 를 참고하자] 그리고 Pintos는 alarm-multiple 테스트 프로그램을 실행해서 몇몇 text를 출력했을거고. [나머지는 필요 없는 부분이야. Bochs 창이 켜졌는데 끄라는 말임]

 Pintos가 실행되면서 VGA display랑 fist serial port에 output을 보냈기 때문에 일어난 일이야. 

```
$pintos run alarm-multple > logfile
```

로 serial output를 로그파일로 저장할 수 있어.

 Pintos 프로그램은 시뮬레이터나 virtual 하드웨어를 구성하기 위해 몇가지 옵션을 제공해.

```
$pintos run options --argument...
```

처럼 쓰면 돼. Bochs가 기본값이지만 `--qemu` 로 QEMU로 시뮬레이팅 할 수 있어. [조교님이 QEMU는 쓰레기라 그러셨다.] 

디버거(Section E.5 [GDB]를 봐)를 사용해서 돌릴 수도 있어. 메모리 제한을 할 수도 있고말야.

마지막으로 `-v` 옵션으로 VGA 디스플레이를 끄거나, `-t` 로 터미널을 안열고 Bochs만 열거나[`-t`는 필요없는 옵션이다. 왜냐면 우린 서버에서 할거니까.] `-s` 로  serial input을 무시하고 stdin으로 인풋, stdout으로 아웃풋을 낼 수 있어.

Pintos kernel에는 `run` 말고 다른 명령어랑 옵션이 있는데, 앞으로는 흥미를 가지길 바란다.[ㅋ] 

`-h` 옵션으로 옵션 리스트를 볼 수 있어. 그러니까 `$pintos -h` 를 말하는거임.



#### 1.1.4 Debugging versus Testing

 코드 디버깅 할 때 프로그램을 두 번 돌려봐서 같은 동작을 하는지 보는 방법이 있어. 두 번 혹은 그 이상 돌려봤는데 이전에는 못 봤던걸 볼 수도 있거든. 이런걸 재현성(reproducibility)이라고 해. Pintos가 기본적으로 Bochs가 재현성(reproducibility)을 보여주도록 설정되어 있을거야. 

 물론 같은 시간에 같은 인풋으로만 재현성이 보여질 수도 있지. 그래서 이런 재현성을 보려면 컴퓨터의 모든 부분이 같아야 해. 동일한 command-line argument, disks, Bochs 버전, 그리고 아무 키도 누르면 안돼. (정확히 같은 시간/지점에서 그 키를 누를 수 없을테니까)

 재현성은 디버깅에 유용하긴 한데, thread synchronization을 테스팅 하는 것 등 대부분 프로젝트에 문제가 될거야. 특히, Bochs가 재현성을 보기 위해 설정된다면, timer interrupts는 완벽하게 재현가능한 부분에서 발생할거고, thread switch가 될거야. 그러니까 동일한 테스트를 여러번 한다고 해서 코드가 맞게 돌아간다고 말할 수는 없는거야.

 [하지만 이대로 안된다고 하면 노잼이잖아] 그래서 Bochs에 "jitter"라는 기능이 있지. 

> jitter는 안절부절못하다, 신경질적으로 행동한다는 뜻과 조금씩 움직인다는 뜻이 있어. 
>
> 네이버 IT용어사전에는 ''이상적인 기준(reference point)으로부터의 시간 변위. 신호가 기준점보다 얼마나 빨리 혹은 늦게 나타나는가를 표현하는 값''이라고 말하네

timer interrupt는 random한 간격으로 발생하지만[그렇게 보이는 것 뿐], 완벽하게 예측할 수 있어. Pintos를 `-j seed` 라는 옵션으로 돌리면, timer interrupt는 불규칙적인 간격을 가질거야. 

[seed가 seed글자가 아니라 숫자를 쓰는 것 같다] 

이 옵션만 있고 다른 옵션을 포함하지 않고 돌리면 재현성이 있을거야, 물론 seed 값 변화에 따라 timer가 다르게 동작할거야. [이건 대체 무슨 소리인지 모르겠다ㅠ] 그래서 신뢰할만한 결과를 얻으려면 여러 seed값으로 코드를 테스트해야해.

 반면 Bochs가 재현성 버젼으로 실행되면, time이 비현실적일 수 있어. 뭔 말이냐면 현실의 1초가 시뮬레이터 상에서 1초가 아니라 더 짧거나 길 수 있단 말이야. `-r` 옵션으로 pintos 돌리면 Bochs를 현실 시간처럼 돌아가게 할 수 있어. 하지만 Bochs의 시뮬레이터 상에서는 1초가 아닐 가능성이 높아. [무슨 말이냐면 현실에서 1초로 기준으로 돌아가도록 설정하려면 `-r` 을 쓰지만 Bochs 상에서는 1초가 아닐 수 있고, Bochs 시뮬레이터 위에서 1초 기준으로 돌아가도록 설정하려면 `-j seed` 를 쓰라는거지. 물론 현실에서의 1초가 아니겠지만. ] 따라서 옵션 `-j` 랑  `-r` 는 상호 배타적이야. 

 QEMU 시뮬레이터를 Bochs대신 사용할 수 있어. `--qemu` 옵션을 쓰면 된다구. Bochs보다는 훨씬 빠르지만 실시간 시뮬레이션만 지원해서 재현성을 볼 수 없어. 



### 1.2 그렇담 점수는?

 [일단 인터넷 코드 복붙하지 말자. F 받는다..]

 [pintos.pdf 기준이긴 하지만] 테스트 결과와 디자인 quality가 각각 50%씩 차지할거야



#### 1.2.1 테스트?

 점수가 테스트로 될거야. 각 프로젝트마다 몇개의 test가 있는데, 'tests'라는 문자로 시작하는 이름들이야. 제출을 완료하려면 `build/` directory에서  `$make check` 를 해봐. 각 테스트를 build해서 'pass' 랑 'fail' 메세지 보여줄거야. 테스트가 실패하면 왜 실패했는지도 나올거고. 테스트가 끝나면 `$make check` 로 테스트 결과를 요약해서 볼 수 있어.

 프로젝트 1의 경우에는 Bochs에서 아마 더 빠를거야. 그런데 나머지 프로젝트는 QEMU에서 훨씬 빠르게 됨. `$make check` 는 기본적으로 빠른 시뮬레이터를 선택하지만 `SIMULATOR = --bochs` 나 `SIMULATOR = --qemu` 를 옵션으로 넣어서 우리가 선택할 수 있지.

 물론 테스트를 하나씩 돌려볼 수 있지! t라는 테스트의 결과는 `t.output` 으로 저장돼고, script가 `t.result` 에 'pass' 나 'fail' 을 써둠. 테스트 하나를 돌리고 결과 보고 싶으면 `build/` directory에서  `.result` 파일을 `make` 해. 그러니까 `$make tests/threads/alarm-multiple.result` 처럼 하라고. 만약에 `make` 가 결과가 up-to-date[그니까 최신판이래]라고 해도, 그 `.output` 파일을 지우던가 `$make clean` 으로 돌리면 됨.

 기본적으로, 각 테스트는 완료시에만 피드백을 주고 돌아가는 도중에는 안줘. 하지만 돌아가는 도중에 받고 싶다면 `VERBOSE=1` 를 `make` 옵션에 추가해. `$make check VERBOSE=1` 처럼 말야. 진행 상황을 이런 저런 옵션에서 보고 싶으면 `PINTOSOPTS='...'` 옵션을 추가하자. `$make check PINTOSOPTS='-j 1'` 는 jitter 의 seed를 1로 해주는거야. 이것저것 임의의 옵션은 저기에 추가하면 된다구. (Section 1.1.4 [디버깅이냐 테스트냐!(Debugging versus Testing)]을 보자)

 모든 테스트 관련 파일들은 `pintos/src/tests` 에 있어. 쫌 위에서 말했듯이 조교님들이 채점하기 전에는 수정없는 초기 test 파일들로 할거야.

 모든 소프트웨어에 버그가 있을 수 있어. 그래서 test도중에 실패할 수 있는데 우리가 짠 코드의 버그가 아니라고 생각되면 조교님께 알려드리자. [그런 일은 일어나지 않을것이다. 아마도…] 조교님이 고쳐주실거임. 

 물론 테스트만 돌아가게 억지로 끼워맞춰서 코딩하지 마렴. [학점 산산조각난다]



#### 1.2.2 디자인! 

 이거 50%점수 있다. 여기에 시간 많이 할애해야함.

##### 1.2.2.1 디자인 Document



**Data Structures**

 이 부분은 항상 동일해. 수정했거나 새로운 struct나 struct member, global or static 변수, typedef, 혹은 enumeration을 썼다면 여기(Data Stuctures)에 25자 이하로 명시해줘.

 당연히 각 수정사항이나 새로 추가한 부분에 주석(comment)을 넣는 건 잊으면 안돼. [주석을 안쓰면 그게 코드냐] 그리고 Data Structure에 새로운게 추가되었거나 수정이 있다면 간단하게 25자 이하로 목적을 설명해줘. 



**Algorithms**

 코드 작동이 어떻게 되는지 알려줘야지, 질문을 통해서 너가 코드를 이해하는지 알아볼거야. 대부분 OS문제에 매우 많은 창의적 해결 방법이 존재해서 조교님이 코드만 보고 쉽게 이해를 못 할 수도 있으니 자세하게 설명을 써주자. 

 여기의 답변은 주어진 요청사항의 말 수준을 그대로 쓰면 안된다. 무슨 말이냐면, 과제 내용이 'A를 해서 B를 완성하세요' 인데 여기 Algorithms 부분에다가 'A를 해서 B를 완성했다' 처럼 적지 말라는 말이지. 적어도 'A를 C로 한 다음 D로 해서 B를 완성했다' 같은 말로 풀어서 쓰라는 말이야. 그리고 반대로 코드수준으로 적지도 말고. '이 줄에서는 뭘 했다. 다음 줄에서는 뭘 했다.' 가 아니라 '이 부분은 어떤 input에 어떤 output을 준다. 이러한 알고리즘이다.' 정도로 써달라는 말이겠지?



**Synchronization**

 OS kernel은 복잡하고, multitrheaded program이라 여러 threads를 synchronize하는게 어렵겠지. 여기에다가는 특정 타입의 행동을 어떻게 synchronize한 건지 적으면 될거야.



**Rationale [이론적인 이유]**

 위에 것들은 뭘 어떻게 였는데, 여기서는 왜? 라는 질문에 대답을 하면 돼. 이렇게 짜는게 왜 정당한지, 다른 방법보다 좋은 이유가 뭔지. rough한 time & space complexity를 첨가해서 써주면 좋을거야. 



 잘 해보자. 맞춤법 틀리지 말고. Appendix D [Project Documentation]를 참고해.



##### 1.2.2.2 소스 코드

 조교님들이 소스코드를 물론 볼거야. `$diff -urpb pintos.orig pintos.submitted` 같은 걸로 비교하실거야. 그리고 디자인 Document랑 코드랑 최대한 매칭하며 이해하실거임. [조교님들 화이팅..] 그니까 매칭이 완전 안되면 점수 막 깎이는거지.

 소스 코드 디자인의 가장 중요한 부분은 OS가 가지고 있는 문제들을 얼마나 잘 처리했느냐의 여부겠지? 예를 들어 프로젝트 4 : File system에서 inode의 구성은 file system 디자인의 중요한 부분인데 제대로 디자인&코딩 하지 않으면 점수가 깎인다고. [당장 inode가 뭔지는 이해할 필요 없음] 별로 안 중요한 부분도 있는데. multiple Pintos design 문제 'priority queue' 처럼 dynamic collection에서 빨리 extract(추출) 되는게 중요하긴 하지만, 너가 아무리 쩌는 data structure로 performance를 향상시켜도 조교님은 그거에 별 관심이 없으실거임. 그니까 걍 linked list쓰셈. (Pintos는 최대최소값 찾고 sorting해주는 편리한 함수가 있으니까)

------

[조교님이나 교수님이 아니시라면, 필요 없는 부분인 것 같네여.법적인 문제나 감사의 말 혹은 사소한 것.. ]

### 1.3 Legal and Ethical Issues

### 1.4 Acknowledgements

### 1.5 Trivia

 이름을 왜 Pintos로 지었게? 1. pinto 콩은 나쵸처럼 멕시코 음식임. 2. pintos는 작고, 'pint'라는 말은 적은 양을 말함. 3. 최초의 차를 운전하는 것처럼, 학생들이 폭발로 문제를 가지는 것 처럼 보여서.?? [뭐라는거야 ㅡㅡ]

------



## 2. 프로젝트 1 : Threads

 이 과제에서는 최소한의 기능을 가진 thread 시스템을 이미 제공하고 있어. 우리는 synchronization 문제들을 더 잘 해결하도록 이 시스템의 기능을 확장시키면 되는거야. 

 대부분 `threads/` directory에서 작업할거긴 한데, `devides/` directory에서도 조금 할 게 있어. 그리고 반드시 `treads/` directory에서 컴파일 해야해.

 이 chapter 읽기 전에 chpater1, Appendix C, E, F, 적어도 Sectino A.1~5, 특히 A.3은 보고 오자. 그리고 프로젝트를 끝내기 위해서는 Appendix B도 읽어야 해. [꼭 읽고 시작하자. 제발. 마음이 급해도 하다가 모르는 것들 투성이고 다 저기에 있을거야]

### 2.1 Background

#### 2.1.1 Understanding Threads

 일단 주어진 thread system코드를 읽고 이해하자. Pintos는 이미 thread 생성과 completion, 그리고 single scheduler to swtich between thread, synchronization primitives(semaphore, locks, condition 변수, optimization barrier)를 다 만들어 놨어.

 이 코드들은 좀 이상하게 보일 수 있어. base system을 돌려서 컴파일을 아직 안했으면, 하렴. (chapter 1[Introduction]을 보면 됨) 니가 원하면 `printf()`를 넣어서 컴파일하고 실행해서 발생한 일의 순서를 확인할 수 있어. 또 디버거에서 breakpoint를 설정해서 디버깅 할 수도 있어.

 thread가 생성되면, 새 context를 scheduled되도록 만들어야해. `thread_create()` 의 argument로 context에서 실행될 함수를 제공해야 해. 처음으로 thread가 scheduled되고 실행되면, 그 thread는 함수의 처음 부분부터 시작해서 context안에서 실행될거여. 함수가 return하면, thread가 종료돼. 각 thread는 `thread_create()`에 상속된 함수들은 `main()` 같이 Pintos 안에서 mini-program처럼 실행되는거야. 

 어떤 주어진 시간에, 오직 하나의 thread만 실행되고 나머지 thread(가 있다면)는 비활성화 되어 있어야해. scheduler가 어떤 thread가 다음에 실행될 지 결정하지. (만약 주어진 시간에 실행되길 기다리는 ready된 thread가 없다면, `idle()`에 구현된 "idle" thread[특이한 놈임]이 실행되도록 해야함.) Synchronization primitives는 한 thread가 다른 thread가 뭔갈 하도록 기다려야 할 때 context switch할 수 있어. 

 context switch의 메카니즘은 80x86 어셈블리 코드이고 `threads/switch.S`안에 있어.(이해할 필요는 없다네. 하하! 당연하지…어셈블리 코드 이해하라 하면 미친 플젝이지…) 실행중인 thread의 현재 state를 저장하고 switching할 thread의 state를 restore해줌.

 GDB 디버거를 쓰렴!. 천천히 context switch를 추적해서 어떤 일이 일어나는지 보여줄거야. 시작하기 전에 `schedule()` 에 breakpoint를 설정해서, 한 단계씩 실해할 수 있어. thread의 address와 staet를 계속 추적해서, 각 thread의 call stack위에 어떤 procedure가 있는지 확인해. 한 thread가 `switch_threads()` 를 호출할 때, 그니까 어떤 다른 thread가 실행되기 시작할 때, 그 새 thread가 처음 `switch_threads()` 에서 return되는걸 알려줘야해.  thread system을 이해하고, 왜 그리고 어떻게 호출된 `switch_threads()`가  `switch_threads()` 가 return한 거랑 다른지 이해할거여. Section A.2.3 [Thread Switching]을 봐.

 **Warning** : Pintos에서 각 thread는 4kB미만의 작고 고정된 실행 스택을 할당받아. kernel은 stack overflow를 감지하려고 시도할테지만, 완벽하지는 않아. `int buf[1000]` 같은 non-static 변수처럼 large data structure를 사용하면 큰 문제가 발생하고 kernel이 panic할거야. stack allocation은 page allocator과 block allocator를 포함하는게 방지방법이야. (Section A.5 [Memory Allocation]을 봐)



#### 2.1.2 Source Files

 `threads` directory에 대한 간략한 개요…가 있어여…대부분 수정할 필요가 없긴 한데, 시작하려면 어떤 코드를 봐야 할지 대충 알려줍니다.

**`loader.S`**

**`loader.h`**

 kernel loader임. 512 bytes의 코드와 데이터고 PC BIOS가 disk위 kernel을 찾아서 momory에 load한 뒤, **`start.S`**에 `start()` 로 점프한다. Section A.1.1[Pintos Loader]를 보면 되는데 **이 코드 보거나 수정할 필요 없음.**

**`start.S`**

 기본 설정은 32bit의 80x86 CPU와 memory protection이 필요하다. loader랑은 다르게 kernel의 실제 부분 코드이다. Section A.1.2 [Low-Level Kernel Initialization]을 봐.

**`kernel.lds.S`**

 kernel에 연결하기 위해 쓰이는 linker script임. kernel의 load address를 설정히거 **`start.S`** 를 위해 kernel image의 시작부분에 위치시킨다. Section A.1.1 [Pintos Loader]를 봐. **얘도 수정이나 볼 필요 없음**

**`init.c`**

**`init.h`** 

 `main()` 을 포함해서 kernel을 초기화한다. kernel의 "main program"이지. 어떤게 초기화되었는지 확인하기 위해서 적어도 `main()`을 봐야해. 아마도 너의 초기화코드를 여기에 포함할거야. Section A.1.3 [High-level Kernel]을 봐.

**`thread.c`**

**`thread.h`**

 기본적인 thread를 지원해줘. 너의 대부분의 작업이 여기서 실행될거야. **`thread.h`** 는 `struct thread` 를 정의하고(너가 모든 프로젝트에서 수정해야 할 거임) 있어. Section A.2 [Threads]를 봐.

**`switch.S`**

**`switch.h`**

 switching threads를 위한 어셈블리 언어 루틴임. 위에서 말 했어. Section A.2.2[Thread Functions]를 봐

**`palloc.c`**

**`palloc.h`**

 4kB 배수의 page를 넘겨주는page allocator야. Section A.5.1[Page Allocator] 봐

**`malloc.c`**

**`malloc.h`**

 kernel을 위한 `malloc()` 과`free()`의 간단히 구현해놨어. Section A.5.2 [Block Allocator]를 봐.

**`interrupt.c`**

**`interrupt.h`**

 기본적인 interrupt handling과 interrupt를 키고 끄는 함수들이야. Section A.4 [Interrupt Handling]을 봐

**`intr-stubs.S`**

**`intr-stubs.h`**

 low-level interrupt 처리를 위한 어셈블리 코드야. Section A.4.1 [Interrupt Infrastructure]봐

**`synch.c`**

**`synch.h`**

 기본 synchroniation 요소야. semaphore, locks, condition변수, optimization barriers가 있지. 네 개의 플젝에서 다 써야해. Section A.3 [Synchronization]봐

**`io.h`**

 I/O port를 access할 함수야. ` devices` directory에 있는 코드에서 쓰니까 건들 필요 없어

**`vaddr.h`**

**`pte.h`**

 가상 address와 page table 항복들의 함수와 매크로야. 플젝3에서 쓸거니까 지금은 무시하자.

**`flags.h`**

 80x86 "flags" register로 몇 비트의 매크로야. 별 필요 없음.



##### 2.1.2.1 'devices' code

 기본 threaded kernel은 `devices` directory에서 이 파일들을 포함해.

**`timer.c`**

**`timer.h`**

 system timer는 기본적으로 초당 100번 tick함. 이 플젝에서 코드를 수정할거야.

**`vga.c`**

**`vga.h`**

 VGA display driver임. screen에 텍스트 쓸 때 필요해. 이거 볼 필요는 없긴 한데 `printf()` 가 VGA display driver을 부를거야. 그래서 너 코드에서 가끔 얘들을 부를 필요가 있어.

**`serial.c`**

**`serial.h`**

 serial port driver야. `printf()`가 이 코드를 호출할텐데, 너가 뭘 할건 없음. serial input을 input layer로 전달해서 처리할거야. (밑에 보셈)

**`block.c`**

**`block.h`**

  block devices를 위한 abstraction layer이야. 고정 크기 block의 array로 구성된 random-access disk형태의 devices야. Pintos는 두 타입의 block device를 지원하는데, IDE disk 과 partition임. 플젝2까지는 사용 안할거임.

**`ide.c`**

**`ide.h`**

  최대 4개의 IDE disk에서 읽기와 쓰기 지원

**`partition.c`**

**`partition.h`**

 disk의 partition structure을 이해하여, 단일 disk를 독립적으로 사용하도록 여러 부분(partition)으로 조각낼 수 있다.

**`kbd.c`**

**`kbd.h`**

 keyboard driver. 키 입력을 input layer로 전달함

**`input.c`**

**`input.h`**

 input layer. 키보드나 serial driver가 전달한 입력 문자를 queue에 넣는다

**`intq.c`**

**`intq.h`**

 kernel thread와 interrupt handler가 access하려는 순환 queue를 관리하기 위한 interrupt queue.키보드나 serial driver에서 쓴다.

**`rtc.c`**

**`rtc.h`**

 real-time clock driver. kernel이 현재 날짜와 시간을 결정할 수 있게 한다. 기본적으로 얘는 `thread/init.c` 가 random number을 생성할 때 초기 seed선택하는데만 사용됨.

**`speaker.c`**

**`speaker.h`**

 PC speaker에서 tone을 생성할 수 있는 driver

**`pit.c`**

**`pit.h`**

 8254 Programmable Interrupt Timer 코드다. 이 코드는 각 장치가 PIT의 출력 채널 중 하나를 사용하기 때문에 `devices/timer.c` 와 `devices/speaker.c` 에서 모두 사용된다.



##### 2.1.2.2 'lib' files

 마지막으로 `lib` 과 `lib/kernel` 은 쓸만한 라이브러리 루틴을 포함하고 있어. (`lib/user` 는 플젝 2에서 시작하는 user program에서 사용될거지만 kernel의 일부는 아님). 여기 몇 가지 세부사항이야 :

**`ctype.h`**

**`inttypes.h`**

**`limits.h`**

**`stdarg.h`**

**`stdbool.h`**

**`stddef.h`**

**`stdint.h`**

**`stdio.c`**

**`stdio.h`**

**`stdlib.c`**

**`stdlib.h`**

**`string.c`**

**`string.h`**

 C 라이브러리의 subset이다. 첨 보는거 있으면 Section C.2 보셈.

**`debug.c`**

**`debug.h`**

 디버깅을 돕기 위한 함수와 매크로. Appendix E [Debugging Tools] 참고해

**`random.c`**

**`random.h`**

 Pseudo 난수 생성기. 랜덤 값의 실제 순서는 세 가지중 하나를 수행하지 않는 한 Pintos의 실행마다 달라지지 않아. 각 실행에서 `-rs` 는 새로운 랜덤 시드 값을 지정하거나 pintos에서 `-r` 옵션을 지정한다. 

**`round.h`**

 반올림 매크로

**`syscall-nr.h`**

 system call number. 플젝2까지는 안씀.

**`kernel/list.c`**

**`kernel/list.h`**

 doubly linked list 구현함. Pintos 코드 전체에서 사용되며, 플젝1에서 직접 몇 군대를 사용할거야.

**`kernel/bitmap.c`**

**`kernel/bitmap.h`**

 비트맵 구현. 이 코드는 플젝1에서 안필요할거야.

**`kernel/hash.c`**

**`kernel/hash.h`**

 해시 테이블 구현. 플젝 3에서 쓸거야. 

**`kernel/console.c`**

**`kernel/console.h`**

**`kernel/stdio.h`**

 `printf()`외 다른 함수 구현했음. 



#### 2.1.3 Synchronization

 적절한 synchronization은 이러한 문제의 해결의 중요한 부분이야. 모든 synchronnization 문제는 interrupt를 끄면 해결된다. concurrency가 사라지므로. 하지만 이 방법은 쓰지 마렴. semaphore, lock, condition 변수를 쓰자. 

 Pintos 프로젝트들에서, interrupt를 비활성화 해서 해결할 수 있는 문제의 클래스는 kernel thread와 interrutp handler사이에 공유되는 데이터를 조정하는 거야. 이 플젝에서는 interrupt handler에서 thread state의 몇 bit에 대한 access하는게 필요해. alarm clokc의 경우, timer interrupt는 sleeping thread를 깨울 필요가 잇어. advanced scheduler에서, timer interrupt는 몇 global and per-thread 변수에 access해야해. 이 변수들을 kernel thread에서 접근하려 한다면, imter interrupt가 간섭할 수 없도록 interrupt를 비활성화 해야 해. 

 interrupt 를 끌 때, 가능한 적은 양의 코드를 처리하도록 주의하자. 그렇지 않으면 timer tick이나 input event같은 중요한 사항을 손실할 수 있어. interrupt를 끄면 , interrupt handling 기간이 길어져 시스템이 느려질 수 있어.

 synchronization primitives는 `synch.c` 안에 interrupt를 비활성화 하면서 구현된다. 넌 코드의 양을 늘려야 할 수 있지만, 최소한으로 유지하도록 하자.

 interrupt를 비활성화 하면 코드 섹션이 interrupt되지 않았을 때, 디버깅에 유용해. 플젝이 시작될 때 딥깅 코드를 제거하렴. (코드를 읽기 어려울 수 있으니, 그냥 주석처리만 하지 말고..)

 제출할 때 busy waiting이 없어야 해. `thread_yield()` 를 작은 loop에서 부르는 것도 busy waiting이야.



#### 2.1.4 Development Suggestions

 제출 직전에 막 짜지 말렴…그리고 문서 다 읽어. 디버깅 도구도 잘 쓰고..



### 2.2 Requirements

#### 2.2.1 Design Document

 샘플 디자인 문서 Appendix D [Project Documentation]에 있으니까 꼭 읽자. `pintos/src/threads/DESIGNDOC` 에 플젝1 디자인 Document 뎀플릿을 만들고 채워야 해. 



#### 2.2.2 Alarm Clock

 `devices/timer.c`에 있는 `timer_sleep()` 을 다시 구현해야해. 비록 "busy waits"로 작업 구현은 되어있지만, 루프에서 돌다가 시간이 다 되면 `thread_yield()` 를 호출해. busy wait를 피해서 재구현하라는 거야. 

<u>Function</u> : void **timer_sleep** (int64_t **ticks**)

 시간이 적어도 x timer tick만큼 진행될 동안 호출 thread의 실행을 일시 정지해. 시스템이 유휴상태가 아닌 이상, thread는 정확히 x tick이후에 깨어날 필요가 없어. 적당한 시간 동안 기다린 뒤에 ready queue에 집어넣어.

 `timer_sleep()` 은 real-time으로 작동하는 유용한 thread다. (초당 커서 한 번 깜빡이도록 하는 것처럼)

 `timer_sleep()` 의 argument는 timer tick으로 표시된다. `devices/timer.h` 에 정의된 `TIMER_FREQ`라는 초당 timer ticks이 있어. 기본값은 100이야. 이거 변경하면 많은 test가 실패하니까 최대한 변경하지 않는게 좋아.



`timer_msleep()`, `timer_usleep()`, 그리고 `timer_nsleep()` 은 각각 밀리, 마이크로, 나노 초의 특정 시간 동안 sleep하도록 하지만, 필요하면 `timer_sleep()`을 호출한다. 수정할 필요는 없음.

 delay가 너무 짧거나 길면 `-r` 옵션을 읽어.

 alarm clock 구현은 나중에 필요 없을테지만, 플젝4에 쓸 수도 있음. 



#### 2.2.3 Priority Scheduling

Pintos 에 priority scheduling을 구현하자. thread가 현재 실행중인 thread보다 높은 priority를 가지고 있으면 즉시 양보해야 한다. 마찬가지로 thread가 lock, semaphore, 혹은 condition 변수를 기다리고 있으면 priority가 가장 높은 waiting thread가 처음으로 깨나야 한다. thread가 언제나 자신의 priority를 올리거나 내릴 수 있지만, 가장 높은 priority를 가진 thread가 낮추면 즉시 CPU에게 권한을 넘겨줘야 한다.

 thread priority는 `PRI_MIN(0)` 에서 `PRI_MAX(63)` 까지 있다. 낮은 숫자가 낮은 priority를 가지니까 0이 가장 낮은 priority임. 최초의 thread의 우서 순위는 `thread_create()` 의 argument로 받는다. 다른 우선 순위를 선택할 이유가 없으면 `PRI_DEFAULT(31)`을 사용해. 이 `PRI_` 매크로는 `threads/thread.h` 에 정의되어 있으니까 값을 변경하면 안됨.

 prioirty scheduling 의 한 이슈는 "priority inversion"이다. H, M, L을 각각 높고, 중간, 그리고 낮은 priority의 thread라고 하자. H가 L을 (L이 lock상태임)기다려야 하고, M이 ready list에 있을 때, L이 CPU에 할당이 안되니까 여기서 막힌다. 그래서 L이 lock을 유지하는 동안 L에 H의 우선순위를 donate해서 H로 한 다음 release lock을 한 뒤에 다시 L로 바꿔주는 방법을 쓴다. 

 그러니까 priority donation을 구현하렴. piority donation이 필요한 모든 상황을 고려해야 해. 여러 priority가 single thread에 여러 번 기부되는거 처리해야 하고, 중첩된 것도 처리해야 해. H가 M기다리고, M이 L기다리면, M과 L에 모두 H를 기부해야 하고..

 piority donation for lock도 구현해야 해. 다른 Pintos synchronization 구성에 priority donation은 필요 없을거야. 모든 경우에 대해 priority scheduling을 구현해야해. 

 마지막으로 다음 함수들이 thread의 각 priority를 검사하고 수정할 수 있도록 구현해. skeleton은 `threads/thread.c` 안에 있어

 <u>Function</u> : void **thread_set_priority** (int **new_priority**)

 current thread의 prioirty를 **new_priority**로 설정한다. current thread가 highest priority를 가지고 있지 않다면 yield해야지

 <u>Function</u> : int **thread_get_priority** (void)

 current thread의 priority를 return함. priority donate가 있으면 더 높은 priority를 return함.

 thread가 다른 thread의 priority를 직접 수정할 수 있도록 interface를 제공할 필요는 없어. 

priority는 다음 플젝부턴 사용 안할거야. [여기서 땡이라니….ㅠㅠ]



#### 2.2.4 Advanced Scheduler

 시스템에서 실행중인 작업의 평균 응답 시간을 줄이려면 4.4BSD scheduler와 같이 multilevel feedbakc queue scheduler를 구현하렴. Appendix B [4.4BSD Scheduler]를 봐.

 priority scheduler와 마찬가지로, advanced scheduler는 pirorities에 따라 실행할 thread를 선택해. 그러나, priority donation을 안함. 따라서 advanced scheduler에서 작업하기 전에 priority donation을 제외하고 사용하는게 좋아. 

 Pintos 시작 시간에 scheduling algorithm을 선택할 수 있도록 코드를 작성해야 해. 기본적으로 priority scheduler가 활성화 되어 있어야 하지만, `-mlfqs` 옵션을 사용해서 4.4BSD scheduler를 선택할 수 있어야 한다. `main()` 초기에 발생하는 `parse_options()` 에서 옵션이 해석되면 `threads/thread.h` 안에 `thread_mlfqs`가 true로 설정되는거야. 

 4.4BSD scheduler가 실행되면 thread는 더 이상 직접 자신의 priority를 제어하지 않아. `thread_create()` 로 설정된 priority argument는 무시되어야 하고, `thread_set_priority()` 는 scheduler가 설정한 thread의 current priority를 return해야 해.

 이후의 플젝에선 advanced scheduler 안쓰임. [ㅋ]



### 2.3 FAQ

- 코드 얼마나 작성해야해여?

![스크린샷 2017-03-20 17.50.16](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-03-20 17.50.16.png)



- 새 소스 파일 추가할 때 `makefile` 을 어케 업데이트함?

  - `.c` 파일을 추가하려면 최상위 `Makefile.build` 를 수정해. `dir_src` 라는 변수에 새 파일을 추가하면 됨. 이러면 `$make` 할 때 `threads/build/Makefile` 에 자동으로 복사됨. 반대로는 안되니까 `threads/` driectory에서 `$make clean` 을 실행할 때 변경 사항 날라감. 변경 사항이 임시적인게 아니라면 `Makefile.build` 를 편집하렴. [`Makefile.build` 를 수정하고 `$make` 하면 내용이 고대로 `threads/build/Makefile` 에 복사되는거임.]

    `.h` 파일은 `Makefile` 을 수정할 필요 없다. 

- `warning: no previous prototype for 'func'` 이 뭔소리냐

  - prototype으로 하지 않고 non-static funtion 으로 함수를 정의했단 말이야. non-static function은 다른 `.c` 파일에서 사용하기 위한 것이니까 안전을 위해서 헤더 파일에서 프로토 타이핑 해야해. 아니면 다른 `.c` 파일에서 실제로 사용하지 않는 경우 static으로 만들자.

- timer interrupt 의 간격은 몇임?

  - 초당 `TIMER_FREQ` 회 발생함. `devices/timer.h` 를 편집해서 이 값을 조정할 수 있음. 기본값은 100Hz. 그런데 이거 바꾸면 테스트들 많이 실패하니까 안바꾸는거 추천함

- time slice는 얼마나 김?

  - `TIME_SLICE` 는  time slice마다 tick하는데, 이 매크로는 `threads/thread.c` 에 선언되어 있어. 기본값은 4 tick임.

- 테스트 어케 실행함?

  - Section 1.2.1 [Testing]보셈 [ㅡㅡ]

- `pass()` 에서 실패하는 이유가 뭐야?

  - 다음과 같은 backtrace가 보일거야.![스크린샷 2017-03-20 18.21.09](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-03-20 18.21.09.png)

    이건 backtrace 라는 프로그램의 출력일거야. 이건 `debug_panic()` 에서 `pass()` 를 불렀다는 뜻이 아냐. 사실 `PANIC()`이라는 매크로 속 `debug_panic()`이  `fail()` 을 부른거야. `debug_panic()` 은 return하는 애가 아니야. 그래서 `debug_panic()` 이 `fail()`했다는 걸 받을 수 있는 코드가 없어. 그래서 stack의 return 주소가 memory의 `fail()` 다음에 발생하는 함수의 시작 부분에 있는 것처럼 보인다는걸 의미해. 이 경우에 `pass()`가 발생하지.

    자세한건 Section E.4 [Backtraces] 를 봐

- `schedule()`에 따르는 새로운 thread에서 interrupt를 재활성화 하는 방법이 뭐야?

  - `schedule()` 의 모든 path는 interrupt를 비활성화시켜. 그들은 결국 다음 thread에 의해 다시 사용되도록 설정돼. new thread는 `switch_thread()` 에서 실행되고, 몇 가지 가능한 함수들은 `schedule()`에 의해 실행되지.

    - `thread_exit()` 하지만 다시 thread로 switch되지 않음. 신경쓰지말자.
    - `thread_yield()` 는 `schedule()` 에서 return할 때 즉시 interrupt의 레벨을 복구한다. 
    - `thread_block()` 은 여러 위치에서 호출되는데:
      - `sema_down()` return하기 전에 interrupt의 레벨을 복구함
      - `idle()` 명시적 어셈블리 STI 명령으로 interrupt를 활성화함
      - `wait()` in `devices/intq.c` caller가 interrupt를 다시 활성화 해야함

    새로 생성된 thread가 처음으로 실행되는 특별한 경우가 있는데, 이러한 thread는 `intr_enable()`을 첫 번째 작업으로 호출해야해. 이 thread는 모든 kernel thread의 call stack의 맨 아래에 있음.



#### 2.3.1 Alarm Clock FAQ

- timer value의 overflow를 확인해야해?
  - timer value가 overflow될거란 걱정은 안해도 됌. imter value는 부호가 있는 64bit의 숫자니까 초당 100tick으로 거의 2,924,712,087년 동안 괜찮을거야. 



#### 2.3.2 Priority Scheduling FAQ

- priority scheduling은 결국 starvation으로 이어지지 않음?

  [starvation : 기아는 우선순위가 낮은 프로세스가 영원히 작동하지 않을 수 있다는 의미로 받으면 됨 ]
  - 글쳐. 엄격하게 하면 기아로 이어질 수 있어. 그래서 advanced scheduler는 동적으로 변경함. 하지만 priority가 높은게 더 많은 처리 시간을 가지도록 하는게 맞는거임. "fair"하진 않지만. 

- lock을 해제한 뒤엔 어떤 thread를 실행해야 함?

  - lock을 기다리는 최상위 priority의 thread는 lock을 해제하고 ready threads의 list에 넣어져야함. 그 다음에 scheduler가 ready threads list에서 가장 높은 priority를 가진 thread를 실행하는거야.

- highest-priority thread가 yield해도, 자기가 계속 실행됨?

  - 글쳐. 왜냐면 yield해도 본인이 highest-priority thread니깐. 여러 thread가 같은 highest priority가지고 있으면 `thread_yield()`는 "round robin" order로 switch되어야 함. 

    [round robin이 뭐시냐면 A, B, C의 3개의 task가 있으면 ABCABCA 순서로 전환하는거. 평등한 CPU시간이 할당되게 돌리면 된다]

- donating thread의 priority는 어케됨?

  - donate했다고 안변함. 그리고 5가 3한테 donate해도 8이 되는게 아니라 3이 되는겨.

- thread가 ready queue에 있는 동안 priority 바뀔 수 있음?

  - 당근이지. ready중이고 lock걸린 L을 생각해보면 H가 lock과 block을 원하면 L에게 H를 donate할거야.

- thread의 priority가 block된 채 바뀔 수 있나여?

  - ㅇㅋ. lock중인 L이 이 어떤 이유로 block되도, 높은 priority의 donation이 일어나면 높아질 수 있어. 이건 `priority-donate-sema` 테스트로 check할 수 있음.

- ready list에 추가된 thread가 processor를 선점할 수 있음?

  - 추가된 thread가 실행중인 thread보다 priority 높으면 바로 추가된 thread가 바로 yield해서 실행되어야함. 다음 timer-interrupt를 기다리는건 안됨. 바로바로 실행해야지. 물론 실행이 가능하다는 전제 하임.

- `thread_set_priority()`는 thread가 donation 받은거에 어떻게 영향을 줄까?

  - thread의 기본 priority를 설정하는 건데. 새롭게 설정된 priority나 highest priority의 donated priority로 설정됨. donate가 끝나면, priority는 fuction call로 설정된 priority로 된다. 이건 `priority-donate-lower` 테스트로 체크할 수 있어.

- 출력에 test 이름이 두 번 있으면 faile해요..ㅠ

  ![스크린샷 2017-03-20 19.07.14](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-03-20 19.07.14.png)

  - 이름이 두 번 출력되는 경우를 보고 있다고 해보자. 이건 두 thread의 출력이 겹치는거야. 이건 priority scheduler의 버그임. 결국 29의 thread가 30이 실행중에 실행되었단거야. 일반적으로  Pintos kernel에서 `printf()` 함수를 구현하면 `printf()` 호출하는 동안 콘솔이 잠금되고 이후에 해제하여 출력이 겹치는걸 방지하려고 시도하는데. output되는 test의 이름이 (alarm-priority)라고 하면, printf()를 두 번 실행하니까, 두 번째 호출에서도 콘솔이 잠금되고 출력된 다음에, 두 번 release된다. (resulting int the console lock being acquired and released twice)



#### 2.3.3 Advanced Scheduler FAQ

- priority dontation과 어떻게 상호작용 하나요?

  - 그럴 필요 없음. 동시에 테스트 안해요. 우선 순위 정책 하나만 씀.

- 64개의 queue대신 1개 queue쓰면 안됨?

  - 응 안돼. 같은 issue에도 구현이 달라질 수 있어. 

- 몇몇 scheduler test가 실패하는데 이해 못하겠음. 도움!

  ​	다음을 시도해보자.

  - 일단 소스파일 읽어보자. 그리고 각 목적과 결과를 설명하는 주석을 가지고 있을거야.
  - scehduler routine랑 여기에서 사용하는 fixed-point arithmetic routine를 다시 확인해.
  - timer interrupt에서 수행하는 작업량을 고려해보자. timer interrupt hanlder가 너무 오래 걸리면, timer interrupt에서 preempt한 thread가 대부분의 timer tick을 제거할거야. 이 thread로 제어권을 반환하면 다음 timer interrupt가 도착하기 전에 많은 작업을 수행하지 않아. 그래서 실제로 사용한 것보다 더 많은 CPU시간을 쓴거지. 이러면 우선순위를 낮춰서 scheduling 결정이 변경된다고. load average도 높혀줌.




## 3 Project 2: User Programs

 이제 user program을 실행할 수 있는 system 부분에 대한 작업을 시작하자. 기본 코드는 이미 user program을 실행할 수 있지만, I/O interactivity가 불가능함. 이 플젝에서는 system call을 써서 interact하도록 해야해.

 `userprog` directory에서 작업할건데, Pintos 의 거의 모든 부분과 interact할거야. [잘 짜란 얘기겠지]

 이건 플젝1코드 필요없음. alarm clock은 플젝3이랑 4에서 필요할 수 있지만 필수는 아니야.



### 3.1 Backgroud

 지금까지 짠 Pintos 코드는 다 OS kernel부분이였어. test code가 다 kernel에서 시스템이 특권을 줘서 실행된거야. 하지만 user program을 실행하면 더 이상 kernel에서 도는게 아니야. 이 플젝에서는 결과가 중요함.

 한 번에 하나 이상의 process를 실행할 수 있어. 각 프로세스에는 하나의 thread가 있어. (multi-threaded process안할겨. 지원안해). user program은 전체 machine을 가지고 있다고 생각하고 쓰여질거야. 그래서 여러 process를 실행할 때, 이런 illusion을 유지하려면 memory, scheduling이나 다른 상태를 잘 관리해야해.

 이전 플젝은 직접 kernel에서 컴파일했으니까 특정 함수가 필요했었어. 하지만 지금은 user program을 실행해서 OS를 검사할거야. 이게 훨씬 자유로워. user program interface가 여기 설명된 스펙만 충족하면 되긴 하는데, kernel code를 조건 아래에서 다시 쓰고싶으면 rewrite해.



#### 3.1.1 Source Files

 보자. ㅋ. `userprog` 엔 파일이 별로 없어.

`process.c`

`process.h`

ELF binaries를 load하고 실행해.

`pagedir.c`

`pagedir.h`

 80x86 하드웨어 page table의 간단한 관리자임. 수정하진 않는데, 일부 함수를 호출할 수 있어. 자세한건 Section 4.1.2.3 [Page Tables]보셈.

`syscall.c`

`syscall.h`

user process가 kernel에 access하려고 하면 system call호출한다. system call handler의 skelton code야. 지금은 메세지를 print하고 user process를 종료하는데, 이 플젝의 part2에서 system call에 필요한 코드를 많이 추가할거야. [너가 ㅋ]

`exception.c`

`exception.h`

 user program이 권한 밖, 금지된 작업 실행하면 "exception"이나 "fault"로 kernel에 trap됨. exception다룬단 얘기임. 지금은 message print하고 process종료하는게 다임. 전부는 아닌데 플젝2에서 `page_fault()` 를 수정해야 할 거야. 

`gdt.c`

`gdt.h`

 80x86은 segmeted architecture야. GDT(global descriptor table)은 segment를 설명하는 table임. 이 파일들은 GDT설정하는데, 이건 수정할 필요가 없어. GDT가 어떻게 작동하는지 궁금하면 읽어보던가.

`tss.c`

`tss.h`

 TSS(Task-State Segment) 는 80x86 architecture의 task switching에 사용됨. Pintos는 Linux처럼 user process가 interrupt handler를 시작할 때 stack을 switch 하기 위해서 사용됨. [퀴즈에 나올 것 같다.] 이거 수정할 필요 없어. TSS가 어케 되는지 궁금하면 읽어보던가.



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

 simulated 된 file system을 안<->밖 복사할 수 있어야 해. 




[번역 종료 너무 많다]












## A. Reference Guide

------

### A.1 Loading

[번역 필요성을 못 느끼겠음]

------

### A.2 Threads

#### A.2.1 `struct thread`

`threads/thread.h` 에 선언된 `struct thread` 가 Pintos의 threads data structure이다.

**struct thread**	<u>Structure</u>

 thread혹은 user process를 보여준다. 이 프로젝트에서 `struct thread` 에 own member를 추가해야해. 이미 존재하는 members를 바꾸거나 삭제할 수도 있어.

 모든 `struct thread` 는 고유의 momory page의 시작 부분을 차지해. page 나머지 부분은 thread의 stack으로 쓰는데 이거 페이지 끝에서부터 downward로 자라는거 알지?

![스크린샷 2017-03-16 00.14.12](/Users/prayanalog/Library/Mobile Documents/com~apple~CloudDocs/KAIST/2017 Spring/CS330 Operating Systems and Lab/Pintos Project/pintos 번역용 사진/스크린샷 2017-03-16 00.14.12.png)

 이런 방식은 두 가지 제한 요소를 만들어. 첫 번째로, struct thread가 너무 커지면 안돼. kernel stack을 위한 공간이 부족해지거든. 기본 struct thread는 고작 몇 바이트 정도야. 1kB 이하로 유지되어야 할거야. 두 번째로, kernel stack이 비대해질 수 없어. stack overflow가 발생하면 thread state가 오염될거야. 그러니까 kernel functions는 non-static local 변수들로 된 array나 large structure로 allocate하면 안돼. 대신 dynamic allocation인 `malloc()` 이나 `palloc_get_page()` 를 쓰자. (Section A.5[Memory Allocation]을 보자)



<u>Member of struct thread</u> : tid_t **tid**

 thread의 id이거나 tid임. 모든 thread는 고유 tid가 있어야 해. tid_t는 int의 typedef고 새 thread는 1부터 시작했던(initial process가 받았던) 순차적으로 증가하는 tid를 부여받아야 해. type을 바꿔도 되고 넘버링 방식을 바꿔도 됨.



<u>Member of struct thread</u> : enum thread_status **status**

 thread의 state이고 다음 중 하나임

- <u>Thread State</u> : **THREAD_RUNNING**

  thread가 실행중이다. 주어진 시간에 정확히 하나의 thread만 실행중이다. `thread_current()` 가 실행중인 thread를 return해줌

- <u>Thread State</u> : **THREAD_READY**

  thread가 실행할 준비가 되었지만, 실행중이 아니다. scheduler가 다음 thread를 부를 때 실행되도록 선택당할 수 있는 상태. 준비상태의 threads는 ready_list라는 doubly linked list에 보관된다.

- <u>Thread State</u> : **THREAD_BLOACKED**

  thread가 뭔가를 기다리고 있다. lock이나 interrupt되길. 상태가 `thread_unblock()` 을 통해 THREAD_READY가 될 때 까지 scheduler가 선택당하지 않음. Pintos synchronization primitieves중 하나를 쓰면 block이랑 unblock이 자동으로 될거야. (Section A.3 [Synchronizatino]을 보자)

- <u>Thread State</u> : **THREAD_DYING**

  scheduler가 switch하고 이 thread는 없어질거야



<u>Member of struct thread</u> : char **name[16]**

 string으로 thread의 이름이거나, 적어도 그 이름의 처음 부분이다.

<u>Member of struct thread</u> : uint8_t * **stack**

 모든 thread는 자신의 state를 track할 수 있는 stack을 가지고 있다. 이걸 편의상 A라고 할게. thread가 실행중일 때, CPU의 stack pointer register는 stack의 top을 track하며 그 동안 A는 사용되지 않아. 하지만 CPU가 다른 thread로 switch하면, 이 A에 thread의 stack pointer를 저장해. 다른 Member가 thread의 register를 저장할 필요가 없어, 왜냐면 stack에 저장될거니까.

 user program이나 kernel에서 interrupt가 발생하거나, `struct intr_frame` 이 stack에 push될거야. user program에서 interrupt되면, `struct intr_frame` 이 항상 page의 가장 위에 있을거야. Section A.4 [Interrupt Handling] 문서를 보렴.

<u>Member of struct thread</u> : int **priority**

 thread의 priority는 PRI_MIN(0)에서 PRI_MAX(63)의 범위 안에 있어. 63이 우선순위가 가장 높은거야. 0이 가장 낮고. 제공된 코드에서 우선순위를 무시하지만, 프로젝트 1에서 priority scheduling을 구현해야해. (Section 2.2.3 [Priority Scheduling] 을 봐)

<u>Member of struct thread</u> : struct list_elem **allelem**

 list element는 thread를 모든 thread목록과 연결하는데 쓰여. 각 thread가 생성될 때 list에 insert되고, 종료될 때 list에서 remove되어야 해. `thread_foreach()` 라는 함수는 모든 thread를 반복해서 실행할 때 사용될거야. 

<u>Member of struct thread</u> : struct list_elem **elem**

 ready상태의 thread list인 `ready_list` 나 `sema_down()` 에서 semaphore로 대기중인 thread list로 thread를 넣는데 사용되는 list이다. 위의 두 list는 doubly linked list임. semaphore위에서 대기중인 thread라면 ready상태가 아니고, 반대로 ready상태면 [ready는 run할 준비가 끝난 상태지만 ready하는 중임] semaphore위에서 대기중인 상태가 아니다. 그러니 둘 중 한 가지 일만 수행할 거야. `ready_list` 에 넣던가 semaphore위에서 대기중인 list에 넣던가.

<u>Member of struct thread</u> : unit32_t * **pagedir**

 프로젝트 2부터 할거야. Section 4.1.2.3 [Page Tables] 를 보렴.

<u>Member of struct thread</u> : unsigned **magic**

 항상 `threads/thread.c` 안에서 정의된 임의의 숫자인 `THREAD_MAGIC` 으로 설정되어 있는데 stack overflow를 감지하는데 쓰여. `thread_current()` 는 실행중인 thread의 `struct thread` 의 **magic**이  `THREAD_MAGIC` 로 설정되어 있는지 확인해줘. stack overflow가 일어나서 이 값을 바꾸는 경우가 있거든. 가장 좋은 점은, `struct thread` 에 member를 추가할 때, 마지막에 **magic** 을 남겨둔다는 점이야. 

[?????????allelem부터 magic 해석이 불완전하다 나중에 추가적으로 보완해야지]



#### A.2.2 Thread Functions

`theads/thread.c` 는 thread 지원을 위해서 몇가지 public 함수를 제공해뒀다. 쓸만한 것들을 보자.

<u>Function</u> : void **thread_init** (void)

 `main()` 에 의해 호출되며 thread 시스템을 초기화한다. 이 함수의 중요한 목표는 Pintos 의 초기 thread를 위해 `struct thread` 를 만드는 거야. Pintos loader가 초기 thread의 stack을 top of page에 넣고, Pintos thread가 같은 위치에 있어서 가능하게 됨. 

 `thread_init()` 이 실행되기 전에는 `thread_current()` 가 실패할거야. 당연히 돌아가는 thread의 **magic**이 틀리기 때문이지. 많은 함수들이 `thread_current()` 를 직/간접적으로 호출할텐데(lock을 걸기 위한 함수인 `lock_acquire()` 도 호출함) 그래서 `thread_init()` 은 Pintos 초기화 초반에 호출될거야. 

<u>Function</u> : void **thread_start** (void)

 `main()` 에 의해 호출되며 scheduler를 시작한다. idle thread(게으른 thread라 암것도 안할거임)를 만들고 다른 thread중 ready상태인게 없으면 저 idle thread를 schedule한다. 그 다음엔 interrupt를 활성화한다. interrupt는 scheduler에 부작용을 줄 수 있는데, 왜냐면`intr_yield_on_return()` 를 써서 scheduler가 timer interrupt에서 실행되다가 return될 수 있거든. (Section A.4.3 [External Interrupt Handling] 을 보렴.) [당장 이해 안되도 넘어가자. 프로젝트 2 문서를 읽고 코딩하다보면 이해될거임]

<u>Function</u> : void **thread_tick** (void)

 timer tick을 timer interrupt에서 호출한다. thread 통계를 track하고 time slice가 끝나면 scheduler대로 하라고 지시해준다.

<u>Function</u> : void **thread_print_stats** (void)

 Pintos가 종료되면서 thread 통계를 출력한다.

<u>Function</u> : tid_t **thread_create** (const char * **name**, int **prioirty**, thread_func * **func**, void * **aux**)

 **name**이라는 이름의 **priority**의 priority를 가지는 thread의 tid를 return한다. 이 thread는 **func**을 실행하며, **aux**라는 single argument(단일 변수)를 **func**에 전달한다. 

 `thread_create()` 는 thread의 `struct thread` 와 stack을 위해 각 member를 초기화하고, fake stack frame을 설정한다(Section A.2.3 [Thread Switching]을 봐). 이 thread는 차단된 상태에서 초기화되며, return되기 직전에 차단이 풀려. 뭔 말이냐면 초기화가 다 끝난 뒤에 schedule될 수 있단 말이야. 

​	<u>Type</u> : void **thread_func** (void * **aux**)

​	 이 함수의 타입이   `thread_create()`에서 **func**부분의 타입이야. **aux**부분이 `thread_create()` 의 **aux**고

<u>Function</u> : void **thread_block** (void)

 실행중인 thread의 state를 blocked state로 바꾼다. `thread_unblock()` 이 호출되기 전에는 실행되지 않으니까 뭐든 해야해. `thread_block()` 은 low-level 함수라, synchronization primitives중 하나를 쓰는걸 추천해. (Section A.3 [Synchronization]을 보자)

<u>Function</u> : void **thread_unblock** (struct thread * **thread**)

 blocked state인 thread를 state를 ready state로 바꿔서 실행될 수 있도록 해준다. thread가 뭔가 발생하기를(가령 lock된 thread가 available되기를) 기다리고 있을 때 호출되는 함수이다. 

<u>Function</u> : struct thread * **thread_current** (void)

 실행중인 thread를 return해줌

<u>Function</u> : tid_t **thread_tid** (void)

 실행중인 thread의 thread id를 return해줌. `thread_current() -> tid` 랑 같다.

<u>Function</u> : const char * **thread_name** (void)

 실행중인 thread의 이름을 return해준다. `thread_current() -> name` 이랑 같다.

<u>Function</u> : void **thread_exit** (void) **NO_RETURN** [리턴값 없도록 코드 짜는거 잊지 마라]

 현재 thread를 종료시킨다. 절대로 return하지 않는다. (Section E.3 [Function and Parameter Attributes]를 보렴)

<u>Function</u> : void **thread_yield**  (void)

 CPU가 scheduler실행하도록 한다. 즉 새로운 thread를 골라 실행시키게 한다. 새 thread는 current thread일 수도 있어. 그러니까 이 새 thread를 어떤 특정시간동안 실행되도록 **thread_yield**를 짤 수 없다는 말이야. [??????????]

<u>Function</u> : void **thread_foreach** (thread_action_func * **action**, void * **aux**)

 모든 threads에 대해서(각 thread를 **t**라고 하면) **action**(**t**, **aux**)를 실행하는 함수야. **action**은 다음과 같은 type의 함수여야 해.

​	<u>Type</u> : void **thread_action_func** (struct thread * **thread**, void * **aux**)

​	주어진 **aux**로 **thread**를 실행한다. 

<u>Function</u> : int **thread_get_priority** (void)

<u>Function</u> : void **thread_set_priority** (int **new_priority**)

 thread의 priority을 가져오고 **new_priority**로 바꾼다. (Section 2.2.3 [Priority Scheduling])

<u>Function</u> : int **thread_get_nice** (void)

<u>Function</u> : void **thread_set_nice** (int **new_nice**)

<u>Function</u> : int **thread_get_recent_cpu** (void)

<u>Function</u> : int **thread_get_load_avg** (void)

 advanced scheduler를 위한 Stub. [stub이 뭐야] Appendix B[4.4BSD Scheduler]를 봐.



#### A.2.3 Thread Switching

 `schedule()` 은 thread를 switch하는 담당이다. `threads/thread.c` 에 있으며 `thread_block()`, `thread_exit()`, `thread_yield()` 의 세 가지 public thread functions에 의해서만 호출된다. 위의 세 함수들이 `schedule()` 를 부르기 전에 interrut를 비활성화하고(이미 비활성화 되어 있으면 확인하고) 실행되는 thread의 state를 '실행'을 제외한 다른 상태로 변경해.

 `schedule()` 는 짧지만 까다로워. 현재 thread를 local 변수 **cur**에 기록하고, `next_thread_to_run()` 으로 부른 다음에 실행할 thread를 local 변수 **next**로, `switch_threads()` 로 진짜 thread switch를 실행하면 돼. switch한 thread는 `switch_threads()` 내부에서 실행중이고, 이전에 실행되었던 thread는 `switch_threads()` 로 return될거야. 

 `switch_threads()` 는 어셈블리 언어 방식이고 `threads/switch.S` 에 있어.

1. 얘가 레지스터를 stack에 저장하고
2. CPU의 current stack pointer를 current `struct thread` 의 **stack** member 안에 저장할거야. 
3. 그리고 새 thread의 **stack**을 CPU의 stack pointer로 복원하고
4. stack에서 레지스터를 복원한 다음에 return해.

 scheduler의 나머지는 `thread_schedule_tail()` 에 구현되어 있어. 여기에는 새 thread를 실행중으로 표시해줘. 방금 swtich한 thread가 dying state면, dying thread의 `struct thread`가 가지고 있던 page와 stack을 free하게 해줘야 해. thread가 switch되기 전엔 free할 수 없는데, switch하기 위해선 이것들을 써야 하기 때문이지. 

 thread를 처음 실행하는 건 특별한 경우야. `thread_create()` 가 새 thread를 만들고, 제대로 시작하려면 많은 문제가 있어. 특히, 새 thread는 아직 시작되지 않은 상태인데, scheduler가 원하는대로 `switch_threads()` 안에서 돌아가게 할 방법이 없어. 이 문제를 해결하기 위해서는 `thread_create()` 가 가짜 stack frame들을 새 thread의 stack안에 만들어줘야 해. 

- `struct switch_threads_frame` 에 있는 `switch_threads()`를 위해 topmost fake stack이 있어. 이 frame의 중요한 부분은 **eip** member, 즉 return address야. **eip**가 `switch_entry()` 를 가리키고 있거든. `switch_entry()` 가 함수를 호출다는 뜻이야.
- 다음 fake stack frame은 `threads/switch.S` 안의 어셈블리 언어 방식으로 되어 있고 stack pointer를 조정하는 `switch_entry()` 를 위해서 존재해. `thread_schedule_tail()` 를 호출하고(이 특별한 경우 때문에 `thread_schedule_tail()` 이 `schedule()` 과 분리된거야) return한다. stack frame을 채우면 `threads/thread.c` 안에 있는 `kernel_thread()` 를 return한다. 
- final stack frame은 `kernel_thread()` 를 위해 있는데, interrupt할 수 있으며 thread의 함수를 호출할 수 있어 (`thread_create()` 안에 있는 함수). thread의 함수가 return하면 `thread_exit()` 를 불러서 thread를 종료시킴.

[위의 세 부분은 잘 번역 못한 것 같음]



### A.3 Synchronization

 thread간의 자원 공유가 신중하고 통제된 방식으로 잘 다뤄지지 않으면, 큰 문제가 생긴다. 특히 OS kernel에서는 전체 시스템을 손상시킬 수 있어. Pintos는 몇 가지 synchronization 기본 요소를 제공해.



#### A.3.1 Disabling Interrupts

 synchronization을 쉽고 편하게 하려면 interrupt를 disable하게 하면 된다, 즉 CPU가 interrupt에 일시적으로 반응하지 못하게 막으면 된다. interrupt가 꺼지면, thread가 실행되고 있을 때 다른 thread가 선점(preempt)할 수 없다. 왜냐면 thread의 선점방식은 timer interrupt에 의해 일어나기 때문이지.  interrupt가 정상적으로 실행되면, 실행되는 thread는 언제든 다른 thread에 의해 preempt될 수 있고, C코드가 있을때 2개의 코드 사이에서 일어날 수도 있고, 심지어 하나가 실행이 되는 중에도 일어날 수 있어.

 덧붙여서 , Pintos가 'preemptible kernel'이라는 걸 의미한다. 그니까 kernel thread도 언제나 preempt당할 수 있다는 말이야. 원래 Unix 시스템은 nonpreemptible했는데, kernel thread가 scheduler에 명시되어 있는 상황에서만 preempt될 수 있었단 뜻이지. (두 모델 모두 User 프로그램은 항상 preempt될 수 있었음.) preemptible kernel은 더 명쾌한 synchronization이 필요해. 왜인지는 생각해보면 답이 나올거야. 

 interrupt의 state를 직접 설정할 필요는 거의 없어. 대부분의 시간을 뒤에 section에서 설명하는 다른 synchronization primitives를 사용할거야. interrupt를 비활성화 하는 중요한 이유는 kernel thread외부의 interrupt handler와 synchronize하기 위해서야 외부의 interrupt hanlder 재울 수 없어서 다른 형태의 synchronization으로 사용할 수 없어. (Section A.4.3 [External Interrupt Handling]을 보자)

 몇몇 외부 interrupt는 interrupt를 비활성화 해도 미뤄지지 않아. 이런 애들을 non-maskable interrupts(NMIs)라고 하는데 응급 상황에서만 사용하렴(컴퓨터에 불이 나거나 하는 경우를 말하는거야). 그런데 Pintos는 non-maskable interrupt를 안다뤄.

 interrupt를 비활성화하거나 활성화 하는 함수랑 타입은 `threads/interrupt.h` 에 있어

<u>Type</u> : enum **intr_level**

 interrupt가 비활성화 되어 있으면 `INTR_OFF`, 반대로 활성화 되어 있으면  `INTR_ON` 야.

<u>Function</u> : enum intr_level **intr_get_level** (void)

 현재 interrupt 상태를 return해줘.

<u>Function</u> : enum intr_level **intr_set_level** (enum intr_level **level**)

 **level**에 따라 interrupt를 켜거나 끄고 이전의 interrupt state를 return해줘.

<u>Function</u> : enum intr_level **intr_enable** (void)

 interrupt를 키고, 이전 interrupt state를 return해줘.

<u>Function</u> : enum intr_level **intr_disable** (void)

 interrupt를 끄고, 이전 interrupt state를 return해줘.



#### A.3.2 Semaphores

 samaphore는 0혹은 양의 정수이며, 두 개의 operations가 원자적으로 작동시킨다. [바로 밑의 예시를 보면 원자적으로 작동한다는게 뭔 소린지 이해가 될거임]

- "Down" 혹은 "P" : 값이 양수가 될 때 까지 기다렸다가 감소시킨다.
- "Up" 혹은 "V" : 값을 증가시킨다.(그리고 대기하는 thread가 있으면 하나를 깨운다)

  0으로 초기화 된 semaphore는 정확히 한 번 발생하는 이벤트를 기다리는데 사용할 수 있다. 예를 들어, thread A가 시작되고 thread B가 뭔가를 끝내고 신호를 보낼 때 까지 기다리길 원한다고 가정하자. A는 semaphore를 0으로 초기화해서 B에게 전달하면 semaphore를 "Down"한다. [그럼 여기서 "Down"은 양수가 될 때 까지 기다렸다가 감소시키는 거고, 0인 상태니까 기다린다] B가 뭔갈 끝내면 semaphore를 "Up"한다. [그럼 0이었던 semaphore가 1이 된다. 이러면 waiting thread가 깨어나고 동시에 걸려있던 "Down"이 실행되면서 semaphore가 0이 된다.] 이 작업은 semaphore를 A가 "Down"시켰거나나 B가 "Up"시켰더라도 상관없이 작동해. 

   1로 초기화 된 semaphore는 일반적으로 resource에 대한 access를 제어하는데 사용된다. code가 resource를 쓰기 전에, semaphore를 "Down"시킨다. [이러면 semaphore가 0이다] 그리고 사용하고 나서 "Up"시킨다. lock을 쓰는게 더 좋을 수 있긴 해.(아래 설명되어 있음)

   semaphore가 1보다 큰 값으로 될 수도 있는데 거의 사용 안해.

 Pintos의 semaphore type과 operations는 `threads/synch.h`  에 있어

<u>Type</u> : stuct **semaphore**

 semaphore를 나타낸다.

<u>Function</u> : void **sema_init** (struct semaphore * **sema**, unsigned **value**)

 **sema**를 **value**값으로 초기화시킨다.

<u>Function</u> : void **sema_down** (struct semaphore * **sema**)

 "Down" 이나 "P" operation을 **sema**에 operate한다. 위의 설명대로 양수일때 까지 기다리고 1 감소시킨다. 

<u>Function</u> : bool **sema_try_down** (struct semaphore * **sema**)

 "Down" 이나 "P" operation을 **sema**에 operate하길 대기하지 않고 시도해본다. **sema**가 성공적으로 감소하면 true를 return한다. 0이거나 대기하지 않고 감소시킬 수 없으면 false를 return한다. 작은 loop에서 이 함수를 쓰면 CPU 시간이 낭비되니까, **sema_down()**을 쓰거나 다른 방법을 찾아봐

<u>Function</u> : void **sema_up** (struct semaphore * **sema**)

 "Up" 이나 "V" operation을 **sema**에 operate하고, 값을 1 증가시킨다. **sema**를 기다리는 thread가 있으면 그들 중 하나를 깨운다. 

 대부분의 synchronization과는 다르게, **sema_up()**은 외부 interrupt handler를 부를거야. (Section A.4.3 [External Interrupt Handling]을 보자)



 semaphore는 interrupt 를 비활성화해서 thread를 block하거나 unblock한다. (Section A.3.1 [Disabling Interrupts]를 보자) (`thread_block()`  이랑 `thread_unblock()` ) 각각의 semaphore는 `lib/kernel/list.c` 에 있는 linked list를 통해 대기중인 thread list를 유지 및 관리한다. 



#### A.3.3 Locks

 lock은 semaphore가 1인 상태와 같다.(Sectino A.3.2 [Semaphores] 보셈). "Up"하면 release하는 것과 같고, "down" operation은 "acquire"과 같다. semaphore 과 비교하여 lock은 제한 사항이 더 있어. lock이 acquire된 thread는 lock의 소유자만 할 수 있어. [semaphore는 A가 down하든 B가 down하든 상관 없는거랑 다르지?] 이 제한사항이 문제가 된다면, lock 대신 semaphore를 사용해야 한다는 좋은 신호야.

 Pintos에서 lock는 recursive하지 않아. 그러니까 lock을 hold하고 있는 thread가 lock을 acquire하려고 하는 건 에러란 말이지.

 lock 타입들이랑 함수들은 `threads/synch.h` 에 있어

<u>Type</u> : struct **lock**

 **lock**을 보여준다

<u>Function</u> : void **lock_init** (struck lock * **lock**)

 **lock**을 새 lock으로 초기화한다. 이 lock은 어떤 thread의 소유로 초기화되는게 아니야.

<u>Function</u> : void **lock_acquire** (struct lock * **lock**)

 현재 thread의 **lock**을 acquire한다. 필요하다면 첫번째 소유자를 위해 기다려야 한다.

<u>Function</u> : bool **lock_try_acquire** (struck lock * **lock**)

 현재 thread의 **lock**을 기다리지 않고 acquire해본다. 성공하면 true return, 이미 lock이 소유되어 있으면 false return. 작은 loop에서 이 함수를 쓰면 CPU 시간이 낭비되니까, **lock_acquire()**을 쓰자. 

<u>Function</u> : void **lock_release** (struck lock * **lock**)

 현재 thread가 소유자면, **lock**을 release한다.

<u>Function</u> : bool **lock_held_by_current_thread** (const struck lock * **lock**)

 실행중인 thread가 **lock**의 소유자면 true return. 어떤 thread가 lock을 소유하고 있는지 확인하는 함수가 없다. 왜냐면 caller가 응답하기 전에 바뀔 수 있거든. 



#### A.3.4. Monitors

 monitor는 semaphore보다 높은 수준의 synchronization 방식이야. monitor는 synchronized된 data와 monitor lock, 그리고 하나 혹은 그 이상의 condition variables(조건 변수)를 포함하고 있어. protected data에 access하기 전에, thread는 monitor lock 을 acquire해. 한 뒤의 상황을 "in the monitor"라고 해. in the monitor에서는, thread가 모든 protected data를 자유롭게 검사하거나 수정할 수 있어. protected data에 access가 끝나면, monitor lock을 release해.

 condition variable은 in the monitor인 code가 condition이 true가 될 때 까지 기다리는걸 허용한다. 각 condition varialbe은 "몇 data가 처리를 위해 도착했어"나 "user가 키를 누른지 10초가 지났어"같은 추상적인 상황과 관련이 있어. in the monitor인 code가 condition이 true가 되길 기다려야 할 때, 그 code는 lock을 release하고 condition이 신호를 받을 때 까지 기다리는 condition variable과 관련된 condition variable을 "waits"하는거야. 반면에, conditions 중 하나가 true가 되면, condition에 하나의 waiter를 깨우기 위해 "signals" 하거나, 모든 condition에 그들을 모두 깨우라고 "broadcases"한다. 

[개념적인 내용이라 코드를 짜면서 이해하자..읽으면서 이해하는거 짱 어렵네]

 Conditino variable의 타입들과 함수들은  `threads/synch.h` 에 있다.

<u>Type</u> : struct **condition**

 condition variable 을 보여준다

<u>Function</u> : void **cond_init** (struct condition * **cond**)

 **cond**를 새 condition variable로 초기화한다.

<u>Function</u> :  void **cond_wait** (struct condition * **cond**, struct lock * **lock**)

 원자적으로 **lock**(monitor lock)을 release하고 **cond**가 다른 코드로부터 신호를 받을 때 까지 기다린다. **cond**가 신호를 받으면, return하기 전에 **lock**을 reacquire하자. **lock**은 반드시 이 함수가 불려지기 전에 hold되어 있어야 한다. 

 신호를 보내고 대기상태로부터 일어나는건 atomic operation이 아니다. 따라서 일반적으로 `cond_wait()` 를 호출한 함수는 대기가 끝난 뒤 condition을 다시 체크해야 하며, 필요하다면 다시 기다려야 한다. A.3.4.1 [Monitor Example]에서 예시를 보여줄거야

<u>Function</u> : void **cond_signal** (struct condition * **cond**, struct lock * **lock**)

 어떤 thread가 **cond**(protected by monitor lock **lock**)때문에 대기중이라면, 이 함수가 그둘 중 하나를 깨워줄거야. 대기중인 thread가 없다면, 아무것도 하지 말고 return하자. **lock**은 반드시 이 함수가 call되기 전에 hold되어 있어야 한다. 

<u>Function</u> : void **cond_broadcast** (struct condition *  **cond**, struct lock * **lock**)

 모든 thread들 중 **cond**(protected by monitor lock **lock**)때문에 대기중인 threads 깨운다. **lock**은 반드시 이 함수가 call되기 전에 hold되어 있어야 한다. 



##### A.3.4.1. Monitor Example

 monitor의 고전적 예시는 하나 이상의 "producer" thread가 characters를 wrtie하고 "consumer" thread가 characters를 읽는 buffer를 처리하는 것이다. 이를 구현하기 위해서는 monitor lock외의 `not_full` 과 `not_empty` 라는 2개의 conditino variables가 필요하다. 

```c
char buf[BUF_SIZE];		/* Buffer */
size_t n = 0;			/* 0 <=n<=BUF_SIZE # of characters in buffer */
size_t head = 0;
size_t tail = 0;		/* buf index of next char to read (mod BUF SIZE). */
struct lock lock;		/* Monitor lock. */
struct condition not_empty; /* Signaled when the buffer is not empty. */ struct condition not_full; /* Signaled when the buffer is not full. */
```

…lock과 condition variables의 초기화...

```c
void put (char ch) {
  lock_acquire (&lock);
  while (n == BUF_SIZE)				/* Can’t add to buf as long as it’s full. */
    cond_wait (&not_full, &lock);
  buf[head++ % BUF_SIZE] = ch;		/* Add ch to buf. */
  n++;
  cond_signal (&not_empty, &lock);	/* buf can’t be empty anymore. */
  lock_release (&lock);
}
char get (void) {
  char ch;
  lock_acquire (&lock);
  while (n == 0)					/* Can’t read buf as long as it’s empty. */
    cond_wait (&not_empty, &lock);
  ch = buf[tail++ % BUF_SIZE];		/* Get ch from buf. */
  n--;
  cond_signal (&not_full, &lock);		/* buf can’t be full anymore. */  
  lock_release (&lock);
}
```

 `BUF_SIZE` 는 `SIZE_MAX + 1` 로 완전히 정확하게 나눠진다. 그렇지 않으면, **head** 는 처음에 0으로 되며 실패할거야. `BUF_SIZE` 는 2의 거듭제곱 꼴이어야 한다. 



#### A.3.5 Optimization Barriers

[barrier 최적화]

 optimization barrier는 barrier의 memory 상태를 컴피일러가 막는 특정한 상태이다. 컴파일러가 barrier를 가로질러서 변수의 쓰기나 읽기를 하지 않을거야. barrier를 가로질러 변수를 수정하는것도 용납안해. 대신 local 변수들은 제외하고. [??????????앞에꺼 번역좀] Pintos에서 `threads/synch.h` 는 `barrier()` 매크로를 optimiaztion barrier로 정의해.

optimization barrier를 쓰는 이유 중 하나는 컴파일러가 모르는 채로 다른 thread나 interrupt handler가 data가 비동기적으로 변화 시킬 때가 있어서다.예를 들어`devices/timer.c` 안의 `too_many_loops()` 함수가 있어. 이 함수는 timer tick이 발생할 때 까지 busy-waiting loop로 시작한다구.

```c
/* Wait for a timer tick. */
int64_t start = ticks;
while (ticks == start)
	barrier();
```

 optimization barrier 없이 이 loop에서는, 컴파일러가 절대 끝나지 않는 loop를 가지고 있어. 왜냐하면 `start`랑 `ticks` 은 시작부터 같았고 while loop가 절대 바꿀 수 없잖아. 그니까 바람직하지 않은 무한 loop를 optimize할 수 있어야 해. 

 optimization barrier은 다른 컴파일러의 optimization을 피하도록 쓸 수 있어.  `devices/timer.c` 에 있는 `busy_wait()` 함수에 이런 loop가 있다.

```c
while (loops--> 0)
	barrier();
```

 이 loop의 목표는 원래의 value에서 0까지 내리는 busy-wait이다. barrier없이는 아무런 효과가 없으므로 컴파일러는 이 부분을 컴파일하지 않고 삭제시켜버린다. 하지만 barrier가 저 loop가 효과가 있도록 해주므로 컴파일을 강제로 하도록 만든다. 

 마지막으로, optimization barrier는 memory의 읽고 쓰기 순서를 바꿀 수 있다. 예를 들어, 

```c
timer_put_char = 'x';
barrier();
timer_do_put = true;
```

 barrier가 없으면, 이 코드는 버그에 걸릴거야. 왜냐하면 컴파일러가 순서를 유지할 필요를 못 느끼고 편한대로 컴파일할 수 있거든. 이 경우에서 커파일러는 순서가 중요하단걸 몰라서, optimizer가 순서 바꾸는걸 허용할거야. 확실하지는 않지만, 컴파일러 버젼이나 다른 최적화 방법에 다른 방식으로 행동할 수 있어.

 interrupt를 그 주변에서 사용하지 않게 하는 것도 한 방법이야. 이건 순서 바꾸는걸 막지는 않지만, 두 assignments사이에 오는 interrupt handler는 방지해줘. interrupt의 활성화/비활성화에 시간이 들어가긴 해.

```c
enum intr_level old_level = intr_disable();
timer_put_char = 'x';
timer_do_put = true;
intr_set_level (old_level);
```

 두 번째 방법은 `timter_put_char` 와  `timer_do_put` 를 'volatile'(휘발성있게) 선언하는거야. 이 keyword는 외부 관측 및 최적화에 대한 위도를 제한하도록 컴파일러에게 알려줘. 그런데 'volatile'은 잘 정의되지 않아서 좋은 방법이 아니야. 그래서 기본 Pintos에선 전혀 안써.

 이 다음건 솔루션이 아니야, lock이 interrupt를 방지하거나 코드가 컴파일러가 순서 바꾸지 않도록 막지 못하거든. 

```c
lock_acquire(&timer_lock);		/* INCORRECT CODE */
timre_put_char = 'x';
timer_do_put = true;
lock_release(&timer_lock);
```

  컴파일러는 외부에서 정의된 함수(다른 소스코드)를 제한된 optimization barrier로 생각해. 특히, 컴파일러는 외부에서 정의된 함수를 정적 혹은 동적으로 할당된 data와  어떤 것의 address인 local변수에 access할 수 있다고 하자. 이것은 종종 explicit barrier가 무시될 수 있다는 말이다. 이게 Pintos가 몇 explit barrier를 포함하고 있는 이유 중 하나야.

 소스파일 header에 포함되어 있거나 같은 소스 파일에서 정의된 함수는 optimization barrier에 의존할 수 없어. 컴파일러가 전체 소스파일을 최적화 전에 읽고 parsing할 수 있어서 정의하기 전 함수의 호출에도 적용돼.

[barrier부분은 번역 다시 해야해]



### A.4 Interrupt Handling

  interrupt는 CPU에게 어떤 이벤트를 알려준다.  OS의 대부분 작업은 어떤 식으로든  interrupt와 관련되어 있어. 우리는 그 목적에 따라 두 종류의 interrupt로 나눴어 :

- **Internal interrupt** CPU의 지시로 직접 interrupt를 발생시키는거야. System call은 invalid memory access(page fults)를 시도하거나 뭔가를 0으로 나누려고 시도할 때 internal interrupt로 인해 발생해. CPU의 명령에 의해 발생하기 때문에, internal interrupt은 CPU의 명령과 동기화되어 있어. `inter_disable()` 는 internal interrupt를 비활성화 할 수 없어.
- **External interrupt** CPU외부에서 interrupt가 발생한거야. 이 interrupt는 hardware devices 즉 system timer, keyboard, serial ports, 그리고 disk같은 곳에서 와. External interrupts는 비동기적이야, 무슨 말이냐면 명령 실행과 동기화되지 않는다고. external interrupt는 `intr_disable()` 이나 관련된 함수로 연기시켜둘 수 있어. (Section A.3.1 [Disabling Interrupts]를 보자)

 CPU는 둘 다 같은 방식으로 다루고,  Pintos도 역시 둘을 다루는 같은 infrastructure가 있어. 뒤의 section이 설명해줄거야. 

 chapter 3 "Basic Execution Environment"를 안 읽었다면, 읽어. chapter 5 "Interrupt and Exception Handling" 도! 

[아니 시발 이거 읽다보면 저거 읽으라하고 저거 읽으면 또 다른거 읽으라하고 걍 플젝 하려면 문서 전체 다 읽으라고 지껄이네 ㅡㅡ]



#### A.4.1 Interrupt Infrastructure

 interrupt가 발생하면, CPU는 가장 중요한 state를 stack에 저장하고 interrupt handler routine으로 건너뛴다. 80x86 architecture는 256가지 interrupt가 있는데, 0부터 255까지 있어. 각 독립적인 handler는 interrupt descriptor table이라고 하는 array나 IDT에 정의되어 있어. 

 Pintos에서 `threads/interrupt.c` 에 있는 `intr_init()` 은 IDT로 설정되어 있는데, 이게 뭔 말이냐면 각 entry point가 `threads/intr-stubs.S` 안에 `intrNN_stub()` 라고 하는 unique한 entry point로 설정되도록 했단 말이야. (NN은 16진수의 interrupt number임) 왜냐면 CPU는 우리가 interrupt number를 찾을 수 있도록 해주는게 하나도 없어서, 이 entry point는 interrupt number를 stack에 push해줘. 그러면 `intr_entry()`로 점프하고(아직 모든 프로세서가 바뀐건 아니야), `intr_handler()` 를 불러서 `threads/interrupt.c` 안의 C코드를 가져와줌. 

 `intr_handler()` 의 주요 임무는 특정 interrupt를 처리하기 위해서 등록된 함수를 호출하는거야. (등록된 함수가 없으면, console에 정보를 넘겨주고 panic해버림.)[panic이라는 함수가 있어. 그냥 panic이라는 문구를 띄워주고 멈춰있음ㅋ 자세한 루틴은 아직 모르겠음. 공부 안함] external interrupt를 위해 몇몇 추가적인 기능도 있어. 

 `intr_handler()` 가 return되면, `threads/intr-stubs.S` 에 있는 어셈블리 코드가 interrupt되기 전 CPU register로 다시 store하고 복귀하도록 해줘.

 모든 interrupt에서 공통적으로 쓰이는 타입들과 함수들이야.:

<u>Type</u> : void **intr_handler_func** (struct intr_frame * **frame**) 

 얘는 interrupt handler 함수를 선언하는 애야. 변수 **frame**은 interrupt의 원인과 상태를 결정해줘.

<u>Type</u> : stuct **intr_frame**

 `intr_entry()` 나 CPU에 저장되고 interrupt의 부분인 interrupt handler의 stack frame이야. 밑에 볼만한 애들을 설명과 함께 써놨어

<u>Member of struct **intr_frame**</u> : uint32_t **edi**

<u>Member of struct **intr_frame**</u> : uint32_t **esi**

<u>Member of struct **intr_frame**</u> : uint32_t **ebp**

<u>Member of struct **intr_frame**</u> : uint32_t **esp_dummy**

<u>Member of struct **intr_frame**</u> : uint32_t **ebx**

<u>Member of struct **intr_frame**</u> : uint32_t **edx**

<u>Member of struct **intr_frame**</u> : uint32_t **ecx**

<u>Member of struct **intr_frame**</u> : uint32_t **eax**

<u>Member of struct **intr_frame**</u> : uint16_t **es**

<u>Member of struct **intr_frame**</u> : uint16_t **ds**

 `intr_entry()` 가 push해 준 interrupted thread의 register 값이다. `esp_dummy` 값은 실제로 쓰이지는 않지롱...

<u>Member of struct **intr_frame**</u> : uint32_t **vec_no**

 interrupt vector 숫자임(0~255)

<u>Member of struct **intr_frame**</u> : uint32_t **error_code**

 error code가 CPU의 internal interrupt를 통해 stack에 push되는데 그 error code임.

<u>Member of struct **intr_frame**</u> : void **(* eip)** (void)

 interrupted thread가 실행할 다음 instruction(명령)의 address.

<u>Member of struct **intr_frame**</u> : void *** esp** 

 중단된(interrupted) thread의 stack pointer

<u>Function</u> : const char * **intr_name** (uint8_t **vec**)

 interrupt numbered **vec**의 이름을 return한다. interrupt가 등록된 이름이 없으면 "unknown"이라고 return해야함.



#### A.4.2 Internal Interrupt Handling

 Internal interrupt는 실행중인 kernel thread나 user process의 실행으로  발생되어서 CPU 명령에 의해 직접 발생한다. 그래서 internal interrupt는 "process context"라 불림.

 internal interrupt handler는 interrupt handler에 상속된 `struct intr_frame` 를 검사하거나 수정할 수 있어. interrupt 가 return하면, `strcut intr_frame` 의 수정사항흔 호출한 thread나 process의 상태를 바꿔. 예를 들어, Pintos system call handler는 user program가 저장된 EAX register를 수정하도록 값을 return해줘. (Section 3.5.2 [System Call Details]를 보렴)

 특별히 internal interrupt handler가 할 수 있는 일과 할 수 없는 일에 기준은 없어. 일반적으로 interrupt가 enabled된 상태에서는, 다른 코드처럼 실행되는데, 그래서 다른 kernel thread로부터 preempt될 수 있지. 따라서 다른 thread들과 공유하는 data나 다른 resources를 synchronize할 필요가 있어. (Section A.3 [Synchronization]을 봤…겠지?)

 Internal interrupt handler는 recursive하게 호출될 수 있어. 예를 들어, user memory를 읽으려고 할 때 발생하는 page fault는 system call handler가 부른거야. recursion을 깊게 하면 kernel stack에 한계가 있으니 overflow가 발생할 수 있어서 필요없어야해. (Section A.2.1 [struct thread]도 봤지?)

<u>Function</u> : void **intr_register_int** (uint8_t **vec**, int **dpl**, enum intr_level **level**, intr_handler_func * **handler**, const char * **name**)

 Register handler는 internal interrup의 숫자인 **vec**이 trigger될 때 불려진다. interrupt의 **name**은 디버깅하려고 부르는거임.

 **level**이 `INTR_ON` 이면 interrupt handler가 실행중일 때, external interrupt는 정상적으로 실행될거야. `INTR_OFF` 면  interrupt handler가 호출될 때 CPU가 external interrupt를 비활성화 할거고. handler 내부에서 `intr_disable()` 을 호출할 때와는 살짝 다른데,  external interrupt가 활성화되어 있는 CPU 명령을 야기하기 때문이야. 이건 page fault handler에서 중요한데, `userprog/exception.c` 에서 자세하게 적혀 있어.

 **dpl**은 interrupt가 어떻게 호출될 지 결정해. **dpl**이 0이면, interrupt는 kernel thread에 의해서만 호출될 수 있어. 아니면 **dpl**은 3이어야 하고, 이건 user process가 INT 명령으로 interrupt를 호출할 수 있단 얘기야. invalid memory reference는 **dpl**과 상관없이 page fault를 발생시키는 것처럼 **dpl**의 값은 user process의 interrupt를 간접적으로 부르는 성능에 영향을 주진 않아.



#### A.4.3 External Interrupt Handling

 CPU 외부에서 발생하는 이벤트가 external interrupt의 원인이야. 걔들은 비동기적이라 interrupt를 비활성화하지 않는 이상 언제나 발생할 수 있어. external interrupt는 "interrupt context"에서 실행된다고 하자. [머라는겨...]

 handler에 상속된 `struct intr_frame` 인 external interrupt는 전혀 의미가 없어. 그건 그냥 thread나 process가 interrupt되었다는 상태를 알려줄 뿐, 예측할 방법이 전혀 없어. 필요 없지만, 만약 예측가능하고 검사하고 수정할 수 있다면 재앙의 불씨가 될거야. […? 왜죠?]

 하나의 external interrupt가 동시에 발생할 수 있어. internal 혹은 external interrupt는 external interrupt handler에 있지 않을 수 있어. 따라서 external interrupt의 handler는 interrupt 가 비활성화된 채로 실행되어야 해. (Section A.3.1 [Disabling Interrupts]를 봐) [?????????? what?]

 external interrupt handler는 `lock_acquire()` , `thread_yield()` 등 비슷한 다른 함수들이 부를 때를 제외하고는 자고 있거나 항복하면 안된다. inetrrupt context에서 잔다는 건, interrupt handler가 scheduled와 returned에 반항할 때 까지 interrupted thread를 효과적으로 재우는 거야.  release a lock하는 것 처럼 handler가 잠자는 thread를 기다리고 있다면 deadlock에 걸릴거야. 

 external interrupt hnadler는 효과적으로 다른 동작을 지연시키고 machine을 독점한다. 따라서 external interrupt handlers는 가능한 빨리 끝내야 해. kernel thread가 도는 것 보다 CPU 시간을 더 많이 쓰거나, 가능하면 interrupt trigger가 synchronization primitive를 사용하는 것 보다. [해석 어렵다..]

 external interrupt는 **PICs(programmable interrupt controllers)**라고 불리는 CPU밖의 devices 한 쌍이 컨트롤한다. `intr_init()` 이 CPU의 IDT를 설정하면, PICs도 초기화한다. PIC는 각 external interrupt의 마지막 처리 부분을 "acknowledged"해야 한다. [어디가 끝인지 알아야 한다는 소리겠지?] `intr_hanlder()` 는 PICs의 신호인 `pic_end_of_interrupt()` 를 불러서 처리한다.

external interrupt에 관한 함수들:

<u>Function</u> : void **intr_register_ext** (uint8_t **vec**, intr_handler_func * **handler**, const char * **name**)

 register handler는 external interrupt 번호 **vec**으로 trigger될 때 호출된다. interrupt의 이름 **name**은 디버깅할 때 필요한 거고.. interrupt가 비활성화일 때도 돌아갈거야.

<u>Function</u> : bool **intr_context** (void)

 interrupt context에서 돌아가고 있으면 true를 return하고 아니면 false를 return한다. 함수가 자고 있거나 아니면 interrupt context로 불려지지 않을 때 주로 사용하는데, 이렇게 사용함

```c
ASSER (!intr_context());
```

<u>Function</u> : void **intr_yield_on_return** (void)

 interrupt context일 때 호출하는데, `thread_yield()` 가 interrupt가 return하지 바로 전에 불려질 때이다. thread의 시간이 만료되거나, 새 thread가 scheduled될 대, timer interrupt handler가 사용한다. 
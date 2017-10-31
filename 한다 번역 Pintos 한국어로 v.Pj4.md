## 5 Project 4:File Systems

 이번엔 File system의 implementation을 증진시킬거야. 거의 `filesys` directory에서 할거임.

 플젝3위에서 빌드, VM기능을 사용하려면 `filesys/Make.vars` 를 편집해야해. [아마 너가 vm에서 만든 파일을 추가해야겠지?]



### 5.1 Background

#### 5.1.1 New Code

 너에게 아마 새로운 파일들일거야. 표시 안해둔것들은 `filesys` directory에 대부분 있어.

`fsutil.c`

 kernel command line에서 access할 수 있는 file system을 위한 simple utilities.

`filesys.h`

`filesys.c`

 file system에 대한 Top-level interface. Section 3.1.2[Using the File System] 보셈.

`directory.h`

`directory.c`

 file name을 inodes로 translate해줌. directory data structure는 file로 저장됨.

`inode.h`

`inode.c`

 disk의 file layout을 보여주는 data structure을 관리함.

`file.h`

`file.c`

 file read 혹은 write를 disk sector의 read나 write로 translate해준다.

`lib/kernel/bitmap.h`

`lib/kernel/bitmap.c`

 bitmap을 읽고 disk file 에 쓰는 routine과 함께 bitmap data structure.

 이 file system은 unix-like interface를 가지고 있어서, create, open, close, read, write, lseek, unlink에 대한 Unix man page를 읽을 수 있어. 이 file system이 가지는 call은 비슷하지만 완전히 같지는 않아. 

 모든 basic functionality가 위에 code에 있어. 그래서 file system을 처음부터 사용할 수 있을텐데, 너가 지워야 할 한계를 가지고 있어.

 



#### 5.1.2 Testing File System Persistence

 [무시]



### 5.2 Suggested Order of Implemnetation

 [무시]



### 5.3 Requirements

#### 5.3.1 Design Docuement

 [무시]

#### 5.3.2 Indexed and Extensible Files

 external fragmentation이 있으니까 inode 구조를 수정해서 해결해라. 

#### 5.3.3 Subdirectories

 open이랑 create에서도 directory를 이용하도록. 


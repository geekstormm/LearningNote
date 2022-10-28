# Linux

## 进程

**概念**

程序是包含一系列信息的文件，这些信息描述了如何在运行时创建一个进程。

进程是正在运行的程序的实例。是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。它是操作系统动态执行的基本单元，在传统的操作系统中，进程既是基本的分配单元，也是基本的执行单元。

**可以用一个程序来创建多个进程**，进程是由内核定义的抽象实体，并为该实体分配用以执行程序的各项系统资源。从内核的角度看，进程由用户内存空间和一系列内核数据结构组成，其中用户内存空间包含了程序代码及代码所使用的变量，而内核数据结构则用于维护进程状态信息。记录在内核数据结构中的信息包括许多与进程相关的标 识号(IDs)、虚拟内存表、打开文件的描述符表、信号传递及处理的有关信息、进 程资源使用及限制、当前工作目录和大量的其他信息。

**单道、多道程序设计**

-  单道程序，即在计算机内存中只允许一个的程序运行。

-  多道程序设计技术是在计算机内存中同时存放几道相互独立的程序，使它们在管理程

  序控制下，相互穿插运行，两个或两个以上程序在计算机系统中同处于开始到结束之 间的状态, 这些程序共享计算机系统资源。引入多道程序设计技术的根本目的是为了 提高 CPU 的利用率。

**时间片**

时间片(timeslice)又称为“量子(quantum)”或“处理器片(processor slice)” 是操作系统分配给每个正在运行的进程微观上的一段 CPU 时间。事实上，虽然一台计算机通常可能有多个 CPU，但是同**一个 CPU 永远不可能真正地同时运行多个任务**。在 只考虑一个 CPU 的情况下，这些进程“看起来像”同时运行的，实则是轮番穿插地运行， 由于时间片通常很短(在 Linux 上为 5ms-800ms)，用户不会感觉到。

时间片由操作系统内核的调度程序分配给每个进程。首先，内核会给每个进程分配相等 的初始时间片，然后每个进程轮番地执行相应的时间，当所有进程都处于时间片耗尽的 状态时，内核会重新为每个进程计算并分配时间片，如此往复。

### 进程创建

```c
/*
    #include <sys/types.h>
    #include <unistd.h>

    pid_t fork(void);
        函数的作用：用于创建子进程。
        返回值：
            fork()的返回值会返回两次。一次是在父进程中，一次是在子进程中。
            在父进程中返回创建的子进程的ID,
            在子进程中返回0
            如何区分父进程和子进程：通过fork的返回值。
            在父进程中返回-1，表示创建子进程失败，并且设置errno

        父子进程之间的关系：
        区别：
            1.fork()函数的返回值不同
                父进程中: >0 返回的子进程的ID
                子进程中: =0
            2.pcb中的一些数据
                当前的进程的id pid
                当前的进程的父进程的id ppid
                信号集

        共同点：
            某些状态下：子进程刚被创建出来，还没有执行任何的写数据的操作
                - 用户区的数据
                - 文件描述符表
        
        父子进程对变量是不是共享的？
            - 刚开始的时候，是一样的，共享的。如果修改了数据，不共享了。
            - 读时共享（子进程被创建，两个进程没有做任何的写的操作），写时拷贝。
        
*/

#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    int num = 10;

    // 创建子进程
    pid_t pid = fork();

    // 判断是父进程还是子进程
    if(pid > 0) {
        // printf("pid : %d\n", pid);
        // 如果大于0，返回的是创建的子进程的进程号，当前是父进程
        printf("i am parent process, pid : %d, ppid : %d\n", getpid(), getppid());

        printf("parent num : %d\n", num);
        num += 10;
        printf("parent num += 10 : %d\n", num);
    } else if(pid == 0) {
        // 当前是子进程
        printf("i am child process, pid : %d, ppid : %d\n", getpid(),getppid());
       
        printf("child num : %d\n", num);
        num += 100;
        printf("child num += 100 : %d\n", num);
    }

    // for循环
    for(int i = 0; i < 3; i++) {
        printf("i : %d , pid : %d\n", i , getpid());
        sleep(1);
    }

    return 0;
}

/*
实际上，更准确来说，Linux 的 fork() 使用是通过写时拷贝 (copy- on-write) 实现。
写时拷贝是一种可以推迟甚至避免拷贝数据的技术。
内核此时并不复制整个进程的地址空间，而是让父子进程共享同一个地址空间。
只用在需要写入的时候才会复制地址空间，从而使各个进行拥有各自的地址空间。
也就是说，资源的复制是在需要写入的时候才会进行，在此之前，只有以只读方式共享。
注意：fork之后父子进程共享文件，
fork产生的子进程与父进程相同的文件文件描述符指向相同的文件表，引用计数增加，共享文件偏移指针。
*/
```

### 进程控制

进程退出exit()

```C
/*
    #include <stdlib.h>
    void exit(int status);

    #include <unistd.h>
    void _exit(int status);

    status参数：是进程退出时的一个状态信息。父进程回收子进程资源的时候可以获取到。
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {

    printf("hello\n");
    printf("world");

    // exit(0);
    _exit(0);
    
    return 0;
}
```

#### 孤儿进程

父进程运行结束，但子进程还在运行(未运行结束)，这样的子进程就称为孤儿进程 (Orphan Process)。

每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init ，而 init 进程会循环地 wait() 它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束了其生命周期的时候，init 进程就会代表党和政府出面处理它的一切善后工作。**因此孤儿进程并不会有什么危害。**

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    // 创建子进程
    pid_t pid = fork();

    // 判断是父进程还是子进程
    if(pid > 0) {
        printf("i am parent process, pid : %d, ppid : %d\n", getpid(), getppid());
    } else if(pid == 0) {
        sleep(1);
        // 当前是子进程
        printf("i am child process, pid : %d, ppid : %d\n", getpid(),getppid());
    }

    // for循环
    for(int i = 0; i < 3; i++) {
        printf("i : %d , pid : %d\n", i , getpid());
    }

    return 0;
}

```

#### 僵尸进程

每个进程结束之后, 都会释放自己地址空间中的用户区数据，内核区的 PCB 没有办法 自己释放掉，需要父进程去释放。

进程终止时，父进程尚未回收，子进程残留资源(PCB)存放于内核中，变成僵尸 (Zombie)进程。

僵尸进程不能被 kill -9 杀死，这样就会导致一个问题，如果父进程不调用 wait() 或 waitpid() 的话，那么保留的那段信息就不会释放，其进程号就会一直被占用， 但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进 程号而导致系统不能产生新的进程，此即为僵尸进程的危害，应当避免。

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    // 创建子进程
    pid_t pid = fork();

    // 判断是父进程还是子进程
    if(pid > 0) {
        while(1) {
            printf("i am parent process, pid : %d, ppid : %d\n", getpid(), getppid());
            sleep(1);
        }

    } else if(pid == 0) {
        // 当前是子进程
        printf("i am child process, pid : %d, ppid : %d\n", getpid(),getppid());
       
    }

    // for循环
    for(int i = 0; i < 3; i++) {
        printf("i : %d , pid : %d\n", i , getpid());
    }

    return 0;
}
```

#### wait()

```c
/*
    #include <sys/types.h>
    #include <sys/wait.h>
    pid_t wait(int *wstatus);
        功能：等待任意一个子进程结束，如果任意一个子进程结束了，此函数会回收子进程的资源。
        参数：int *wstatus
            进程退出时的状态信息，传入的是一个int类型的地址，传出参数。
        返回值：
            - 成功：返回被回收的子进程的id
            - 失败：-1 (所有的子进程都结束，调用函数失败)

    调用wait函数的进程会被挂起（阻塞），直到它的一个子进程退出或者收到一个不能被忽略的信号时才被唤醒（相当于继续往下执行）
    如果没有子进程了，函数立刻返回，返回-1；如果子进程都已经结束了，也会立即返回，返回-1.
*/
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>


int main() {
    // 有一个父进程，创建5个子进程（兄弟）
    pid_t pid;

    // 创建5个子进程
    for(int i = 0; i < 5; i++) {
        pid = fork();
        if(pid == 0) {
            break;
        }
    }

    if(pid > 0) {
        // 父进程
        while(1) {
            printf("parent, pid = %d\n", getpid());
            // int ret = wait(NULL);
            int st;
            int ret = wait(&st);
            if(ret == -1) {
                break;
            }

            if(WIFEXITED(st)) {
                // 是不是正常退出
                printf("退出的状态码：%d\n", WEXITSTATUS(st));
            }
            if(WIFSIGNALED(st)) {
                // 是不是异常终止
                printf("被哪个信号干掉了：%d\n", WTERMSIG(st));
            }
            printf("child die, pid = %d\n", ret);

            sleep(1);
        }

    } else if (pid == 0){
        // 子进程
         while(1) {
            printf("child, pid = %d\n",getpid());    
            sleep(1);       
         }
        exit(0);
    }
    return 0; // exit(0)
}
```

#### waitpid()

```c
/*
    #include <sys/types.h>
    #include <sys/wait.h>
    pid_t waitpid(pid_t pid, int *wstatus, int options);
        功能：回收指定进程号的子进程，可以设置是否阻塞。
        参数：
            - pid:
                pid > 0 : 某个子进程的pid
                pid = 0 : 回收当前进程组的所有子进程    
                pid = -1 : 回收所有的子进程，相当于 wait()  （最常用）
                pid < -1 : 某个进程组的组id的绝对值，回收指定进程组中的子进程
            - options：设置阻塞或者非阻塞
                0 : 阻塞
                WNOHANG : 非阻塞
            - 返回值：
                > 0 : 返回子进程的id
                = 0 : options=WNOHANG, 表示还有子进程或者
                = -1 ：错误，或者没有子进程了
*/
#include <sys/types.h>
#include <sys/wait.h>
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main() {
    // 有一个父进程，创建5个子进程（兄弟）
    pid_t pid;

    // 创建5个子进程
    for(int i = 0; i < 5; i++) {
        pid = fork();
        if(pid == 0) {
            break;
        }
    }

    if(pid > 0) {
        // 父进程
        while(1) {
            printf("parent, pid = %d\n", getpid());
            sleep(1);

            int st;
            // int ret = waitpid(-1, &st, 0);
            int ret = waitpid(-1, &st, WNOHANG);

            if(ret == -1) {
                break;
            } else if(ret == 0) {
                // 说明还有子进程存在
                continue;
            } else if(ret > 0) {
                if(WIFEXITED(st)) {
                    // 是不是正常退出
                    printf("退出的状态码：%d\n", WEXITSTATUS(st));
                }
                if(WIFSIGNALED(st)) {
                    // 是不是异常终止
                    printf("被哪个信号干掉了：%d\n", WTERMSIG(st));
                }

                printf("child die, pid = %d\n", ret);
            }
           
        }
    } else if (pid == 0){
        // 子进程
         while(1) {
            printf("child, pid = %d\n",getpid());    
            sleep(1);       
         }
        exit(0);
    }

    return 0; 
}
```

### 进程间通信

进程是一个独立的资源分配单元，不同进程（这里所说的进程通常指的是用户进程）之间的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源。但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要进程间通信( IPC：Inter Processes Communication )。

#### 进程间通信的目的：

- 数据传输：一个进程需要将它的数据发送给另一个进程。
- 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种 事件（如进程终止时要通知父进程）。
- 资源共享：多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同 步机制。
- 进程控制：有些进程希望完全控制另一个进程的执行（如 Debug 进程），此时控制 进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

#### 进程间通信的方式

![image-20220922223628098](/Users/lgs/Library/Application Support/typora-user-images/image-20220922223628098.png)

- **管道**

管道也叫无名（匿名）管道，它是是 UNIX 系统 IPC（进程间通信）的最古老形式， 所有的 UNIX 系统都支持这种通信机制。

管道其实是一个在内核内存中维护的缓冲器，这个缓冲器的存储能力是有限的，不同的操作系统大小不一定相同。

管道拥有文件的特质：读操作、写操作，匿名管道没有文件实体，有名管道有文件实体， 但不存储数据。可以按照操作文件的方式对管道进行操作。

一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。

通过管道传递的数据是顺序的，从管道中读取出来的字节的顺序和它们被写入管道的顺 序是完全一样的。

在管道中的数据的传递方向是单向的，一端用于写入，一端用于读取，管道是**半双工**的。

从管道读数据是一次性操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写 更多的数据，在管道中无法使用 lseek() 来随机的访问数据。

匿名管道只能在具有公共祖先的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用。

![pipe](/Users/lgs/Desktop/nowcoder-2/2.12/pipe.jpg)

```c
/*
    #include <unistd.h>
    int pipe(int pipefd[2]);
        功能：创建一个匿名管道，用来进程间通信。
        参数：int pipefd[2] 这个数组是一个传出参数。
            pipefd[0] 对应的是管道的读端
            pipefd[1] 对应的是管道的写端
        返回值：
            成功 0
            失败 -1

    管道默认是阻塞的：如果管道中没有数据，read阻塞，如果管道满了，write阻塞

    注意：匿名管道只能用于具有关系的进程之间的通信（父子进程，兄弟进程）
*/

// 子进程发送数据给父进程，父进程读取到数据输出
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
    // 在fork之前创建管道
    int pipefd[2];
    int ret = pipe(pipefd);
    if(ret == -1) {
        perror("pipe");
        exit(0);
    }

    // 创建子进程
    pid_t pid = fork();
    if(pid > 0) {
        // 父进程
        printf("i am parent process, pid : %d\n", getpid());

        // 关闭写端
        close(pipefd[1]);
        
        // 从管道的读取端读取数据
        char buf[1024] = {0};
        while(1) {
            int len = read(pipefd[0], buf, sizeof(buf));
            printf("parent recv : %s, pid : %d\n", buf, getpid());
            
            // 向管道中写入数据
            //char * str = "hello,i am parent";
            //write(pipefd[1], str, strlen(str));
            //sleep(1);
        }
    } else if(pid == 0){
        // 子进程
        printf("i am child process, pid : %d\n", getpid());
        // 关闭读端
        close(pipefd[0]);
        char buf[1024] = {0};
        while(1) {
            // 向管道中写入数据
            char * str = "hello,i am child";
            write(pipefd[1], str, strlen(str));
            //sleep(1);

            // int len = read(pipefd[0], buf, sizeof(buf));
            // printf("child recv : %s, pid : %d\n", buf, getpid());
            // bzero(buf, 1024);
        }
        
    }
    return 0;
}
```

![image-20220922224020289](/Users/lgs/Library/Application Support/typora-user-images/image-20220922224020289.png)



匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了有名管道（FIFO），也叫命名管道、FIFO文件。

有名管道（FIFO）不同于匿名管道之处在于它提供了一个路径名与之关联，以 FIFO 的文件形式存在于文件系统中，并且其打开方式与打开一个普通文件是一样的，这样即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此 通过 FIFO 相互通信，因此，通过 FIFO 不相关的进程也能交换数据。

一旦打开了 FIFO，就能在它上面使用与操作匿名管道和其他文件的系统调用一样的 I/O系统调用了（如read()、write()和close()）。与管道一样，FIFO 也有一 个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。FIFO 的 名称也由此而来：先入先出。

有名管道（FIFO)和匿名管道（pipe）有一些特点是相同的，不一样的地方在于： 

1. FIFO 在文件系统中作为一个特殊文件存在，但 FIFO 中的内容却存放在内存中。 
2. 当使用 FIFO 的进程退出后，FIFO 文件将继续保存在文件系统中以便以后使用。 
3. FIFO 有名字，不相关的进程可以通过打开有名管道进行通信。

```c
/*
    实现 ps aux | grep xxx 父子进程间通信
    
    子进程： ps aux, 子进程结束后，将数据发送给父进程
    父进程：获取到数据，过滤
    pipe()
    execlp()
    子进程将标准输出 stdout_fileno 重定向到管道的写端。  dup2
*/
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <wait.h>

int main() {
    // 创建一个管道
    int fd[2];
    int ret = pipe(fd);

    if(ret == -1) {
        perror("pipe");
        exit(0);
    }

    // 创建子进程
    pid_t pid = fork();

    if(pid > 0) {
        // 父进程
        // 关闭写端
        close(fd[1]);
        // 从管道中读取
        char buf[1024] = {0};

        int len = -1;
        while((len = read(fd[0], buf, sizeof(buf) - 1)) > 0) {
            // 过滤数据输出
            printf("%s", buf);
            memset(buf, 0, 1024);
        }
        wait(NULL);
    } else if(pid == 0) {
        // 子进程
        // 关闭读端
        close(fd[0]);

        // 文件描述符的重定向 stdout_fileno -> fd[1]
        dup2(fd[1], STDOUT_FILENO);
        // 执行 ps aux
        execlp("ps", "ps", "aux", NULL);
        perror("execlp");
        exit(0);
    } else {
        perror("fork");
        exit(0);
    }
    return 0;
}
```





## 网络编程

### 半关闭



### 端口复用


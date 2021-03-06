课后作业 (编码)

通过本作业，您可以探索一些使用锁和条件变量的实际代码，以实现本章中讨论的各种形式的生产者/消费者队列。 
您需要查看这些代码，以各种配置运行它，并使用它们来了解哪些方案有效，哪些无效，以及一些其他的问题。 阅读 README 文件了解详细信息。

问题：

<pre>
flag.s 作用见上面的注释,
这个简单的"锁有一个问题",导致它并不能保证互斥,
比如线程0执行 mov  flag, %ax 完后,时钟中断,切到线程1执行,
而线程1在执行 mov  %ax, count 中断,切到线程0,此时线程1是拥有锁的,
线程0继续执行 test $0, %ax ,这时ax的值是0,因为线程有单独的寄存器!!,所以线程1也获得了锁
</pre>    

1.我们的第一个问题集中在 main-two-cvs-while.c（有效的解决方案）上。 
首先，研究代码。 你认为你了解当你运行程序时会发生什么吗？

<pre>
❯ ./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v
 NF             P0 C0 
  0 [*---  --- ]    c0
  0 [*---  --- ] p0
  0 [*---  --- ]    c1
  0 [*---  --- ]    c2
  0 [*---  --- ] p1
  1 [u  0 f--- ] p4
  1 [u  0 f--- ] p5
  1 [u  0 f--- ] p6
  1 [u  0 f--- ]    c3
  1 [u  0 f--- ] p0
  0 [ --- *--- ]    c4
  0 [ --- *--- ]    c5
  0 [ --- *--- ]    c6
  0 [ --- *--- ] p1
  0 [ --- *--- ]    c0
  1 [f--- u  1 ] p4
  1 [f--- u  1 ] p5
  1 [f--- u  1 ] p6
  1 [f--- u  1 ]    c1
  1 [f--- u  1 ] p0
  0 [*---  --- ]    c4
  0 [*---  --- ]    c5
  0 [*---  --- ]    c6
  0 [*---  --- ] p1
  0 [*---  --- ]    c0
  1 [u  2 f--- ] p4
  1 [u  2 f--- ] p5
  1 [u  2 f--- ] p6
  1 [u  2 f--- ]    c1
  0 [ --- *--- ]    c4
  0 [ --- *--- ]    c5
  0 [ --- *--- ]    c6
  1 [f--- uEOS ] [main: added end-of-stream marker]
  1 [f--- uEOS ]    c0
  1 [f--- uEOS ]    c1
  0 [*---  --- ]    c4
  0 [*---  --- ]    c5
  0 [*---  --- ]    c6

Consumer consumption:
  C0 -> 3
</pre>

<br/>
<br/>

2.指定一个生产者和一个消费者运行，并让生产者产生一些元素。 
缓冲区大小从 1 开始，然后增加。随着缓冲区大小增加，程序运行结果如何改变？
当使用不同的缓冲区大小(例如 -m 10)，生产者生产不同的产品数量(例如 -l 100)，
修改消费者的睡眠字符串(例如 -C 0,0,0,0,0,0,1)，full_num 的值如何变化？

```shell script
./main-two-cvs-while -l 3 -m 1 -p 1 -c 1 -v
./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v
./main-two-cvs-while -l 3 -m 3 -p 1 -c 1 -v
./main-two-cvs-while -l 3 -m 4 -p 1 -c 1 -v

./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v
./main-two-cvs-while -l 6 -m 2 -p 1 -c 1 -v
./main-two-cvs-while -l 12 -m 2 -p 1 -c 1 -v
./main-two-cvs-while -l 24 -m 2 -p 1 -c 1 -v

./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v -C 0,0,0,0,0,0,1
./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v -C 1,0,2,0,0,0,1
./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v -C 0,1,0,0,0,0,1
./main-two-cvs-while -l 3 -m 2 -p 1 -c 1 -v -C 0,0,2,0,0,0,1
```

<br/>
<br/>

3.如果可能，请在其他系统（例如 Mac 和 Linux）上运行代码。您在这些系统上看到不同的行为了吗？

笔者只有 Linux 。。。

<br/>
<br/>

4.我们来看一些 timings。 对于一个生产者，三个消费者，大小为 1 的共享缓冲区以及每个消费者在 c3 点暂停一秒，您认需要执行多长时间？
（./main-two-cvs-while -p 1 -c 3 -m 1 -C 0,0,0,1,0,0,0:0,0,0,1,0,0,0:0,0,0,1,0,0,0 -l 10 -v -t）

    指定 3 个消费者，1 个生产者，缓冲区大小为 1，plainplainplainplain
    消费者睡眠点：c3、c3、c3，睡眠时间为 1 秒
    每个生产者循环 10 次
    
    如果消费者线程先执行，那么睡眠时间为 13s，
    如果生产者者线程先执行，那么睡眠时间为 12s
    
    实际结果：
    Total time: 12.01 seconds
    Total time: 13.01 seconds

<br/>
<br/>

5.现在将共享缓冲区的大小更改为 3（-m 3）。这对总时间有什么影响吗？

    Total time: 11.01 secondsplainplain
    Total time: 12.01 seconds

<br/>
<br/>

6.现在将睡眠点更改为 c6（这将模拟消费者从队列中取出某些东西然后对其进行处理），再次使用大小为 1 的缓冲区。
在这种情况下，您预计运行多长时间? (./main-two-cvs-while -p 1 -c 3 -m 1 -C 0,0,0,0,0,0,1:0,0,0,0,0,0,1:0,0,0,0,0,0,1 -l 10 -v -t)
    
     Total time: 5.00 secondsplain
     c6睡眠点已经释放锁了，其他线程可以进入临界区执行，
     而上面的题目，在 c3 处睡眠，其他线程无法执行，必须等待睡眠线程醒来后释放锁

<br/>
<br/>

7.最后再次将缓冲区大小更改为 3（-m 3）。您现在预计需要运行多长时间？

    Total time: 5.00 secondsplainplainplainplain

<br/>
<br/>

8.现在让我们看一下 main-one-cv-while.c。您是否可以假设只有一个生产者，
一个消费者和一个大小为 1 的缓冲区，配置一个睡眠字符串，让代码运行出现问题
    
    一个消费者一个生产者不会出现问题，见书P257plainplainplainplainplain

<br/>
<br/>

9.现在将消费者数量更改为两个。 为生产者消费者配置睡眠字符串，从而使代码运行出现问题。

    即使不配置睡眠字符串，也可能出现如下情况：plain
    生产者生产后，缓冲区满了，唤醒了两个正在睡眠的消费者中的一个，然后进入睡眠（Mutex_lock）
    消费者消费后，唤醒另一个消费者，进入睡眠（Mutex_lock），
    新的消费者线程被唤醒，发现缓冲区为空，进入睡眠（Cond_wait）,此时三个线程都进入睡眠
    
    无法配置睡眠字符串，使得代码运行必定出现问题，取决与操作系统的线程调度

<br/>
<br/>

10.现在查看 main-two-cvs-if.c。 您是否可以配置一些参数让代码运行出现问题？ 
再次考虑只有一个消费者的情况，然后再考虑有一个以上消费者的情况。

    一个消费者一个生产者不会出现问题plain
    出现问题的情况：
    一个生产者，两个消费者
    生产者生产完成时，消费者 c1 还没有进入临界区，消费者 c2 在 Cond_wait 处等待，
    生产者唤醒一个消费者，c1 抢先执行，执行完后缓冲区为空，c2 开始执行，发现缓冲区为空，do_get 执行发生错误！
    
    ./main-two-cvs-if -m 1 -c 2 -p 1 -l 10 -C 2:0,0,0,3 -P 1 会出现错误

<br/>
<br/>

11.最后查看 main-cvs-while-extra-unlock.c。在向缓冲区添加或取出元素时释放锁时会出现什么问题？ 
给定睡眠字符串来引起这类问题的发生？ 会造成什么不好的结果？
    

    do_get 和 do_fill 在锁外面，这锁等于没加plainplain
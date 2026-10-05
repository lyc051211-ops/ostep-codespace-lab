]633;E;for i in 1 2 3 4 5 6 7 8;eca3a8e6-72fb-49c6-a786-96218f2335d2]633;C## Q1
- Prediction / 预测:进程0先连续跑完5条CPU指令，全部结束之后才运行进程1；总时间10，CPU利用率100%，没有IO。
- Reasoning / 理由:调度策略SWITCH_ON_END，只能等一个进程完全结束之后才切换到另一个进程。两个进程全部都是CPU运算，不会触发IO，中间不会发生调度。
- Verified result / 验证结果:Time       PID: 0      PID: 1      CPU    IOs
1          RUN:cpu     READY       1
2          RUN:cpu     READY       1
3          RUN:cpu     READY       1
4          RUN:cpu     READY       1
5          RUN:cpu     READY       1
6          DONE        RUN:cpu     1
7          DONE        RUN:cpu     1
8          DONE        RUN:cpu     1
9          DONE        RUN:cpu     1
10         DONE        RUN:cpu     1

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
- Analysis / 分析:预测与模拟结果一致。SWITCH_ON_END策略不会中途切换进程，等到进程0结束才运行进程1。

## Q2
- Prediction / 预测:我预测总时间大约40，CPU利用率50%。每次进程发起IO就发生进程切换，两个进程来回交替执行，IO时间很长，CPU会大量等待。
- Reasoning / 理由:SWITCH_ON_IO策略：进程发出IO的时候立刻切换另一个就绪进程；IO完成之后不会抢占CPU。两个进程每执行1条指令就进入IO阻塞，互相切换，CPU利用率会下降。
- Verified result / 验证结果:Total time: 40, CPU utilization: 50.00%
- Analysis / 分析:模拟结果符合预测。每次调用IO就切换进程，但是IO结束后不会抢占，大量时间花费在IO等待上，CPU只有一半时间忙碌。

## Q3
- Prediction / 预测:开启IO_RUN_IMMEDIATE之后，IO一完成进程就抢占CPU。总时间相比Q2会变短，CPU利用率会高于50%，预估总时间大约37。
- Reasoning / 理由:SWITCH_ON_IO加上IO_RUN_IMMEDIATE。当IO完成时，刚刚结束IO的进程会立刻抢占CPU，不需要排在就绪队列后面，调度顺序发生改变，可以减少一部分空闲等待时间。
- Verified result / 验证结果:Total time: 37, CPU utilization: 54.05%
- Analysis / 分析:运行结果符合预期。IO完成抢占CPU让整体执行时间从40降到37，CPU利用率得到小幅提升；但IO本身耗时依旧很长，利用率提升幅度有限

## Q4
- Prediction / 预测:IO_RUN_LATER不会抢占CPU，IO结束后的进程放到就绪队列末尾。总时间和CPU利用率大概率跟Q3接近，大约总时间37，CPU利用率54%左右。
- Reasoning / 理由:依旧使用SWITCH_ON_IO策略，IO完成之后选择IO_RUN_LATER。IO结束的进程**不能立刻抢占CPU**，而是加入就绪队列尾部排队等待调度。
- Verified result / 验证结果:Total time: 37, CPU utilization: 54.05%
- Analysis / 分析:模拟运行结果和Q3数值完全一样。在这一组测试输入下，抢占（IMMEDIATE）和放到队尾（LATER）得到了相同总耗时与CPU利用率；并不是所有场景两者结果都会一样，换别的任务序列结果就可能发生变化。

## Q5
- Prediction / 预测:我预测总运行时间是7，CPU利用率会高于SWITCH_ON_END模式。
- Reasoning / 理由:当进程1发起IO请求的时候，SWITCH_ON_IO会立刻切换CPU。进程1等待IO的这段时间里，进程2可以使用CPU运行。
IO等待和CPU计算可以并行重叠，减少整体运行耗时。
- Verified result / 验证结果:Total time: 7, CPU utilization: 85.71%, IO utilization: 71.43%
- Analysis / 分析:对比SWITCH_ON_END，SWITCH_ON_IO实现了IO与CPU计算的重叠。
任务总执行时间变短，CPU利用率得到明显提升。

## Q6
- Prediction / 预测:我猜测开启IO_RUN_IMMEDIATE之后，IO一结束就会抢占CPU切回进程1，总时间有可能会变化。
- Reasoning / 理由:IO_RUN_IMMEDIATE的作用：IO完成的一瞬间，立刻抢占CPU跑刚结束IO的进程。但本测试里，进程2在IO等待的5个时间片之内就全部执行完了。等到IO结束的时候进程2早就跑完了，没有任务可以被打断。
- Verified result / 验证结果:Total time: 7, CPU utilization: 85.71%, IO utilization: 71.43%
- Analysis / 分析:和Q5结果一模一样。
原因是IO完成那一刻进程2已经结束，抢占功能没有机会生效。
只有当IO还没结束、进程2还没有跑完的时候，开启IO_RUN_IMMEDIATE才会改变结果。

## Q7
- Prediction / 预测:开启IO_RUN_IMMEDIATE抢占之后，一旦某个进程IO完成就立刻拿回CPU，整体的总运行时间应该会变短。
- Reasoning / 理由:现在一共有3个进程在跑。IO_RUN_IMMEDIATE会在IO结束的时候，马上抢占CPU，优先执行刚做完IO的进程，而不是把时间片用完再切换。
抢占会改变调度顺序，任务之间可以更好的重叠，从而改变总耗时和CPU利用率。
- Verified result / 验证结果:Total time: 24, CPU utilization: 50.00%
- Analysis / 分析:因为设置了IO抢占，每当一个进程IO结束，就打断当前正在跑的任务优先跑它。
抢占改变了任务执行的先后顺序。
可以看到多进程场景下IO_RUN_IMMEDIATE就真正生效了，不再和普通版本结果保持一样。抢占会影响最终调度结果。

## Q8
- Prediction / 预测:改成IO_RUN_LATER之后不会抢占CPU。总时间可能不变，但是IO的利用率应该跟Q7会有差别。
- Reasoning / 理由:IO_RUN_LATER就是进程IO做完之后，**不马上抢CPU跑**。
先让现在CPU上面正在跑的任务把这一轮时间用完，之后才轮到刚结束IO的进程。Q7是一做完IO就抢CPU，这两个方式调度的顺序不一样。
- Verified result / 验证结果:Total time: 24, CPU utilization: 50.00%, IO utilization: 79.17%
- Analysis / 分析:
对比Q7：总的跑完时间、CPU占用率一模一样，但是IO利用率不一样了。
就算总耗时没变，抢占开关还是会影响IO能不能叠在一起并行跑。
IMMEDIATE：IO一结束立刻换任务；LATER：等当前时间片跑完再换任务。两种方式IO重叠效果不一样。
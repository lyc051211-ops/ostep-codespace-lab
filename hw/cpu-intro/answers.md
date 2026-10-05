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
- Prediction / 预测:
- Reasoning / 理由:
- Verified result / 验证结果:
- Analysis / 分析:

## Q6
- Prediction / 预测:
- Reasoning / 理由:
- Verified result / 验证结果:
- Analysis / 分析:

## Q7
- Prediction / 预测:
- Reasoning / 理由:
- Verified result / 验证结果:
- Analysis / 分析:

## Q8
- Prediction / 预测:
- Reasoning / 理由:
- Verified result / 验证结果:
- Analysis / 分析:

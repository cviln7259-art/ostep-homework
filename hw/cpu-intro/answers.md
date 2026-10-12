# OSTEP Ch.4 Homework Answers

## Q1
- Prediction / 预测:
Time PID0     PID1    CPU IOs
1    RUN:cpu  READY  1
2    READY    RUN:cpu 1
3    RUN:cpu  READY  1
4    READY    RUN:cpu 1
5    RUN:cpu  READY  1
6    READY    RUN:cpu 1
7    RUN:cpu  READY  1
8    READY    RUN:cpu 1
9    RUN:cpu  READY  1
10   READY    RUN:cpu 1总时间 = 10 tick，CPU利用率 = 100%
- Reasoning / 理由:
两个进程都是5:100，全部为CPU指令，没有I/O。系统默认SWITCH_ON_IO，进程用完CPU时间片就切换。两个进程交替占用CPU，CPU全程不会空闲。PID0执行5条CPU指令，PID1执行5条CPU指令，合计10个时间片。
- Verified result / 验证结果:
Total Time:10，CPU Busy 10 (100%)
- Analysis / 分析:
预测与模拟器运行结果一致。两个纯CPU进程交替调度，CPU没有空闲周期，总时间等于两个进程CPU指令数量之和。

## Q2
- Prediction / 预测:
Time PID0     PID1     CPU IOs
1    RUN:cpu  READY    1
2    RUN:cpu  READY    1
3    RUN:cpu  READY    1
4    RUN:cpu  READY    1
5    DONE     RUN:io   1
6    DONE     BLOCKED      1
7    DONE     BLOCKED      1
8    DONE     BLOCKED      1
9    DONE     BLOCKED      1
10   DONE     BLOCKED      1
11   DONE     RUN:io_done 1总时间 =11 tick，CPU利用率 = (4+2)/11 ≈54.55%
- Reasoning / 理由:
PID0：4条CPU指令；PID1：1条I/O指令。
PID0先连续执行4个tick结束；之后PID1开始I/O：RUN:io(1tick) → BLOCKED(5tick) → RUN:io_done(1tick)。
PID0结束后没有其他就绪进程，I/O阻塞阶段CPU空闲。I/O完整开销7 tick，前面PID0占4 tick，总4+7=11。
- Verified result / 验证结果:
Total Time:11，CPU Busy 6 (54.55%)
- Analysis / 分析:
预测与运行结果一致。PID0执行完成之后才轮到PID1运行；PID1进入BLOCKED时没有其他就绪进程，CPU空闲。

## Q3
- Prediction / 预测:
Time PID0      PID1     CPU IOs
1    RUN:io    READY   1
2    BLOCKED   RUN:cpu     1
3    BLOCKED   RUN:cpu     1
4    BLOCKED   RUN:cpu     1
5    BLOCKED   RUN:cpu     1
6    BLOCKED   DONE        1
7*   RUN:io_done DONE  1总时间 =7 tick，CPU利用率 = (2+4)/7 ≈85.71%
- Reasoning / 理由:
进程顺序调换，PID0执行I/O。t1 PID0执行RUN:io发起I/O，立刻切换到PID1（SWITCH_ON_IO默认策略）。
PID0进入BLOCKED的5个tick期间，CPU调度PID1执行4条CPU指令。PID1在I/O阻塞期间就运行完毕。I/O完成后t7执行RUN:io_done。
I/O等待时间被PID1的CPU计算重叠，总时间缩短为7 tick。
- Verified result / 验证结果:
Total Time:7，CPU Busy 6 (85.71%)
- Analysis / 分析:
预测正确。当进程发起I/O发生阻塞，操作系统切换到另一个就绪进程，CPU和I/O设备并行工作，减少总完成时间。对比Q2，仅仅调换进程顺序，总时间大幅降低。

## Q4
- Prediction / 预测:
Time PID0      PID1     CPU IOs
1    RUN:io    READY   1
2    BLOCKED   READY       1
3    BLOCKED   READY       1
4    BLOCKED   READY       1
5    BLOCKED   READY       1
6    BLOCKED   READY       1
7*   RUN:io_done READY 1
8    DONE      RUN:cpu 1
9    DONE      RUN:cpu 1
10   DONE      RUN:cpu 1
11   DONE      RUN:cpu 1总时间 =11 tick，CPU利用率 = (2+4)/11≈54.55%
- Reasoning / 理由:
参数 `-S SWITCH_ON_END`：**只有进程全部执行完毕才切换进程**。
PID0发起I/O进入BLOCKED，操作系统不做上下文切换。虽然PID1处于READY状态，但CPU保持空闲，不会跑PID1。
等到PID0整个I/O全部完成、PID0结束之后，CPU才调度PID1执行4条CPU指令。总时间等于I/O完整7 tick + PID1的4 tick =11。
- Verified result / 验证结果:
Total Time:11，CPU Busy 6 (54.55%)
- Analysis / 分析:
预测正确。SWITCH_ON_END策略下，进程阻塞I/O时不会切换CPU，就绪进程得不到运行，CPU大量空闲，无法利用I/O等待时间做有用计算，总时间变长。

## Q5
- Prediction / 预测:
Time PID0      PID1     CPU IOs
1    RUN:io    READY   1
2    BLOCKED   RUN:cpu     1
3    BLOCKED   RUN:cpu     1
4    BLOCKED   RUN:cpu     1
5    BLOCKED   RUN:cpu     1
6    BLOCKED   DONE        1
7*   RUN:io_done DONE  1总时间 =7 tick，CPU利用率 = (2+4)/7≈85.71%
- Reasoning / 理由:
参数 `-S SWITCH_ON_IO`，进程发起I/O就立刻切换就绪进程。
PID0发起I/O后进入BLOCKED，CPU马上调度PID1。PID1的4条CPU指令完全和PID0的I/O阻塞时间重叠。I/O完成之后PID0执行io_done，全部结束。和Q3行为一致。对比Q4，相同任务，切换策略不同总时间差距巨大。
- Verified result / 验证结果:
Total Time:7，CPU Busy 6 (85.71%)
- Analysis / 分析:
预测正确。SWITCH_ON_IO在进程发起I/O就切换，让CPU与I/O设备并行，充分利用I/O等待时间，缩短整体作业完成时间。

## Q6
- Prediction / 预测:
PID0：3条I/O指令；PID1 PID2 PID3 都是5条CPU指令。参数 `-S SWITCH_ON_IO -I IO_RUN_LATER`
IO_RUN_LATER：I/O完成，进程放到READY队列尾部，**不会立刻抢占CPU**。当PID0的I/O完成，如果还有其他就绪CPU密集进程，CPU继续运行其他进程。I/O设备在PID0不发起新I/O的时候会空闲。总时间会比较长。
- Reasoning / 理由:
PID0每一条I/O：RUN:io(1) + BLOCKED(5) + RUN:io_done(1)。每次I/O完成PID0变为READY，排在就绪队列末尾，系统继续跑CPU‑bound进程。CPU优先把CPU密集进程跑完，再继续处理PID0剩下I/O相关指令。I/O设备会出现空闲时间段。
- Verified result / 验证结果:
（复制你本地q6.txt的 Total Time、CPU Busy 百分比）
- Analysis / 分析:
（预测总时间偏差，IO_RUN_LATER不会抢占CPU，I/O完成后PID0要等其他就绪进程执行完才得到CPU，I/O设备存在空闲，系统吞吐量下降

## Q7
- Prediction / 预测:
进程配置同Q6，参数 `-S SWITCH_ON_IO -I IO_RUN_IMMEDIATE`。
IO_RUN_IMMEDIATE：I/O设备完成之后，PID0立刻抢占CPU，马上运行RUN:io_done，不需要排到就绪队列尾部。可以尽快发起下一次I/O，让I/O设备保持忙碌，减少I/O设备空闲。整体总时间比Q6更短。
- Reasoning / 理由:
PID0的I/O一完成，立刻获得CPU执行io_done，马上发起下一轮I/O。I/O设备可以连续工作，减少设备闲置。适合I/O密集型程序，提升I/O设备利用率，缩短整体完成时间。
- Verified result / 验证结果:
（复制你本地q7.txt Total Time、CPU Busy百分比）
- Analysis / 分析:
（对比Q6结果，分析IO_RUN_IMMEDIATE优势：I/O密集进程抢占CPU，快速发起后续I/O，硬件设备利用率更高；代价：CPU密集进程会被频繁打断）

## Q8
- Prediction / 预测:
进程定义 `-l 3:50,3:50`，每个指令50%概率CPU或者I/O。分别使用seed=1,2,3；对比default、‑I IO_RUN_IMMEDIATE、‑S SWITCH_ON_END三种配置。
不同种子生成不同的I/O/CPU指令序列。SWITCH_ON_END会造成CPU空闲；IO_RUN_IMMEDIATE会优先调度I/O完成进程。不同参数会改变总运行时间、CPU与I/O利用率。
- Reasoning / 理由:
`‑s seed`决定随机生成每个进程是CPU指令还是I/O指令。
1. SWITCH_ON_END：进程I/O阻塞时不切换CPU，CPU容易空闲，总时间增加。
2. IO_RUN_IMMEDIATE：I/O完成立刻运行对应进程，有利于I/O设备流水线工作。
3. IO_RUN_LATER(default)：I/O完成放入就绪队列末尾。
- Verified result / 验证结果:
（读取 q8‑s1.txt q8‑s2.txt q8‑s3.txt，复制每个seed下三组设置的Total Time、CPU Busy）
- Analysis / 分析:
调度参数对总时间.设备利用率带来的影响

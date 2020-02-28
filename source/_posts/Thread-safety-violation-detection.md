---
title: Efficient Scalable Thread-Safety-Violation Detection
date: 2019-11-13 09:48:24
tags:
---

##### 基本思想：
1. 列出线程不安全的常用类及其方法，如[array add:]
2. hook这些方法，在hook中随机插入延时，以让可能发生冲突的地方更容易发生冲突
##### 特点：
1. 自适应策略，不需要跑多次即可发现很多问题
2. overhead较低，可与工程的单元测试一起跑
3. False positive极少
##### 

```
//Hook方法
OnCall(thread_id, obj_id, op_id) {
    check_for_trap(thread_id, obj_id, op_id)
    if (should_delay(op_id)) {
        set_trap(thread_id, obj_id, op_id)
        delay()
        clear_trap(thread_id, obj_id, op_id)
    }
}
```
检测冲突点：
1. 两个操作在不同线程运行
2. 至少有一个是写操作

##### 关键点：
1. Where to inject delays?
    * 动态维护一个trap set,
    1. Identify near misses
        ```
        <tid1, obj1, op1, t1>
        <tid2, obj2, op2, t2>
        
        Dangerous if:
            tid1 != tid2
            obj1 == obj2
            op1 == write || op2 == write
            |t1 - t2| <= sigma
        ```
        * TSVD maintains a list of accesses per object since `sigma` time ago
    2. Inferring Concurrent Phases
        * TSVD infers concurrent execution phases by checking the execution history of TSVD points
        * It maintains a global history buffer with a fixed number of most recently executed TSVD points.
        * If the TSVD points in this buffer fcome from more than one thread, TSVD considers the execution to be inside a concurrent phase
    3. Inferring likely HB (happens-before) relationship
        * 因果关系：If loc1发生在loc2之前，则loc1之前的delay将导致loc2也成比例地delay
        * TSVD maintains a history of the most recent access made by every thread
        ```
        设：loc1之前插入一个延迟d1, 时间为[t1_start, t1_end]
        随后，另一个线程Thd2在t2时刻访问了loc2
        we'll check Thd2上次访问发生什么时候，记为t0
        
        如果：
        1. t2 - t0 >= sigma_hb * delay_time ;t0和t2之间有个long gap
        2. t0 <= t1_end ;此long gap与injected delay有重合
        
        则认为：loc1和loc2有冲突
        ```
        
    4. Delay Decaying
        * 对于每个dangerous pair {p1, p2}, every injection at either p1 or p2 that fails to make p1 and p2 execute in parallel with the same object accessed will cause TSVD to decay the probability of future delay injection at p1 and p2
        * when probability drops to 0, it's removed from the trap set
2. When to inject delays?
    1. Planning & injection in the same run
        * loc执行前，TSVD检查loc是否在trap set中，若是，则insert a delay at loc with probability P_loc
    2. Multiple testing runs
        * 有些问题只出现一次        
        * 第一次运行生成trap file，第二次运行使用
    3. Parallel delay injection
        * 不限制同一时刻只inject一个delay，多个loc可能都同时inject了delay


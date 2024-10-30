## 定位高CPU占用
* 执行 top 命令，定位高 CPU 占用的 PID
```
  PID USER  PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+     COMMAND
10834 hdfs  20   0 6125996   3.4g   6800 S  107.0  5.4    343:35.01  /usr/java/jdk1.8...

```


* 执行 ps -mp PID -o THREAD,tid,time 命令查看线程耗时情况
```
ps -mp 10834 -o THREAD,tid,time
USER     %CPU PRI SCNT WCHAN  USER SYSTEM   TID     TIME
hdfs      0.2   -    - -         -      -     - 05:43:35
hdfs      0.0  19    - futex_    -      - 11013 00:47:47 --以该线程长耗时为例分析
hdfs      0.0  19    - futex_    -      - 11014 00:01:21
hdfs      0.0  19    - futex_    -      - 11035 00:00:07
hdfs      0.0  19    - futex_    -      - 11037 00:20:04
hdfs      0.0  19    - ep_pol    -      - 11401 00:04:52
hdfs      0.0  19    - ep_pol    -      - 11402 00:04:31
hdfs      0.0  19    - futex_    -      - 11403 00:21:01
```

* 执行 printf "%x\n" TID 将 tid 转换为十六进制
```
printf "%x\n" 11013
2b05
```

* 执行 jstack PID |grep TID -A 30 定位具体线程
  * nid: Native thread id  --> TID
  * tid: Java thread id
* 扩展
```
根据实际线程情况定位相关代码，如果定位到 GC 相关线程引起高 CPU 问题，可使用 jstat 相关命令观察 GC 情况
例如： jstat -gcutil -t -h 5 PID 500 10
```

## 高内存占用、内存泄漏
* 定位 PID
* 使用本机内存跟踪（NMT）追踪内存变化情况
  * `-XX:NativeMemoryTracking=[off|summary|detail]`
  
    | jcmd NMT Option | 描述 |
    |-----------------|-----|
    | off | 关闭，默认处于该状态|
    | summary | 收集摘要信息 |
    | detail | 收集详细信息 |
  
  * `jcmd <pid> VM.native_memory [summary|detail|baseline|summary.diff|detail.diff|shutdown] [scale= KB|MB|GB]`
  
    | jcmd NMT Option | 描述 |
    |-----------------|------|
    | summary	| 打印摘要信息 |
    | detail | 打印详细信息 |
    | baseline | 创建一个新的内存使用情况基准快照以进行比较 |
    | summary.diff | 根据最后一个基准打印新的摘要报告 |
    | detail.diff | 根据最后一个基准打印新的详细报告 |
    | shutdown | 停止本机内存跟踪 |
  
* 分析堆直方图、生成 dump 文件
  * 使用 jcmd 命令分析 
    * 命令查看堆直方图: `jcmd PID GC.class_histogram`
    * 生成dump: `jcmd PID GC.heap_dump filename=filename`
  * 使用 jmap 命令分析
    * 命令查看堆直方图: `jmap -histo:live PID`
    * 生成dump: `jmap -dump:live,format=b,file=test.dump PID`
* GC 相关问题诊断
  * 执行 `jstat -gcutil -t -h 5 PID 500 10` 查看各内存区域占用情况
    ```
    jstat -gcutil -t -h 5 1174 500 10
    Timestamp         S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
             1138.7   0.00 100.00  25.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1139.2   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1139.8   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1140.3   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1140.8   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
    Timestamp         S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
             1141.3   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1141.8   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1142.3   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1142.8   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
             1143.3   0.00 100.00  50.00  22.32  94.42  85.09     29    0.064     4    0.067    0.131
    ```
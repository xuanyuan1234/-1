# Linux常用命令


## top整机

* %CPU %MEM 
* load average: A B C。其中，ABC分别代表：1分钟、5分钟、15分钟的平均负载。如果(A+B+C)/3 > 60%，代表系统负载压力大

## uptime 精简版top

## vmstat (cpu)

* vmstat -n 2 3 。每2秒采样，采样3次

## pidstat (cpu)

* mpstat -p ALL 2
* pidstat -u 1 -p pid

## free

* free [-g] [-m]

## 磁盘IO

* iostat -xdk 2 3
* %util接近100%，表示磁盘带宽饱和
* pidstat -d 2 -p pid

## 网络IO

* ifstat 1

## 如果CPU占用过高，具体定位分析

* 结合linux和jdk命令一起分析
* 先用top命令找出CPU占比最高的
* ps -ef | jps -l，得知具体程序 （ps -ef | grep java | grep -v grep）
* 定位具体线程或代码  ps -mp 进程 -o THREAD,tid,time
* 将线程ID转为16进制小写
* jstack 进程ID | grep tid -A60

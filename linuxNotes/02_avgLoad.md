# 平均负载

指单位时间内，系统处于可运行状态和不可中断状态的平均进程数，即平均活跃进程数。

```bash
# 查看cpu
$ grep 'model name' /proc/cpuinfo
# 安装压测工具和性能分析工具包
$ sudo apt install stress sysstat
```

+ mpstat 是一个多核CPU性能分析工具，用来实时查看每个CPU的性能指标、以及所有CPU的平均指标 -> 多核状态
+ pidstat 是一个常用的进程性能分析工具，用来实时查看进程的CPU、内存、IO、上下文切换等 -> 进程状态

## 实验前

```bash
root@ubuntu:/home/zhouweilin# uptime
 22:16:48 up 39 min,  1 user,  load average: 0.11, 0.03, 0.01
```

## 实验一：CPU密集型

```bash
# tty1 压测
root@ubuntu:/home/zhouweilin# stress --cpu 1 --timeout 600
stress: info: [4160] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd
# tty2 查看负载情况
zhouweilin@ubuntu:~$ watch -d uptime
Every 2.0s: uptime                                      Mon Mar 11 22:24:54 2019

 22:25:12 up 46 min,  1 user,  load average: 0.06, 0.30, 0.17

# tty3 查看CPU使用率变化情况
zhouweilin@ubuntu:~$ mpstat -P ALL 5
10:20:57 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
10:21:02 PM  all   50.45    0.00    0.20    0.00    0.00    0.00    0.00    0.00    0.00   49.35
10:21:02 PM    0  100.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
10:21:02 PM    1    0.80    0.00    0.60    0.00    0.00    0.00    0.00    0.00    0.00   98.60

# tty4 查看是哪个进程导致CPU使用率为100%
zhouweilin@ubuntu:~$ pidstat -u 5 1
Linux 4.15.0-45-generic (ubuntu) 	03/11/2019 	_x86_64_	(2 CPU)

10:22:01 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
10:22:09 PM     0       963    0.40    0.20    0.00    0.60     1  Xorg
10:22:09 PM  1000      1631    0.20    0.20    0.00    0.40     1  compiz
10:22:09 PM  1000      2039    0.20    0.00    0.00    0.20     1  gnome-terminal-
10:22:09 PM     0      3257    0.00    0.20    0.00    0.20     1  kworker/1:1
10:22:09 PM     0      4161  110.76    0.00    0.00  110.76     0  stress
10:22:09 PM  1000      4488    0.00    0.20    0.00    0.20     1  pidstat

Average:      UID       PID    %usr %system  %guest    %CPU   CPU  Command
Average:        0       963    0.40    0.20    0.00    0.60     -  Xorg
Average:     1000      1631    0.20    0.20    0.00    0.40     -  compiz
Average:     1000      2039    0.20    0.00    0.00    0.20     -  gnome-terminal-
Average:        0      3257    0.00    0.20    0.00    0.20     -  kworker/1:1
Average:        0      4161  110.76    0.00    0.00  110.76     -  stress
Average:     1000      4488    0.00    0.20    0.00    0.20     -  pidstat
```

## 实验二：IO密集型

```bash
# tty1
root@ubuntu:/home/zhouweilin# stress -i 1 --timeout 600
stress: info: [4998] dispatching hogs: 0 cpu, 1 io, 0 vm, 0 hdd
# tty2
Every 2.0s: uptime                                      Mon Mar 11 22:31:57 2019

 22:32:13 up 53 min,  1 user,  load average: 0.90, 0.45, 0.25
# tty3
root@ubuntu:/home/zhouweilin# mpstat -P ALL 5 1
Linux 4.15.0-45-generic (ubuntu) 	03/11/2019 	_x86_64_	(2 CPU)

10:30:56 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
10:31:01 PM  all    2.10    0.00   49.05    0.00    0.00    0.10    0.00    0.00    0.00   48.75
10:31:01 PM    0    2.60    0.00   97.40    0.00    0.00    0.00    0.00    0.00    0.00    0.00
10:31:01 PM    1    1.60    0.00    0.80    0.00    0.00    0.20    0.00    0.00    0.00   97.41

Average:     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
Average:     all    2.10    0.00   49.05    0.00    0.00    0.10    0.00    0.00    0.00   48.75
Average:       0    2.60    0.00   97.40    0.00    0.00    0.00    0.00    0.00    0.00    0.00
Average:       1    1.60    0.00    0.80    0.00    0.00    0.20    0.00    0.00    0.00   97.41
# tty4
root@ubuntu:/home/zhouweilin# pidstat -u 5 1
Linux 4.15.0-45-generic (ubuntu) 	03/11/2019 	_x86_64_	(2 CPU)

10:31:01 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
10:31:09 PM     0       963    0.40    0.00    0.00    0.40     1  Xorg
10:31:09 PM  1000      1457    0.20    0.00    0.00    0.20     1  ibus-ui-gtk3
10:31:09 PM  1000      1631    0.40    0.00    0.00    0.40     1  compiz
10:31:09 PM  1000      2039    0.20    0.00    0.00    0.20     1  gnome-terminal-
10:31:09 PM     0      4841    0.00    0.20    0.00    0.20     1  kworker/1:0
10:31:09 PM     0      4999    2.20  108.78    0.00  110.98     0  stress

Average:      UID       PID    %usr %system  %guest    %CPU   CPU  Command
Average:        0       963    0.40    0.00    0.00    0.40     -  Xorg
Average:     1000      1457    0.20    0.00    0.00    0.20     -  ibus-ui-gtk3
Average:     1000      1631    0.40    0.00    0.00    0.40     -  compiz
Average:     1000      2039    0.20    0.00    0.00    0.20     -  gnome-terminal-
Average:        0      4841    0.00    0.20    0.00    0.20     -  kworker/1:0
Average:        0      4999    2.20  108.78    0.00  110.98     -  stress
```

## 实验三： 大量进程场景

```bash
# tty1
root@ubuntu:/home/zhouweilin# stress -c 8 --timeout 600
stress: info: [5379] dispatching hogs: 8 cpu, 0 io, 0 vm, 0 hdd
# tty2
root@ubuntu:/home/zhouweilin# uptime
 22:37:17 up 57 min,  1 user,  load average: 5.82, 2.21, 0.94
# tty3
root@ubuntu:/home/zhouweilin# mpstat -P ALL 5 1
Linux 4.15.0-45-generic (ubuntu) 	03/11/2019 	_x86_64_	(2 CPU)

10:37:29 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
10:37:34 PM  all   99.40    0.00    0.60    0.00    0.00    0.00    0.00    0.00    0.00    0.00
10:37:34 PM    0   99.60    0.00    0.40    0.00    0.00    0.00    0.00    0.00    0.00    0.00
10:37:34 PM    1   99.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00

Average:     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
Average:     all   99.40    0.00    0.60    0.00    0.00    0.00    0.00    0.00    0.00    0.00
Average:       0   99.60    0.00    0.40    0.00    0.00    0.00    0.00    0.00    0.00    0.00
Average:       1   99.00    0.00    1.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00
# tty4
root@ubuntu:/home/zhouweilin# pidstat -u 5 1
Linux 4.15.0-45-generic (ubuntu) 	03/11/2019 	_x86_64_	(2 CPU)

10:37:32 PM   UID       PID    %usr %system  %guest    %CPU   CPU  Command
10:37:37 PM     0       963    1.00    0.40    0.00    1.40     1  Xorg
10:37:37 PM  1000      1415    0.00    0.20    0.00    0.20     0  ibus-daemon
10:37:37 PM  1000      1457    0.00    0.20    0.00    0.20     0  ibus-ui-gtk3
10:37:37 PM  1000      1631    0.40    0.20    0.00    0.60     1  compiz
10:37:37 PM  1000      2039    1.00    0.00    0.00    1.00     1  gnome-terminal-
10:37:37 PM     0      5380   27.54    0.00    0.00   27.54     0  stress
10:37:37 PM     0      5381   26.75    0.20    0.00   26.95     1  stress
10:37:37 PM     0      5382   27.74    0.00    0.00   27.74     0  stress
10:37:37 PM     0      5383   26.95    0.00    0.00   26.95     1  stress
10:37:37 PM     0      5384   26.75    0.00    0.00   26.75     1  stress
10:37:37 PM     0      5385   27.54    0.00    0.00   27.54     0  stress
10:37:37 PM     0      5386   26.95    0.20    0.00   27.15     1  stress
10:37:37 PM     0      5387   27.54    0.00    0.00   27.54     0  stress
10:37:37 PM     0      5507    0.00    0.20    0.00    0.20     0  pidstat

Average:      UID       PID    %usr %system  %guest    %CPU   CPU  Command
Average:        0       963    1.00    0.40    0.00    1.40     -  Xorg
Average:     1000      1415    0.00    0.20    0.00    0.20     -  ibus-daemon
Average:     1000      1457    0.00    0.20    0.00    0.20     -  ibus-ui-gtk3
Average:     1000      1631    0.40    0.20    0.00    0.60     -  compiz
Average:     1000      2039    1.00    0.00    0.00    1.00     -  gnome-terminal-
Average:        0      5380   27.54    0.00    0.00   27.54     -  stress
Average:        0      5381   26.75    0.20    0.00   26.95     -  stress
Average:        0      5382   27.74    0.00    0.00   27.74     -  stress
Average:        0      5383   26.95    0.00    0.00   26.95     -  stress
Average:        0      5384   26.75    0.00    0.00   26.75     -  stress
Average:        0      5385   27.54    0.00    0.00   27.54     -  stress
Average:        0      5386   26.95    0.20    0.00   27.15     -  stress
Average:        0      5387   27.54    0.00    0.00   27.54     -  stress
Average:        0      5507    0.00    0.20    0.00    0.20     -  pidstat
```

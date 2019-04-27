---
title: Get Return Time Statistics With the Linux Time Command
date: 2019-02-11 15:37:22
categories:
- Linux
tags:
- Linux
---

## 转载：[Get Return Time Statistics With the Linux Time Command](https://www.lifewire.com/command-return-time-command-4054237)

The time command is one of the lesser known [Linux](https://www.lifewire.com/what-is-linux-2201940) commands but it can be used to show how long a command takes to run.



This is useful if you are a developer and you want to test the performance of your program or script.



This guide will list the main switches that you will use with the time command along with their meanings.

<!--more-->



### How to Use the Time Command

The syntax of the time command is as follows:



```
time
```



For example, you can run [the ls command](https://www.lifewire.com/uses-of-linux-ls-command-4054227) to list all the files in a folder in a long format along with the time command.



```
time ls -l
```



The results from the time command will be as follows:



```
real 0m0.177s
user 0m0.156s
sys 0m0.020s
```



The statistics shown show the total time is taken to run the command, the amount of time that was spent in [user mode](http://www.linfo.org/user_mode.html) and the amount of time spent in [kernel mode](http://www.linfo.org/kernel_mode.html).



If you have a program that you have written and you want to work on the performance you can run it along with the time command over and over and try and improve on the statistics.



By default, the output is displayed at the end of the program but perhaps you want the output to go to a file.



To output the format to a file use the following syntax:



```
time -o
time --output=
```



All of the switches for the time command must be specified before the command you wish to run.



If you are performance tuning then you may wish to append the output from the time command to the same file over and over so that you can see a trend.



To do so use the following syntax instead:



```
time -a
time --append
```







### Formatting the Output of the Time Command

By default the output is as follows:



```
real 0m0.177s
user 0m0.156s
sys 0m0.020s
```



There are a large number of formatting options as shown in the following list



- C - Name and command line arguments used
- D - Average size of the process's unshared data area in kilobytes
- E - Elapsed time in a clock format
- F - Number of page faults
- I - Number of file system inputs by the process
- K - Average total memory use of the process in kilobytes
- M - Maximum resident set size of the process during the lifetime in Kilobytes
- O - Number of file system outputs by the process
- P - Percentage of CPU that the job received
- R - Number of minor or recoverable page faults
- S - Total number of CPU seconds used by the system in kernel mode
- U - Total number of CPU seconds used by user mode
- W - Number of times the process was [swapped out of main memory](http://www.dictionary.com/browse/swap)
- X - Average amount of shared text in the process
- Z - System's page size in kilobytes
- c - Number of times the process was [context-switched](http://www.linfo.org/context_switch.html)
- e - Elapsed real time used by the process in seconds
- k - Number of signals delivered to the process
- p - Average unshared stack size of the process in kilobytes
- r - Number of socket messages received by the process



- s - Number of socket messages sent by the process
- t - Average resident set size of the process in [kilobytes](https://www.lifewire.com/network-data-rates-817365)
- w - Number of time the process was context-switched voluntarily
- x - Exit status of the command



You can use the formatting switches as follows:



```
time -f "Elapsed Time = %E, Inputs %I, Outputs %O"
```



The output for the above command would be something like this:



```
Elapsed Time = 0:01:00, Inputs 2, Outputs 1
```



You can mix and match the switches as required.



If you want to add a new line as part of the format string use the newline character as follows:



```
time -f "Elapsed Time = %E \n Inputs %I \n Outputs %O"
```







### Summary

To find out more about the time command read the [Linux](https://www.lifewire.com/create-a-linux-bootable-usb-drive-from-linux-4117072) Manual Page by running the following command:



```
man time
```



The format switch doesn't work straight away within Ubuntu. You need to run the command as follows:



```
/usr/bin/time
```
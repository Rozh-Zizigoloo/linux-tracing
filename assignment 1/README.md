# ASSIgn 1 : 🥇
In this part, it is supposed to use tracing tools such as lttng, perf; Let's trace the two ways of communicating and understand the difference between these two ways using analysis.

## Tools ⚙
- Ubuntu 20.04 focal
- LTTng V2.12
- Perf
- TraceCompass
- Netcat
- Telnet

## Netcat 🕹️
It functions as a **command-line utility** specifically designed for reading and writing data across network connections. It supports both **TCP** and **UDP** protocols, offering flexibility in communication methods.

### using for : 🪛
- Port scanning
- Port Listening
- File transfer
- Network debugging
- Simple proxies

 ### Let's do it 🪄
 First, we must create a session to record the communication between the **client** and the **server**.
We use **lttng** for this.
We have to open the session and write the things we want to be recorded and set a time and then stop the desired session.

for opening the session:
```
sudo lttng create $trace --output=/home/rozhina/$trace
sudo lttng enable-event -k --syscall --all
sudo lttng enable-event -k sched_switch,sched_wak'',irq_'',net_'',skb_''
 sudo lttng add-context -k -t vtid -t vpid -t procname -t prio
```
#### Help section ⚠️
**" sudo lttng enable-event -k sched_switch,sched_wak'',irq_'',net_'',skb_'' "**
This line further enables tracing of specific events related to scheduling, interrupts, network activity, and network packets.
The events include:
sched_switch: Context switches between processes.
sched_wak*: Events related to process wakeups.
irq_*: Events related to device interrupts.
net_*: Events related to network activity.
skb_*: Events related to network packets (SKBs - Socket Buffer).

**" sudo lttng add-context -k -t vtid -t vpid -t procname -t prio "**
This command adds context information to the trace data using sudo and lttng add-context.
The -k flag signifies adding kernel context.
The following options specify the context information to be included:
-t vtid: Thread ID (kernel-level)
-t vpid: Process ID
-t procname: Process name
-t prio: Process priority

After that :
```
sudo lttng start
```

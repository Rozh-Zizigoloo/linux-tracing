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
### Run NetCat :

run for server :
```
nc -l 127.0.0.1 4000
```

run for client:

```
nc -p 4444 127.0.0.1 4000
```
### result 🎥

![netcat](https://github.com/Rozh-Zizigoloo/linux-tracing/blob/main/assignment%201/SRC/Screenshot%202024-02-28%20095508.png)

then :

```
lttng stop
lttng destroy
```
### result 🎥
![TraceCompass_1](https://github.com/Rozh-Zizigoloo/linux-tracing/assets/156912661/754dc275-cae1-4210-ad6f-446c062f4066)

![TraceCompass_2](https://github.com/Rozh-Zizigoloo/linux-tracing/assets/156912661/248bea0d-923a-4328-b96a-0c7992179571)

### perf
for count systemcall and show runtime :

```
sudo perf stat -ae 'net:*,skb:*,irq:*,sched:sched_wak*,sched:sched_switch' -I 1000
```
without netcat perf file :
![perf1](https://github.com/Rozh-Zizigoloo/linux-tracing/blob/main/assignment%201/SRC/Screenshot%202024-02-28%20093526.png)

with netcat perf file :
![perf1](https://github.com/Rozh-Zizigoloo/linux-tracing/blob/main/assignment%201/SRC/Screenshot%202024-02-28%20093305.png)

### ftrace (other way) 🔧
For trace, we need a series of filters on kernel functions. Here we filter every function that has the names net, ip, tcp skb.
```
echo 'net*' 'ip*' 'tcp*' 'skb*' >> set_ftrace_filter
cat set_ftrace_filter
echo function_graph >> /sys/kernel/debug/tracing/current_tracer
```
Then,we run trace and get a simple netcat.
```
echo 1 > /sys/kernel/debug/tracing/tracing_on
Tab1 : $nc -l 127.0.0.1 4000
Tab2:$nc -p 4444 127.0.0.1 4000
echo 0 > /sys/kernel/debug/tracing/tracing_on
```
Finally, we save this trace in a file:

- [ftrace.log](https://github.com/Rozh-Zizigoloo/Linux-Kernel-Tracing-in-Docker-Network/blob/main/src/perf_simple.txt) :

![Picture1](https://github.com/Rozh-Zizigoloo/linux-tracing/blob/main/assignment%201/SRC/Screenshot%202024-02-28%20100706.png)

### Description
1. tcp_v4_do_rcv:

Function: This function is responsible for processing incoming TCP segments for IPv4 connections.
2. tcp_rcv_state_process:

Function: This function is a central point for handling incoming TCP segments based on the current state of the TCP connection.
3. tcp_rcv_synsent_state_process:

Function: This function specifically handles incoming segments for TCP connections that are in the SYN_SENT state (i.e., the connection is being initiated by the local socket).
4. ip_queue_xmit:

Function: This function is responsible for queuing packets (datagrams) for transmission on the network interface.
5. __netif_receive_skb, __netif_receive_skb_one_core, __netif_receive_skb_core:

Function: These functions are part of the network interface processing path and are responsible for handling received packets from the network. They work together to:
__netif_receive_skb: This function is the entry point for received packets. It performs initial processing and calls __netif_receive_skb_one_core for further handling.
__netif_receive_skb_one_core, __netif_receive_skb_core: These functions handle individual packets, performing tasks like:
Updating network device statistics.
Passing the packet up the protocol stack for further processing (e.g., to the IP layer for IPv4 packets).

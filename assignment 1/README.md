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

save :
```
sudo perf record -ae 'net:*,skb:*,irq:*,sched:sched_wak*,sched:sched_switch' -g
perf report > /home/rozhina/simple.perf
chown -R rozhina:rozhina /home/rozhina/simple.perf

```
![Picture1](https://github.com/Rozh-Zizigoloo/linux-tracing/blob/main/assignment%201/SRC/Screenshot%202024-02-28%20100706.png)

### ftrace (other way) 🔧
For trace, we need a series of filters on kernel functions. Here we filter every function that has the names net, ip, tcp skb.
```
echo 'net*' 'ip*' 'tcp*' 'skb*' 'sched*' >> set_ftrace_filter
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
# ASSIgn 2 : 🥇
Next, we utilize Telnet to establish a connection with www.google.com and execute an HTTP GET request in the subsequent manner.

First, we establish the Telnet with google :

```
telnet www.google.com 80
```

After our connection is successfully set, we send a simple GET message:

```
GET /search?q=hello+world
```

Simultaneously, we run Perf:

![Screenshot 2024-02-28 104500](https://github.com/Rozh-Zizigoloo/linux-tracing/assets/156912661/af86b4cc-b783-4148-b984-12a21d0db4f4)

## difference 1,2:
Now let's compare the Perf logs in Experiment 1 & 2.

We could easily see that:

**netif_rx** is present in netcat log and absent in Telnet log.
*napi_gro_receive_entry is present in Telnet log and absent in netcat log.


In the context of the Linux kernel, netif_rx is a function responsible for receiving and processing network packets from network devices (interfaces) like Ethernet cards or Wi-Fi adapters. It acts as a central entry point for incoming network traffic.

Here's a detailed breakdown of its functionality:

**Receiving Packets:**
Network devices capture incoming raw data frames from the physical network (e.g., Ethernet cable).
These frames are converted into kernel data structures called sk_buff (sk_buffer).
The network device driver then invokes netif_rx and passes the sk_buff containing the packet data.

**napi_gro_receive_entry** is a tracepoint defined in the Linux kernel related to the NAPI (Network Application Programming Interface) and GRO (Generic Receive Offload) functionalities.

Data Captured by napi_gro_receive_entry (may vary depending on kernel version):

skb: Pointer to the sk_buff (kernel data structure) containing the received packet.
dev: Pointer to the network device that received the packet.
len: Length of the received packet.
flags: Flags associated with the received packet.
gro_cookie: Cookie value used for GRO operations.
queue: Queue number on the network device where the packet was received.

# ASSIgn 3 : 🥇

This time, we redo the previous experiments, but this time, we use pinging.

## ping

First, we specify a custom packet size of 25000 bytes and we ping the default gateway:

```
ping -s 65507 10.0.2.2
```

![Ping_Fragment](https://github.com/Rozh-Zizigoloo/linux-tracing/assets/156912661/c4243771-cba5-4fd2-8a15-168b06af1c5b)

 The trace would show individual ping packets with a larger size (64KB) compared to the standard ping packet size (typically around 56 bytes).
 
Looking at Soft_irq Network RX in CPU5, it is obvious that fragmentation has occurred. The packet length in transmit events is 65507bytes; which is due to MTU (14 bytes for ethernet header).

**"Soft_irq Network RX"** is a term used in the Linux kernel to describe a specific type of software interrupt related to network packet reception.

Here's a breakdown of the components:

Soft IRQ: An asynchronous interrupt handled in software context (outside the real-time interrupt handlers). This allows for more efficient processing of tasks that don't require immediate attention, like network packet processing.
Network RX: Refers to receiving network packets from the network interface card (NIC) into the operating system.

## flood ping

```
sudo ping -f 10.0.2.2
```
![Screenshot 2024-02-28 112710](https://github.com/Rozh-Zizigoloo/linux-tracing/assets/156912661/dc50fa19-7656-4510-9f3d-79e2ca3b0c1b)

We can conclude that the result is the same as Telnet experiment. Because default gateway engages with our Network Adapter.
**BUT**
Ping: Only verifies basic reachability (whether the device is "alive").
Telnet: Potentially allows limited interaction and access to services on the target device (if Telnet is enabled and secure measures are not in place).

## References

- [Deep Linux](https://www.youtube.com/@deeplinux2248)
- [The Linux Kernel documentation](https://docs.kernel.org/)

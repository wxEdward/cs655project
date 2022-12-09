# cs655project


This is the code execution guide for CS655 project. Please copy the code to respected terminal step by step.

**Setup:**

R1:

sudo apt-get update  
sudo apt-get -y install iperf3 
sudo apt-get -y install moreutils r-base-core r-cran-ggplot2 r-cran-littler
sudo sysctl -w net.ipv4.tcp_no_metrics_save=1

J1:

sudo apt-get update  
sudo apt-get -y install iperf3

D1:

sudo tc qdisc del dev eth1 root
sudo tc qdisc add dev eth1 root handle 1: htb default 3
sudo tc class add dev eth1 parent 1: classid 1:3 htb rate 1Mbit
sudo tc qdisc add dev eth1 parent 1:3 handle 3: bfifo limit 0.1MB
sudo tc qdisc del dev eth2 root
sudo tc qdisc add dev eth2 root handle 1: htb default 3
sudo tc class add dev eth2 parent 1: classid 1:3 htb rate 1Mbit
sudo tc qdisc add dev eth2 parent 1:3 handle 3: bfifo limit 0.1MB

R1:

wget -O ss-output.sh https://raw.githubusercontent.com/ffund/tcp-ip-essentials/gh-pages/scripts/ss-output.sh

**Exp 1**:

J1: 

iperf3 -s -1

R1:

bash ss-output.sh 10.10.2.100  

R2:

iperf3 -c juliet -P 3 -t 60 -C reno 

**Exp 2:**

J1: 

iperf3 -s -1

R1:

bash ss-output.sh 10.10.2.100  

R2:

iperf3 -c juliet -P 3 -t 60 -C cubic

**Exp 3.1:**

J1: 

iperf3 -s -1

R1:

iperf3 -c juliet -t 60 -C reno

R2:

ping juliet -c 50 

(record the results)

R1:

sudo modprobe tcp_vegas  
sudo iperf3 -c juliet -t 60 -C vegas

R2:

ping juliet -c 50

(record the results)

**Exp 3.2:**

J1: 

iperf3 -s -1

J2:

iperf3 -s -1 -p 5301

R1:

sudo iperf3 -c juliet -t 60 -C vegas 

R2:

iperf3 -c juliet -t 60 -C reno -p 5301

(record the results)

**Exp 4:** 

R1:

sudo modprobe tcp_bbr

J1: 

iperf3 -s -1

R1:

bash ss-output.sh 10.10.2.100  

R2:

iperf3 -c juliet -P 3 -t 60 -C bbr

**Exp 5:**

D1: 

sudo tc qdisc del dev eth1 root  
sudo tc qdisc add dev eth1 root handle 1: htb default 3  
sudo tc class add dev eth1 parent 1: classid 1:3 htb rate 1Mbit  
sudo tc qdisc add dev eth1 parent 1:3 handle 3:  codel limit 100 target 10ms ecn

sudo tc qdisc del dev eth2 root  
sudo tc qdisc add dev eth2 root handle 1: htb default 3  
sudo tc class add dev eth2 parent 1: classid 1:3 htb rate 1Mbit  
sudo tc qdisc add dev eth2 parent 1:3 handle 3:  codel limit 100 target 10ms ecn

R1:

sudo sysctl -w net.ipv4.tcp_ecn=1 

J1:

sudo sysctl -w net.ipv4.tcp_ecn=1 

R1:

sudo tcpdump -s 80 -i eth1 'tcp' -w $(hostname -s)-tcp-ecn.pcap  

J1:

sudo tcpdump -s 80 -i eth1 'tcp' -w $(hostname -s)-tcp-ecn.pcap

J2:

iperf3 -s -1 

R2:

iperf3 -c juliet -t 60 -C reno  

R3:

ping -c 50 juliet

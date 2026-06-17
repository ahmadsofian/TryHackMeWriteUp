### Task 1: Introduction
SNORT is an open-source, rule-based Network Intrusion Detection and Prevention System (NIDS/NIPS). It was developed and still maintained by Martin Roesch, open-source contributors, and the Cisco Talos team.  The official description: "Snort is the foremost Open Source Intrusion Prevention System (IPS) in the world. Snort IPS uses a series of rules that helps define malicious network activity and uses those rules to find packets that match against them and generate alerts for users.

### Task 2: Interactive Material and VM
There are two sub-folders available:
* Config-Sample: Sample configuration and rule files. These files are provided to show what the configuration files look like. Installed Snort instance doesn't use them, so feel free to practice and modify them. Snort's original base files are located under /etc/snort folder.
* Exercise-Files: There are separate folders for each task. Each folder contains pcap, log, and rule files ready to play with.

**Traffic Generator**\
Run the "traffic generator.sh" file by executing it as sudo. 
*user@ubuntu$ sudo ./traffic-generator.sh*

### Task 3: Introduction to IDS/IPS
#### Intrusion Detection System (IDS)
IDS is a passive monitoring solution for detecting possible malicious activities/patterns, abnormal incidents, and policy violations. It generates alerts for each suspicious event. 
There are two main types of IDS systems:
* **Network Intrusion Detection System (NIDS):** NIDS monitors the traffic flow from various areas of the network. The aim is to investigate the traffic on the entire subnet. If a signature is identified, an alert is created.
* **Host-based Intrusion Detection System (HIDS):**  HIDS monitors the traffic flow from a single endpoint device. Its aim is to investigate the traffic on that device. If a signature is identified, an alert is created.

#### Intrusion Prevention System (IPS)
IPS is an active protecting solution for preventing possible malicious activities/patterns, abnormal incidents, and policy violations. It is responsible for stopping/preventing/terminating the suspicious event as soon as it is detected.
There are four main types of IPS systems;
* **Network Intrusion Prevention System (NIPS):** NIPS monitors the traffic flow from various areas of the network. The aim is to protect the traffic on the entire subnet. If a signature is identified, the connection is terminated.
* **Behaviour-based Intrusion Prevention System (Network Behaviour Analysis - NBA):** Behaviour-based systems monitor the traffic flow from various areas of the network. The aim is to protect the traffic on the entire subnet. If an anomaly is identified, the connection is terminated.

#### Detection and Prevention Techniques
There are three main detection and prevention techniques used in IDS and IPS solutions;
<img width="753" height="369" alt="image" src="https://github.com/user-attachments/assets/b821e8e4-3219-4ade-85fb-d400ab68ffd4" />

### Task 4: First Interaction with Snort
1. Verify snort using: *snort -V*
2. Test the configuration and identify configuration file using: *sudo snort -c /etc/snort/snort.conf -T*
    
### Task 5: Operation Mode 1: Sniffer Mode
* **Sniffing with parameter "-i"**: sudo snort -v -i eth0
* **Sniffing with parameter "-v"**: sudo snort -v
* **Sniffing with parameter "-d"**: sudo snort -d
* **Sniffing with parameter "-de"**: snort -d -e
* **Sniffing with parameter "-X"**: sudo snort -X

### Task 6: Operation Mode 2: Packet Logger Mode
* **Logfile Ownership**: sudo chown username file / sudo chown username -R directory
* **Logging with parameter "-l"**: sudo snort -dev -l
* **Logging with parameter "-K ASCII"**: sudo snort -dev -K ASCII
* **Reading generted logs with parameter "-r":**: sudo snort -r

### Task 7: Operation Mode 3: IDS/IPS
* **IDS/IPS mode with parameter "-c and -T"**: sudo snort -c /etc/snort/snort.conf -T
* **IDS/IPS mode with parameter "-N"**: sudo snort -c /etc/snort/snort.conf -N
* **IDS/IPS mode with parameter "-D"**: sudo snort -c /etc/snort/snort.conf -D
* **IDS/IPS mode with parameter "-A console"**: sudo snort -c /etc/snort/snort.conf -A console
* **IDS/IPS mode with parameter "-A cmg"**: sudo snort -c /etc/snort/snort.conf -A cmg
* **IDS/IPS mode with parameter "-A fast"**: sudo snort -c /etc/snort/snort.conf -A fast
* **IDS/IPS mode with parameter "-A full"**: sudo snort -c /etc/snort/snort.conf -A full
* **IDS/IPS mode with parameter "-A none"**: sudo snort -c /etc/snort/snort.conf - A none
* **IDS/IPS mode "Using rule file without configuration file"**: sudo snort -c /etc/snort/rules/local.rules -A console
* **IPS mode and dropping packets**: sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1


### Task 8: Operation Mode 4: PCAP Investigation
* **Investigating single PCAP with parameter "-r"**: sudo snort -c /etc/snort/snort.conf -q -r icmp-test.pcap -A console -n 10
* **Investigating multiple PCAPs with parameter "--pcap-list"**: sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console -n 10
* **Investigating multiple PCAPs with parameter "--pcap-show"**: sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console --pcap-show

### Task 9: Snort Rule Structure
The primary structure of the Snort rule is shown below;
<img width="1140" height="784" alt="image" src="https://github.com/user-attachments/assets/2b6d9a20-3063-4043-bc9b-fd09c945a917" /> \


### Task 10: Snort2 Operation Logic: Points to Remember
**Main Component of Snort**:
* Packet Decoder - Packet collector component of Snort. It collects and prepares the packets for pre-processing.
* Pre-processors - A component that arranges and modifies the packets for the detection engine.
* Detection Engine - The primary component that processes, dissects, and analyzes the packets by applying the rules.
* Logging and Alerting - Log and alert generation component.
* Outputs and Plugins - Output integration modules (i.e., alerts to syslog/mysql) and additional plugins (rule management detection plugins) support is done with this component. 

**Snort Rules**:
* Community Rules - Free ruleset under the GPLv2. Publicly accessible, no need for registration.
* Registered Rules - Free ruleset (requires registration). This ruleset contains subscriber rules with 30 30-day delay.
* Subscriber Rules (Paid) - Paid ruleset (requires subscription). This ruleset is the main ruleset and is updated twice a week (Tuesdays and Thursdays).

  

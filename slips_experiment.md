## 所涉及实验模块

|  模块   | 内容  |注释与配置地址  |
|  ----  | ----  |  ----  |
| **1.ARP** | module to check for ARP attacks in ARP traffic|  |
| 2.blocking| module to block malicious IPs connecting to the device |start slips with the -p flag.|
| 3.CESNET|Send and receive alerts from warden servers  |  |
| 4.ensembling|  |  |
| 5.ExportingAlerts| module to export alerts to slack, STIX or **suricata-like JSON** format |  |
| 6.flowalerts|module to find malicious behaviour in each flow. Current measures are: long duration of the connection, successful ssh  |  |
| **7.flowmldetection**| module to detect malicious flows using ML pretrained models |mode = train  # You should have trained at least once with 'Normal' data and once with 'Malicious' data in order for the test to work.|
| 8.http_analyzer|module to analyze HTTP traffic  |  |
| 9.IP_Info|module to find Geolocation, ASN, RDNS info about IPs and MAC vendors  |  |
| 10.kalipso| 可视化界面模块 | npm install chalk@4.1.2 |
| **11.leak_detector**| module to detect leaks of data in the traffic using YARA rules |目前包含yara rules：gps_location_leaked yara规则存储于modules/leak_detector/yara_rules/rules并会自动编译|
|**12.portscanDetector-1**| detects Horizontal and Vertical port scans |  |
| 13.RiskIQ  |This module is used to get different information (passive DNS, IoCs, etc.)   |  modules/RiskIQ/credentials |
| **14.rnn-cc-detection-1**|detects command and control channels using recurrent neural network and the stratosphere behavioral letters  |  |
| 15.template|  |  |
|**16.ThreatIntelligence1**|checks if each IP is in a list of malicious IPs  | 将网络上一些汇总的异常地址集中到modules/ThreatIntelligence1/remote_data_files中 |
| **17.timeline**| creates a timeline of what happened in the network based on all the flows and type of data available |  |
| 18.UpdateManager| Takes care of downloading each of the files used by Slips, but only if there is a need to update them. It stores and checks the ETags of remote files to know if they changed. It can be configured to update each file with a different frequency. Most importantly it updates all the Threat Intelligence feeds. | malicious_data_update_period key in slips.conf |
| 19.virustotal| module to lookup IP address on VirusTotal |modules/virustotal/api_key_secret |  |

这些模块可以在slips.conf当中被添加至忽略
template，   ensembling,   threatintelligence1（16）, blocking（2）,     portscanDetector-1（12）,    timeline（17）,     virustotal（19）,     rnn-cc-detection-1（14）,    flowmldetection（7）

# Slips实验
## 安装
docker当中没有安装  apt update && apt install -y --no-install-recommends python3-tzlocal zeek
跳过
RUN npm set registry https://registry.npm.taobao.org/
RUN npm install blessed blessed-contrib redis@3.1.2 async chalk@4.1.2 strip-ansi@6.0.0 clipboardy fs sorted-array-async yargs

## 打开
docker run -it --rm --net=host --cap-add=NET_ADMIN stratosphereips/slips:latest

./slips.py -c slips.conf -i eno1 -p

（**以下是非capture模式，只能读文件**）
docker run -it --rm --net=host slips

./slips.py -c slips.conf -f dataset/test3.binetflow
## 开启redis服务：
./redis-server /media/wuguo-buaa/LENOVO_USB_HDD/redis-3.2.8/redis.conf
###修改max_user_watches
https://www.cnblogs.com/jincon/p/3702545.html

**在192.168.0.100上开启diagslave作为从站，在192.168.1.133上开启modpoll作为主站**

## 1、正常流量和异常流量存入
label =

normal正常流量：modpoll对于100-104五个线圈的各种读操作以及对101线圈的写操作

malicious异常流量：modpoll对于100-104线圈以外的写操作

unknown未知流量：用于test

**wireshark抓包可能导致所有经过接口的包都被捕获，需要使用tcpdump：**

sudo tcpdump -i enp2s0 host 192.168.0.100 -w /media/wuguo-buaa/LENOVO_USB_HDD/Exp_data/modbus_test.pcap

**若实验有误，清空数据库**

进入./redis-cli			select 1			flushall

清华源下载tensorflow-gpu 2.4.1

**禁用全部模块**

disable = [template , ensembling , RiskIQ , blocking , http_analyzer , ThreatIntelligence1 , IP_Info , ExportingAlerts , UpdateManager , flowalerts , CESNET , virustotal , ARP , flowmldetection , leak_detector , portscanDetector , rnn-cc-detection-1 , timeline]

**禁用除ML外全部模块 disable all except ML**

disable = [template , ensembling , RiskIQ , blocking , http_analyzer , ThreatIntelligence1 , IP_Info , ExportingAlerts , UpdateManager , flowalerts , CESNET , virustotal , ARP , leak_detector , portscanDetector , rnn-cc-detection-1 , timeline]

**禁用大多数模块disable unmarked**

disable = [template , ensembling , RiskIQ , blocking , http_analyzer , ThreatIntelligence1 , IP_Info , ExportingAlerts , UpdateManager , flowalerts , CESNET , virustotal]

## 打开界面
./kalipso.sh

## modbus发包
./i686-linux-gnu/modpoll -r 101 -c 1 192.168.0.100 12

## 1)   train_normal
**deletePrevdb = True**覆盖原来的数据库

(venv) wuguo-buaa@wuguo-buaa:/media/wuguo-buaa/LENOVO_USB_HDD/PycharmProjects/StratosphereLinuxIPS$ ./slips.py -c slips.conf -f /media/wuguo-buaa/LENOVO_USB_HDD/Exp_data/modbus_normal.pcap
## 2)   train_malicious
将label =参数修改为malicious

**deletePrevdb = False**沿用之前的数据库

(venv) wuguo-buaa@wuguo-buaa:/media/wuguo-buaa/LENOVO_USB_HDD/PycharmProjects/StratosphereLinuxIPS$ ./slips.py -c slips.conf -f /media/wuguo-buaa/LENOVO_USB_HDD/Exp_data/modbus_malicious.pcap
## 3）test_unknown
将label =参数修改为unknown

**deletePrevdb = False**沿用之前的数据库

(venv) wuguo-buaa@wuguo-buaa:/media/wuguo-buaa/LENOVO_USB_HDD/PycharmProjects/StratosphereLinuxIPS$ ./slips.py -c slips.conf -f /media/wuguo-buaa/LENOVO_USB_HDD/Exp_data/modbus_test.pcap

## 数据备份
redis-cli: BGSAVE

重命名后放置exp_data中，完成后再flushall以及redis-cli shutdown

重新启用时，复制到redis_database中并改名为dump.rbd

最后再打开redis

## in 工控机
### redis
安装在/usr/local/bin/下
配置conf文件地址：
/usr/local/bin/redis-server /etc/redis/redis.conf

### zeek
#### 安装
      ./configure && make && sudo make install
#### 修改软连接路径
      sudo ln --symbolic /usr/local/zeek/bin/zeek /usr/bin/zeek
#### 添加环境
      export PATH=$PATH:/usr/local/zeek/bin
      source ~/.bashrc

## 如何在工控机开启slips
工控机cpu版本较低，不支持avx指令

      ubuntu tensorflow非法指令(核心已转储)
需要把rnn模块置于禁用模块状态
首先开启./home/wuguo-buaa/PycharmProjects/StratosphereLinuxIPS/kalipso.sh
网卡监听需要使用sudo权限
sudo /home/wuguo-buaa/anaconda3/envs/slips/bin/python3.8 /home/wuguo-buaa/PycharmProjects/StratosphereLinuxIPS/slips.sh
### 全部模块
template , ensembling , RiskIQ , blocking , http_analyzer , ThreatIntelligence1 , IP_Info , ExportingAlerts , UpdateManager , flowalerts , CESNET , virustotal , ARP , flowmldetection , leak_detector , portscanDetector , rnn-cc-detection-1 , timeline
**删除模块**
template , ensembling , RiskIQ , blocking , http_analyzer , ThreatIntelligence1 , IP_Info , ExportingAlerts , UpdateManager  , CESNET , virustotal  ,  rnn-cc-detection-1
**保留模块**
leak_detector , portscanDetector , flowalerts , ARP , flowmldetection , timeline

### 流程
slips.py加载初始参数（数据库、检测源等）

根据用户指定的生效模块来进行功能模块的初始化，采用多线程机制来开启模块，模块继承自slips_files/common/abstracts.py中的Moudle抽象类以及多线程的基类（多继承）

在ml方法中，首先加载之前训练过的模型
从数据库中订阅new_flow的数据，在死循环当中使用get_message方法获取数据，进行数据核验。

 flow = json.loads(flow)
 把每一个新的flow内容加载到 self.flow_dict中
**训练阶段**
 每50个数据进行一次重新训练
 将全部数据转为pandas dataframe
 模型使用SGD分类器，借助其自身可以在线增量学习且资源占用小的优势部署在边缘端内部
 self.clf = SGDClassifier(warm_start=True, loss='hinge', penalty="l1")
**验证阶段（实现模型阶段）**
预测阶段将单一一个flow作为dataframe
在flow的label属性当中就会存入预测的结果
if pred[0] == 'Malware'
set_evidence_malicious_flow
把相应的参数组加入到证据中

根据证据来确定是否启用Blocking模块，在iptables中添加该语句

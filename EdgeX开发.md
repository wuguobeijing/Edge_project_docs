#EdgeX开发过程
##可能用得到的工具
[EdgeX-compose 文件地址](https://github.com/edgexfoundry/edgex-compose)
[EdgeX-compose 文件地址](https://github.com/edgexfoundry/edgex-compose)
##框架
![](assets/EdgeX开发-1b848da8.png)
### Core Services Layer

#### 核心服务（CS）层将边缘处的北侧和南侧层分开。 核心服务包括以下组件：

    核心数据：从南侧对象收集的数据的持久性存储库和相关的管理服务。

    命令：处理北向应用发往南向设备的请求。

    元数据：有关连接到EdgeX Foundry的对象的元数据的存储库和关联管理服务。 提供配置新设备并将其与其拥有的设备服务配对的功能。

    注册与配置：为其他EdgeX Foundry微服务提供有关EdgeX Foundry和微服务配置属性（即初始化值存储库）中相关服务的信息。
#### 此时EdgeX Foundry核心服务层包括以下微服务：

        Architecture--Core Services--Configuration and Registry
        Architecture--Core Services--Core Data
        Architecture--Core Services--Metadata
        Architecture--Core Services--Command

##### Configuration and Registry：
        作为配置管理器，配置和注册表微服务在微服务启动时为每个微服务提供配置信息。此配置信息会覆盖微服务可能具有的任何内置配置，并提供满足微服务架构动态特性的方法。例如，配置和注册表微服务提供的配置信息可能会为另一个EdgeX Foundry微服务提供一个新的操作端口号，其中默认端口号已被运行EdgeX Foundry的系统使用。配置和注册表微服务还提供了向EdgeX Foundry微服务通知配置更改的方法。这允许其他微服务动态地响应环境的变化。请注意，虽然配置和注册表微服务可以通知微服务任何配置更改，但微服务必须注册此更改并提供响应通知的工具。

        作为EdgeX Foundry微服务注册表，配置和注册表微服务知道所有EdgeX Foundry微服务的位置和运行状态。当每个EdgeX Foundry微服务启动时，它被要求使用配置和注册表微服务注册自己。然后，配置和注册表微服务定期“ping”其他微服务，以保持微服务集合的健康状况的准确图像。这为其他EdgeX Foundry微服务，系统管理系统和第三方应用程序提供了一个权威的位置，以获得EdgeX Foundry状态。

        EdgeX Foundry微服务可以在没有配置和注册表微服务的情况下运行。当他们这样做时，他们使用内置配置初始化/配置自己并在本地而不是全局操作 - 也就是说，他们不会在任何中央机构或其他微服务中注册它们。如果没有配置和注册表微服务，每个其他微服务只能对位置（通过其自身的本地初始化提供）和其他微服务的运行状态做出假设。

##### Core Data：
        核心数据微服务为设备和传感器收集的数据读数提供集中的持久性工具。用于收集数据的设备和传感器的设备服务，调用Core Data服务以将设备和传感器数据存储在边缘系统（例如网关）中，直到数据可以“向北”移动，然后导出到企业和云系统。

        EdgeX Foundry内部以及可能位于EdgeX Foundry外部的其他服务（如计划服务）仅通过Core Data服务访问存储在网关上的设备和传感器数据。核心数据在数据处于边缘时为设备和传感器收集的数据提供一定程度的安全性和保护。

        Core Data使用REST API将数据移入和移出本地存储。将来，微服务可以扩展，以允许通过其他协议（如MQTT，AMQP等）访问数据。默认情况下，Core Data通过ZeroMQ将数据移动到Export Service层。 Core Data微服务的备用配置允许数据通过MQTT分发到Export Services，但也需要安装ActiveMQ等代理。

        规则引擎微服务默认从Export Distribution微服务接收其数据。在关注延迟或容量的情况下，规则引擎微服务的备用配置允许它还通过ZeroMQ直接从Core Data获取其数据（它成为同一Export Services ZeroMQ分发通道的第二个订户）。

        核心数据“流媒体” 默认情况下，Core Data会保留发送给它的设备和传感器收集的所有数据。但是，当数据过于敏感而无法存储在边缘时，或者本地其他服务（例如，通过分析微服务）不需要边缘数据时，数据可以通过Core Data“流式传输”而不会保留。 对Core Data（persist.data = false）的配置更改使Core Data通过消息队列将数据发送到Export Service，而不在本地持久保存数据。此操作具有减少时延和存储需求的优点，但边缘的代价是没有历史数据可用于基于随时间变化的操作，并且只有基于单个事件数据的最小设备驱动决策。
![](assets/EdgeX开发-17c76f7a.png)

##### Metadata：
        Metadata微服务具有关于设备和传感器的知识以及如何与其他服务（例如核心数据，命令等）使用它们进行通信。

        具体而言，Metadata具有以下能力：

            管理有关连接到EdgeX Foundry并由其运营的设备和传感器的信息
            了解设备和传感器报告的数据类型和组织
            知道如何命令设备和传感器

        Metadata不执行以下活动：

            不执行，也不对设备和传感器的实际数据收集负责，这些数据由设备服务和核心数据执行
            不执行，也不负责向设备和传感器发出命令，这些命令由命令和设备服务执行

        在Metadata微服务层中，主要存储三类主要信息：（1）设备配置文件；（2）具体设备的物理信息（如设备ID、设备地址等）；（3）设备服务的信息。

        （1）设备配置文件。EdgeX Foundry中的设备配置文件显示了设备的一般特性，这些配置文件提供设备基本数据信息以及如何命令设备。 设备配置文件可以被认为是设备类型或分类的模板。 例如，BACnet恒温器的设备配置文件为BACnet恒温器发送的数据类型提供了一般特性，例如当前温度，以及可将哪些类型的命令或操作发送到BACnet恒温器，如冷却设定点或加热设定点。 因此，设备配置文件是元数据服务必须能够以本地持久性存储或管理的首要项目，并提供给EdgeX Foundry的其他服务。

        （2）具体设备的物理信息。有关实际设备和传感器的数据是元数据微服务存储和管理的另一类信息。 由EdgeX Foundry管理的每个特定设备和传感器都必须在元数据中注册并具有与其关联的唯一ID。信息（如设备或传感器的地址）与该标识符一起存储。每个设备和传感器也与设备配置文件相关联。该关联使元数据能够将设备配置文件提供的通用信息应用于每个设备和传感器。例如，位于戴尔建筑的CTO解决方案实验室中的特定设备（如BACNet恒温器）与上述BACnet恒温器设备配置文件相关联，此连接意味着此特定BACnet恒温器提供当前温度数据并响应命令，即设置冷却点或加热点。

        （3）设备服务的信息。元数据存储和管理设备服务的信息。单一设备服务便于EdgeX Foundry与一个或多个实际设备或传感器之间的通信。通常，设备服务被构建用于通过特定协议与使用该协议的设备和传感器进行通信。例如，一个Modbus设备服务，便于各种类型的Modbus设备（如电机控制器，恒温器，功率计等等）之间的通信。

        Matadata必须知道每个设备服务，其地址以及如何与之通信，还必须跟踪每个设备与设备服务的关联（所有权）。有了这些信息，当任何其他EdgeX Foundry服务需要特定设备或传感器的请求时，Metadata会向EdgeX Foundry的其余部分提供有关哪些特定设备服务进行通信的知识。Metadata还需要了解设备服务，以便在添加，更新或删除新设备或传感器时，Metadata可以将设备或传感器信息的更改传达给相应的设备服务。

        Metadata作为EdgeX Foundry其余所有设备，设备配置文件和设备服务信息的单一访问点。通过这种方式，Metadata为EdgeX Foundry收集的“Meta”信息提供了一定程度的安全性和保护。

        目前，Metadata提供了一个REST API，用于将数据输入和输出EdgeX Foundry的本地存储。将来，微服务可以扩展，以允许通过其他协议（如MQTT，AMQP等）访问数据。
### Device Services Layer
        设备服务（DS）是与设备或物联网对象（“东西”）交互的边缘连接器，包括但不限于：家庭和办公楼中的报警系统，供暖和空调系统，灯，工业中的任何机器，灌溉系统，无人机，目前的自动化运输，如一些铁路系统，目前的自动化工厂，以及家中的电器。未来，这可能包括无人驾驶汽车和卡车，交通信号灯，全自动快餐设施，全自动自助杂货店，从患者那里获取医疗读数的设备等。

        DS可以一次为一个或多个设备（传感器，执行器等）提供服务。DS管理的“设备”不仅可以是简单的单个物理设备，也可以是另一个网关（以及所有该网关的设备）;设备管理员;或者设备聚合器;或设备的集合。

        DS层的微服务通过每个IoT对象的本地协议与设备，传感器，执行器和其他物联网对象进行通信。DS层将IoT对象生成和传递的数据转换为通用EdgeX Foundry数据结构，并将转换后的数据发送到Core Services层，以及EdgeX Foundry其他层中的其他微服务。

        EdgeX Foundry提供了一个设备服务软件开发工具包（SDK），用于生成设备服务的shell。它使新设备服务的创建更容易，并为核心服务层提供连接代码
  #### 包含Architecture--Device Services--Virtual Device
  Examples of Device Services

1.BACNet DS将BACNet设备提供的温度和湿度读数转换为通用的EdgeX Foundry对象数据结构。
2.DS接收并转换来自其他EdgeX Foundry服务或企业系统的命令，并将这些请求传递给设备，以便以设备理解的编程语言进行激活。
3.DS可能会收到关闭Modbus PLC控制电机的请求。DS会将通用EdgeX Foundry“关闭”请求转换为Modbus串行命令，PLC控制电机可以通过该命令进行驱动。


##开发测试流程
在/home/wuguo-buaa/edgex-compose当中下载comose.yml
#### 修改其中：
services:
  app-service-rules、app-service-sample、command、consul、data（有两处）、database、device-rest、device-virtual、metadata、notifications、rulesengine、scheduler、system
  共14处的127.0.0.0被修改为0。0。0。0
  ![](assets/EdgeX开发-0c2f0ac6.png)
  ### Modbus自动更新数据功能实现
写了两个文件：
**robot.profile.yml**
**device.config.toml**
从git上克隆edgex-compose到edgex-compose文件夹，把包含上述两个设备相关文件的文件夹custom-config添加到compose-builder目录中

    make gen ds-modbus
![](assets/EdgeX开发-be7964c2.png)
在资源下的docker-compose.yml中加入device-modbus微服务的相关配置地址
![](assets/EdgeX开发-c3fc8104.png)
![](assets/EdgeX开发-024ba362.png)
**上传device profile到metadata**
![](assets/EdgeX开发-2665b585.png)
**上传设备基本配置到metadata**
![](assets/EdgeX开发-29b2b3a4.png)
**set方法测试到command**
![](assets/EdgeX开发-8c514f60.png)
**set方法测试到command**
![](assets/EdgeX开发-3af408ad.png)

(把volume地址改为了edgex_compose_Modbus尚未测试)
### MQTT通讯
完全按照EdgeX官网测试还是会有restarting问题
相关链接：
#### 结构
![](assets/EdgeX开发-8a622d8b.png)
#### 修改配置文件
将geneva的composefile——docker-compose-geneva-redis-no-secty.yml修改为docker-compose.yml
向其中添加mqtt与ui的服务，不做内容修改
**打开4000端口的UI录入设备**
![](assets/EdgeX开发-6aaaadbf.png)
**打开EMQX broker的docker**
![](assets/EdgeX开发-15603a9c.png)
#### MQTT client
mock-device-driver
模拟一个基于mqtt协议通信的虚拟device，用于演示EdgeX Foundry的device-mqtt设备微服务的通信演示。两个独立线程，主线程负责接收处理命令，子线程负责模拟device的主动发送数据能力。
假设这个python脚本就是边缘物理设备上的驱动，用于监听命令，响应命令，且具备主动发送数据的能力。

**打开监听程序并进行功能测试**
包括：
testping

testrandnum

testmessage

#### 过程
从command微服务发送一个命令出来，转发到了device_service微服务，也就是device-mqtt微服务
device-mqtt发送到注册broker

### Kuiper与EdgeX Foundry集成
[Kuiper与EdgeX Foundry集成实践](https://www.jianshu.com/p/0726d41b00bf)

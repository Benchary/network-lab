家里或小工作室，一共十几个人要上网，不想花太多钱买设备，如何搞定？

用 GNS3 模拟器，拉一台 Cisco 路由器，配置 NAT 和 DHCP，二十分钟就能跑通。  
这篇文章就是完整可复现的方案——适合家庭和小型组织。

# 1.说明

本网络架构面向 **0–50 人** 的家庭或小型组织（办公室、门店、工作室等），设计简单，易于维护，成本低廉。

使用 **Cisco 设备** + **GNS3 模拟器** 完成全部配置，所有步骤均可直接迁移到真实硬件上。

# 2.需求与方案对比

|项目|说明|
| :---------| ----------------------------------------------------|
|规模|0–50 人|
|复杂度|✅ 简单|
|优点|架构清晰，维护容易，经济高效|
|劣势|单链路、单路由器，无冗余，不保证业务连续性|
|业务价值|具备基本可用性，可快速接入互联网，满足日常上网需求|

> [!TIP]
> 这是一个**高性价比、能直接干活**的方案，但不是高可用方案。

# 3.拓扑图

![CISCO-NAT-Internet](assets/CISCO-NAT-Internet-20260531221320-st06fet.png)

# 4.模拟器与真实环境对照表

|模拟器|真实环境|
| :-----------| --------------|
|VMnet8|互联网(外网)|
|R1|运营商侧设备|
|R2|用户侧设备|
|TinyCore-1|电脑终端|

> 实际环境中，R2 可以是家用无线路由器或 企业级出口路由器、防火墙。

# 5.部署过程

5.1 **模拟器部署过程（完整可复现）**

- 添加Cloud1（ **VMnet8 云**作为互联网出口）
- 添加两台 Cisco 路由器（R1 模拟运营商，R2 模拟用户路由器）
- 添加一台 TinyCore 虚拟机
- 按拓扑图连线

5.2 **配置 R1（模拟运营商侧）**

R1 的作用仅仅是让内网能“看到”互联网。

```c#
R1#configure
Configuring from terminal, memory, or network [terminal]?
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface fastEthernet 0/0
R1(config-if)#ip address 192.168.157.135 255.255.255.0 #和VMnet8必须同一子网
R1(config-if)#exit
R1(config)#interface fastEthernet 0/1
R1(config-if)#ip address 223.0.113.1 255.255.255.0
R1(config-if)#exit
R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.157.2 #指向VMnet8网关
```

> 🧠 **经验提醒**：真实环境中 R1 并不归你管，但模拟时把这条默认路由配上，内网流量才能出去。

5.3 **配置 R2（核心：用户侧路由器）**

R2 承担 **NAT + DHCP + 默认路由** 三大任务。

```c#
R1#configure
Configuring from terminal, memory, or network [terminal]?
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#interface fastEthernet 0/0
R1(config-if)#ip address 192.168.157.135 255.255.255.0 #和VMnet8必须同一子网
R1(config-if)#exit
R1(config)#interface fastEthernet 0/1
R1(config-if)#ip address 223.0.113.1 255.255.255.0
R1(config-if)#exit
R1(config)#ip route 0.0.0.0 0.0.0.0 192.168.157.2 #指向VMnet8网关
```

‍

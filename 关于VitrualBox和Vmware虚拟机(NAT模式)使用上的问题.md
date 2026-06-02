最近在部署一台 Web 服务器时，我考虑使用虚拟机来完成。由于长期使用 VMware Workstation（以下简称 VMware）部署虚拟机，这次想尝试一下其他虚拟化产品，于是选择了由 Oracle 收购的开源虚拟化产品 VirtualBox（以下简称 VBox）。然而，实际使用体验并不理想。
下面先说明本次部署所涉及的软件及版本：

- **软件版本**  
  宿主机：Windows 10  
  虚拟机软件：VirtualBox 7.1  
  虚拟机客户机：Debian 12

- **网络配置**  
  宿主机：192.168.2.30/24  
  虚拟机客户机：203.0.113.129/24

通常情况下，VMware 的客户机设置为 NAT 模式后，宿主机既能 ping 通客户机，也能访问客户机中的任何服务。而 VBox 的客户机同样设置为 NAT 模式时，宿主机却无法 ping 通客户机，也无法访问客户机中的任何应用服务，但客户机本身可以正常访问外部网络和服务。

为了解决 VBox 的 NAT 问题，我查阅了官方产品手册（全英文），结合翻译和实际环境中的问题，简单梳理一下 VBox 的 NAT 工作机制

VBox 中的 NAT 可以看作是一台虚拟路由器，属于 VBox 的核心网络引擎。在这款虚拟机中，虚拟路由器的逻辑位置位于虚拟机和宿主机之间（详见上图示意图）。VBox 的 NAT 模式有一个特点，同时也是缺点：它与真实环境中的路由器网络非常相似——虚拟客户机对外部网络（包括宿主机）是不可见且无法访问的。对此，官方提供了解决办法：使用 NAT 端口转发，即可实现宿主机访问客户机的网络服务。

此外，官方还说明了 VBox NAT 模式的一些使用限制，需要注意以下几点:

- ICMP 协议限制：ping、tracert 等工具可能无法正常使用。
- 仅支持特定协议：只支持 TCP 和 UDP，不支持 GRE、VPN、DDNS 等。
- NAT 端口转发限制：不支持公共端口（0-1024）。

接下来再看一下 VMware 的 NAT 工作机制。VMware 安装到 Windows 宿主机系统后，会自动生成一个名为 VMnet8 的虚拟网络适配器。客户机在 NAT 模式下访问外部网络时，机制与 VBox 大致相似。不同的是，VMware 中宿主机可以直接访问虚拟客户机，而无需依赖端口转发。这是因为宿主机和虚拟客户机共用 VMnet8 这个虚拟适配器，宿主机可以通过 NAT 直接与虚拟客户机进行双向通信。

关于 VMware NAT 模式的使用限制，官方也给出了说明，需要注意以下几点：

- NAT 并不完全透明：虽然可以手动配置 NAT 设备来建立虚拟机与互联网服务器的连接，但通常不允许从外部互联网主动向虚拟机发起连接。在实际环境中，这会导致一些需要外部互联网主动发起连接的 TCP 和 UDP 协议无法自动运行或根本无法运行（例如 DDNS）。
- 提供一定的防火墙保护：NAT 只允许虚拟机通过它访问外部网络，而外部网络无法直接访问虚拟机。
- 与宿主机之外的设备互访受限：在 NAT 模式下，虚拟机可以与宿主机双向访问，但无法与宿主机所在子网中的其他外部主机直接互访，除非启用 NAT 端口转发。

此外，上述两种虚拟化产品的 NAT 模式还有一个共同缺点：会造成一定的性能损失。当然，如果宿主机硬件性能足够充裕，这种损失的影响非常有限。

附：官方资料
- [VirtualBox中的虚拟网络](https://www.virtualbox.org/manual/topics/networkingdetails.html#nat-limitations)
- [VMware Workstation Pro中配置网络地址转换](https://docs.vmware.com/cn/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-89311E3D-CCA9-4ECC-AF5C-C52BE6A89A95.html)
- [VMware Workstation Pro中NAT配置的功能和限制](https://docs.vmware.com/cn/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-122B85E2-E60F-4C44-87B2-A3F8DDC88D66.html)

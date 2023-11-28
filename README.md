# 虚拟化测试用户指南
本文介绍虚拟化和服务器兼容性测试环境准备和用例自动化测试的操作流程。

#### 虚拟化介绍

在计算机技术中，虚拟化一般指通过对计算机物理资源的抽象，提供一个或多个操作环境，实现资源的模拟、隔离或共享等。虚拟化使得一台物理服务器上可以运行多台虚拟机，虚拟机共享物理机的处理器、内存、I/O资源等，但逻辑上虚拟机之间是互相隔离的。 

详细介绍可参考openEuler社区虚拟化用户指南：  

https://docs.openeuler.org/zh/docs/22.03_LTS_SP2/docs/Virtualization/%E8%AE%A4%E8%AF%86%E8%99%9A%E6%8B%9F%E5%8C%96.html  

#### 测试各部件介绍

服务器：通用服务器即可，支持AArch64和x86_64处理器架构。   

openEuler：主机操作系统。  

基础虚拟化组件：  

KVM：提供核心的虚拟化基础设施，使Linux系统成为一个hypervisor，支持多个虚拟机同时在该主机上运行。已包含在linux内核中。  
Stratovirt：openEuler社区项目，面向云数据中心的企业级虚拟化VMM(Virtual Machine Monitor)，实现了一套架构统一支持虚拟机、容器、Serverless三种场景。StratoVirt在轻量低噪、软硬协同、Rust语言级安全等方面具备关键技术竞争优势。  

QEMU：模拟处理器并提供一组设备模型，配合KVM实现基于硬件的虚拟化模拟加速。  

Libvirt：为管理虚拟机提供工具集，主要包含统一、稳定、开放的应用程序接口（API）、守护进程 （Libvirtd）和一个默认命令行管理工具（virsh）。  

Open vSwitch：为虚拟机提供虚拟网络的工具集，支持编程扩展，以及标准的管理接口和协议（如NetFlow， sFlow，IPFIX， RSPAN， CLI， LACP， 802.1ag）。  

#### 环境安装

##### 服务器要求

在openEuler系统中安装虚拟化组件，最低硬件要求：

AArch64处理器架构：ARMv8以上并且支持虚拟化扩展

x86_64处理器架构：支持VT-x

2核CPU

4GB的内存

16GB可用磁盘空间

##### 环境安装过程

1.  安装openEuler操作系统

    镜像获取地址：https://www.openeuler.org/zh/download/?version=openEuler%2022.03%20LTS%20SP2

    选择“x86_64”-“服务器”-“Offline Standard ISO”。

    注意：软件选择的基本环境可以按照“虚拟化主机”、附加安装“虚拟switch”安装。

    具体可参考社区安装指导：

    https://docs.openeuler.org/zh/docs/22.03_LTS_SP2/docs/Installation/%E5%AE%89%E8%A3%85%E6%8C%87%E5%AF%BC.html

2.  安装虚拟化组件

    #yum install -y qemu

    启动libvirtd服务并开机自启动（安装系统时已安装）：

    #systemctl start libvirtd

    #systemctl enable libvirtd

    验证安装：

    $ ls /dev/kvm

    $ ls /sys/module/kvm

    $ rpm -qi qemu

    $ rpm -qi libvirt

    $ systemctl status libvirtd

    安装rust\cargo等组件:

    #curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

    装完后rustc -V，如果错误，重启主机

    #yum install -y git

    #git clone https://gitee.com/openeuler/stratovirt.git

    #cd stratovirt

    #git checkout dev

    #cargo build

    将构建的stratovirt拷贝至/usr/bin/

    #cp ${pwd}/target/debug/stratovirt /usr/bin/

    验证stratovirt版本：

    #stratovirt -V

    禁用SELinux的强制访问控制功能

    #setenforce 0

    安装依赖包edk2-ovmf

    #yum install -y edk2-ovmf

##### 准备oec-hardware测试环境

1.  安装oec-hardware

    使用 dnf 在被测主机安装oec-hardware。

    #dnf install oec-hardware

    输入 oech 命令，可正常运行，则表示安装成功。

    #oech

    安装组件expect：

    #yum install -y expect

2.  获取虚拟机镜像

    使用qemu创建虚拟机时使用的镜像：

    https://repo.openeuler.org/openEuler-23.09/virtual_machine_img/x86_64/openEuler-23.09-x86_64.qcow2.xz

    使用stratovirt创建虚拟机时使用的镜像：

    https://repo.openeuler.org/openEuler-23.09/stratovirt_img/x86_64/openEuler-23.09-stratovirt-x86_64.img.xz

    https://repo.openeuler.org/openEuler-23.09/stratovirt_img/x86_64/std-vmlinuxz

    本地下载后创建目录：

    #mkdir /home/kvm

    上传至/home/kvm并解压镜像：

    #cd /home/kvm

    #xz -dk openEuler-23.09-stratovirt-x86_64.img.xz

    #xz -dk openEuler-23.09-x86_64.qcow2.xz

3.  修改虚拟机xml文件中的镜像位置

    qemu：
    
    #cd /opt/oec-hardware/tests/virtualization/qemu_*

    #vi qemu.xml

    修改source file:

    source file='/home/kvm/openEuler-23.09-x86_64.qcow2'

    stratovirt：

    #cd /opt/oec-hardware/tests/virtualization/stratovirt_*

    #vi vm_stratovirt.xml

    修改kernel：

    在kernel行修改值为/home/kvm/std-vmlinuxz

    修改source file:

    source file='/home/kvm/openEuler-23.09-stratovirt-x86_64.img'

#### 执行测试

#####自动化测试执行

    #oech
    
    Please select test category.  
    No.   category  
    1     compatible  
    2     virtualization  
    Plese selection test category No:2  
    The openEuler Virtualization Test Suite  
    Please provide your Compatibility Test ID:  
    Please provide your Product URL:  
    Please provide the Compatibility Test Server (Hostname or Ipaddr):  
    oec-hardware:             1.1.4  
    Compatibility Test ID:  
    Hardware Info:           xxx  
    Product URL:  
    OS Info:                 openEuler 22.03 (LTS-SP2)  
    Kernel Info:             5.10.0-153.12.0.92.oe2203sp2.x86_64  
    Test Server:  

    These tests are recommended to complete the virtualization test:  
    No. Run-Now?  status    Class  
    1     yes     NotRun    qemu_001  
    2     yes     NotRun    qemu_003  
    3     yes     NotRun    qemu_005  
    4     yes     NotRun    qemu_007  
    5     yes     NotRun    qemu_008  
    6     yes     NotRun    qemu_010  
    7     yes     NotRun    stratovirt_004  
    8     yes     NotRun    stratovirt_010  
    Ready to begin testing? (run|edit|quit) e  

    选择要跑的脚本（只能逐条执行），将其他脚本的“Run-Now”状态置为no，输入r回车，愉快地跑起来吧。  
    Select tests to run:  
    No. Run-Now?  status    Class  
    1     yes     NotRun    qemu_001  
    2     no      NotRun    qemu_003  
    3     no      NotRun    qemu_005  
    4     no      NotRun    qemu_007  
    5     no      NotRun    qemu_008  
    6     no      NotRun    qemu_010  
    7     no      NotRun    stratovirt_004  
    8     no      NotRun    stratovirt_010    
    Selection (<number>|all|none|quit|run): r  
    There are 1 selected test suites: qemu_001.  
    Start to run 1/1 test suite: qemu_001.  
    Strat testcase QemuTest001.  
    Create VM succeed.  
    The number of vCPUs is correct.  
    ***teardown start  
    Destory vm succeed.  
    End to run 1/1 test suite: qemu_001.  
    The  doesn't have quadruple information, couldn't get hardware.  
    There is no cert device need to export.  
    -----------------  Summary -----------------  
    qemu_001                                PASS  
    Log saved to file: /usr/share/oech/logs/oech-20231116144103-9dH7WoRTen.tar succeed.  
    Do you want to submit last result? (y|n)

    所有用例全部执行通过，无法继续执行自动化脚本，需要清除已有结果，重新跑：

    #oech --clean

#####SRIOV测试

    1）主机BIOS开启SR-IOV

    举例：超聚变：Advanced>Peripheral Configuration>SR-IOV Setup Settings>PCIe SR-IOV:Enabled  

    2）系统设置：  
    #vi /etc/default/grub  
    在GRUB_CMDLINE_LINUX最后追加“intel_iommu=on iommu=pt iommu.passthrough=1”参数。  
    刷新 grub.cfg 文件：  
    #grub2-mkconfig -o /boot/grub2/grub.cfg   
    #vi /boot/efi/EFI/openEuler/grub.cfg  
    找到kernel的两个位置，在末尾添加“intel_iommu=on iommu=pt iommu.passthrough=1”参数。  
    重启主机，配置成功后:  
    #cat /proc/cmdline | grep intel_iommu有回显  
    BOOT_IMAGE=/vmlinuz-5.10.0-153.12.0.92.oe2203sp2.x86_64 root=UUID=f68bef38-e26c-4067-8e29-3718f6e632cc ro resume=UUID=9244f971-4fc4-492e-8683-4cd47b90a44d cgroup_disable=files apparmor=0 crashkernel=512M intel_iommu=on iommu=pt iommu.passthrough=1  
    #dmesg | grep -e IOMMU有回显  
    [    0.058175] DMAR: IOMMU enabled  
    [    0.215681] DMAR-IR: IOAPIC id 12 under DRHD base  0xeaffc000 IOMMU 6  
    [    0.215683] DMAR-IR: IOAPIC id 11 under DRHD base  0xe13fc000 IOMMU 5  
    [    0.215686] DMAR-IR: IOAPIC id 10 under DRHD base  0xdabfc000 IOMMU 4  
    [    0.215688] DMAR-IR: IOAPIC id 18 under DRHD base  0xfbffc000 IOMMU 3  
    [    0.215690] DMAR-IR: IOAPIC id 17 under DRHD base  0xf27fc000 IOMMU 2  
    [    0.215692] DMAR-IR: IOAPIC id 16 under DRHD base  0xebffc000 IOMMU 1  
    [    0.215695] DMAR-IR: IOAPIC id 15 under DRHD base  0xeb7fc000 IOMMU 0  
    [    0.215697] DMAR-IR: IOAPIC id 8 under DRHD base  0xda3fc000 IOMMU 7  
    [    0.215699] DMAR-IR: IOAPIC id 9 under DRHD base  0xda3fc000 IOMMU 7  

    3）设置网卡VF  
    #ls -l /sys/class/net/ | grep -v virtual  
    #cat /sys/class/net/[ethx]/device/sriov_totalvfs  
    #echo [num] > /sys/class/net/[ethx]/device/sriov_numvfs  
    #lspci | grep "Virtual Function"  
    37:00.2 Ethernet controller: Mellanox Technologies MT27800 Family [ConnectX-5 Virtual Function]

    4）设置虚拟机xml
    将PCI号修改为系统查询(lspci | grep "Virtual Function")回显;tag id需要此网卡对应的交换机端口放通此vlan。

    <interface type='hostdev' managed='yes'>
    <source>
    <address type='pci' domain='0x0000' bus='0x37' slot='0x00' function='0x02'/>
    </source>
    <vlan>
    <tag id='300'/>
    </vlan>
    </interface>

    由于要拉起两个虚拟机互ping，将镜像复制到新目录，修改镜像路径：

    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' iothread='1'/>
      <source file='/home/kvm/openEuler-23.09-x86_64.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <boot order='1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
    </disk>

    5）拉起虚拟机测试

    # virsh create qemu-test1-PING.xml  
    # virsh create qemu-test2-PING.xml  
    登录虚拟机控制台配置IP（根据交换机vlanif配置合法IP和网关）：  
    # virsh console test*  
    # [root@localhost ~]# ifconfig ens7 x.x.x.x/x  
    # [root@localhost ~]# route add default gw x.x.x.x  
    跨网卡互ping测试：  
    # [root@localhost ~]#ping x.x.x.x  
    巨型帧互ping测试：  
    # [root@localhost ~]# ping -s 9000 x.x.x.x  
    6）直通13vf查询测试：  
    # virsh create qemu-test4-13VF.xml  
    在主机ip link show与虚拟机里ip link show对比mac地址是否一致。  

#### 参考链接

1. openEuler社区虚拟化用户指南：  
https://docs.openeuler.org/zh/docs/22.03_LTS_SP2/docs/Virtualization/%E8%AE%A4%E8%AF%86%E8%99%9A%E6%8B%9F%E5%8C%96.html  

2. openEuler安装指导：  
https://docs.openeuler.org/zh/docs/22.03_LTS_SP2/docs/Installation/%E5%AE%89%E8%A3%85%E6%8C%87%E5%AF%BC.html  

3. stratovirt用户指南：  
https://docs.openeuler.org/zh/docs/22.03_LTS_SP2/docs/StratoVirt/%E5%AE%89%E8%A3%85StratoVirt.html  

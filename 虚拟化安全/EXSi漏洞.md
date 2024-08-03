- https://straightblast.medium.com/my-poc-walkthrough-for-cve-2021-21974-a266bcad14b9
  - 本地备份文件：My RCE PoC walkthrough for (CVE-2021–21974) VMware ESXi OpenSLP heap-overflow vulnerability _ by Johnny Yu (@straight_blast) _ Medium
  - 打开本地备份文件时需要关闭浏览器javascript，从而避免页面跳转到404页面
  - 由于关闭js，本地备份文件中部分超链接无法访问并返回内容，可以在html文件中搜索http:// 或github，从而找到该位置所引用的外部链接的内容

- https://blog.csdn.net/weixin_46944519/article/details/125411554

- https://paper.seebug.org/1500/

- https://xz.aliyun.com/t/9243

- http://www.ctfiot.com/39518.html

- https://www.geekby.site/2022/05/vcenter%E6%BC%8F%E6%B4%9E%E5%88%A9%E7%94%A8/


esxi相关新漏洞
# https://op-c.net/blog/vmware-issues-security-patches-for-esxi-workstation-and-fusion-flaws/
VMware 已发布安全补丁来解决 ESXi、Workstation 和 Fusion 中的四个漏洞。两个严重的释放后使用漏洞 CVE-2024-22252 和 CVE-2024-22253 可能会导致代码执行。此外，还修复了越界写入漏洞（CVE-2024-22254）和信息泄露漏洞（CVE-2024-22255）。

漏洞详细信息：
CVE-2024-22252和CVE-2024-22253：
类型：XHCI USB 控制器中的释放后使用漏洞。
影响：可能导致代码作为虚拟机的 VMX 进程执行。
CVSS 分数：工作站和 Fusion 为 9.3，ESXi 为 8.4。
受影响的产品：ESXi、工作站、Fusion。
CVE-2024-22254：
类型：ESXi 中的越界写入漏洞。
影响：可能允许恶意行为者触发沙箱逃逸。
CVSS 分数：7.9。
CVE-2024-22255：
类型：UHCI USB 控制器中的信息泄露漏洞。
影响：可能允许攻击者从 vmx 进程中泄漏内存。
CVSS 分数：7.1。
减轻：
VMware 已在以下版本中解决了这些问题：

ESXi 6.5 – 6.5U3v
ESXi 6.7 – 6.7U3u
ESXi 7.0 – ESXi70U3p-23307199
ESXi 8.0 – ESXi80U2sb-23305545 和 ESXi80U1d-23299997
VMware 云基础 (VCF) 3.x
工作站 17.x – 17.5.1
Fusion 13.x (macOS) – 13.5.1
建议用户更新到最新版本以缓解这些漏洞。


# 天工产出 https://mp.weixin.qq.com/s/3QVV8F97f9TsSczWxUIMFA

CVE-2024-22252

VMware Workstation/ESXi 远程代码执行
CVSSv3: 9.3

VMware ESXi、Workstation和Fusion的XHCI USB控制器存在use-after-free漏洞。在虚拟机上拥有本地管理权限的恶意行为者可能会利用此问题以在主机上运行的虚拟机VMX进程的身份执行代码。在ESXi上，利用包含在VMX沙箱中，而在Workstation和Fusion上，这可能导致在安装Workstation或Fusion的机器上执行代码。



CVE-2024-22255 

VMware Workstation/ESXi 信息泄露

CVSSv3: 7.1

VMware ESXi、Workstation和Fusion的UHCI USB控制器存在信息泄露漏洞。具有虚拟机管理权限的恶意行为者可能会利用这个问题从vmx进程泄漏内存。



CVE-2024-22251

VMware Workstation 信息泄露
CVSSv3: 5.9

VMware Workstation和Fusion在USB CCID(芯片卡接口设备)中存在越界读取漏洞。在虚拟机上具有本地管理权限的恶意行为者可能会触发越界读取，从而导致信息泄露。


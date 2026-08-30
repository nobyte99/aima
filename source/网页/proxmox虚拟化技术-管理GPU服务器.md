
proxmox可以用来实现一台多卡GPU服务器的虚拟化工作，通过构建虚拟机分配显卡，实现资源的高效利用。
1、proxmox的基本简介，见[@ProxmoxXuNiHuaPingTaiJianJieZhongGuoZhiWang];
2、proxmox主要支持单台物理服务器资源的虚拟化，见[@ProxmoxFuWuQiXuNiHuaXiTongZaiGaoXiaoWangLuoFuWuZhongDeYingYongZhongGuoZhiWang]; 支持将物理服务器上的GPU映射为虚拟机上的vGPU，实现在虚拟机上使用GPU显卡。见[@NVIDIAVGPUProxmox],[@ProxmoxVE84], 
3、多个物理服务器上的proxmox虚拟机可以组成proxmox虚拟集群，如果该集群计划启用高可用性（HA），需要配置共享存储（如 NFS、Ceph 或 GlusterFS），以确保虚拟机和容器数据在节点间可访问,见[@chrisProxmoxVEFuWuQiJiQun2025].如果不启用高可用性，一般也推荐一个集群至少包含3个节点，否则会出现脑裂现象（split-brain）。
### IaaS进一步支持Container Service2.0 
+ 2014.11.13  Amazon发布AWS Container Service  
+ 2014.11.05  Google发布Google Container engine  

本质上都是将虚拟机以集群方式提供给容器使用。在集群上定义容器业务。  

### 分析AWS Container Service  
AWS首款结合着Docker技术提供的云计算平台，让用户可以通过AWS云平台快速实现Dockers部署、 管理和扩展。  
Amazon EC2 Container Service的主要特点包括：

1.简化集群管理： 最大限度对容器集群进行启动、管理和终止，实现对集群运行状态的监控。并可以跨不同可用区域，
管理几千集群容器。  
2.高性能：根据应用需求，通过多节点满足高性能需求。且节点扩展在几秒即可完成。  
3.灵活：跨集群分布容器，优化资源可用性和利用率。    
4.移动性：通过Dockers技术，实现应用双向可迁移性（从已有数据中心迁移到云，或者从云迁移到已有数据中心）  

使用过程：  
1. Create a cluster, or decide to use the default one for your account in the target Region.  
2. Create your task definitions and register them with the cluster.  
3. Launch some EC2 instances and register them with the cluster.  
4. Start the desired number of copies of each task.  
5.  Monitor the overall utilization of the cluster and the overall throughput of your application, and make adjustments as desired. For example, you can launch and then register additional EC2 instances in order to expand the cluster's pool of available resources.  

### 分析Google Contianer Service  

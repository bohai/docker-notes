..  
声明：   
本博客欢迎转发，但请保留原作者信息!   
博客地址：http://blog.csdn.net/halcyonbaby   
新浪微博：[@寻觅神迹]( www.weibo.com/u/2230330930)

内容系本人学习、研究和总结，如有雷同，实属荣幸！   

-----
# Docker1.9后的network
### 介绍与使用
1.9之后，在Docker中network从实验特性转为正式特性发布。   
从命令行可以看到新增如下命令：
<pre><code>[root@localhost system]# docker help network

Usage:  docker network [OPTIONS] COMMAND [OPTIONS]

Commands:
  create                   Create a network
  connect                  Connect container to a network
  disconnect               Disconnect container from a network
  inspect                  Display detailed network information
  ls                       List all networks
  rm                       Remove a network

Run 'docker network COMMAND --help' for more information on a command.

  --help=false       Print usage
</code></pre>
可以看到Docker daemon启动后默认创建了3个网络：
分别使用了bridge、null、host三种内置network driver。
<pre><code>[root@localhost system]# docker network ls
NETWORK ID          NAME                DRIVER
f280d6a13422        bridge              bridge
f5d11bed22a2        none                null
18642f53648f        host                host
</code></pre>
我们来仔细看下三个网络的详细信息：  
Name是network的名字，用户可以随意定义。  
Id是network内部的uuid，全局唯一。   
Scope目前有两个值“local”、“remote”，表示是本机网络还是多机网络。  
Driver是指network driver的名字。  
IPAM是负责IP管理发放的driver名字与配置信息（我们在bridge网络中可以看到该信息）。  
Container内记录了使用这个网络的容器的信息。  
Options内记录了driver所需的各种配置信息。  
<pre><code>[root@localhost temp]# docker network inspect none
[
    {
        "Name": "none",
        "Id": "1abfa4750ada3be20927c3c168468f9a64efd10705d3be8958ae1eef784b28ef",
        "Scope": "local",
        "Driver": "null",
        "IPAM": {
            "Driver": "default",
            "Config": []
        },
        "Containers": {},
        "Options": {}
    }
]
[root@localhost temp]# docker network inspect host
[
    {
        "Name": "host",
        "Id": "001c9c9047d90efff0b64bf80e49ff7ec33421374b2c895169a0f9e096eb791d",
        "Scope": "local",
        "Driver": "host",
        "IPAM": {
            "Driver": "default",
            "Config": []
        },
        "Containers": {},
        "Options": {}
    }
]
[root@localhost temp]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "201fbcb64b75977889f5d9c1e88c756308a090eb611397dbd0bb5c824d429276",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.17.42.1/16",
                    "Gateway": "172.17.42.1"
                }
            ]
        },
        "Containers": {
            "4d4d37853115562080613393c6f605a9ec2b06c3660dfa0ca4e27f2da266773d": {
                "EndpointID": "09e332644c539cec8a9852a11d402893bc76a5559356817192657b5840fe2de3",
                "MacAddress": "02:42:ac:11:00:01",
                "IPv4Address": "172.17.0.1/16",
                "IPv6Address": ""
            }
        },

        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        }
    }
]
</code></pre>
容器网络的各种操作：  
-> 创建/添加/解除/删除网络
<pre><code>[root@localhost temp]# docker network create -d bridge --ip-range=192.168.1.0/24 --gateway=192.168.1.1 --subnet=192.168.1.0/24  bridge2
b18f4fb74ebd32b9f67631fd3fd842d09b97c30440efebe254a786d26811cf66
[root@localhost temp]# docker network ls
NETWORK ID          NAME                DRIVER
1abfa4750ada        none                null
001c9c9047d9        host                host
b18f4fb74ebd        bridge2             bridge
201fbcb64b75        bridge              bridge
[root@localhost temp]# docker exec vim ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
69: eth0@if70: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:01 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:1/64 scope link
       valid_lft forever preferred_lft forever
[root@localhost temp]# docker network connect bridge2 vim
[root@localhost temp]# docker exec vim ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
69: eth0@if70: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:01 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:1/64 scope link
       valid_lft forever preferred_lft forever
72: eth1@if73: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:c0:a8:01:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.2/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::42:c0ff:fea8:102/64 scope link
       valid_lft forever preferred_lft forever
[root@localhost temp]# docker network disconnect bridge2 vim
[root@localhost temp]# docker exec vim ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
69: eth0@if70: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:01 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:1/64 scope link
       valid_lft forever preferred_lft forever
[root@localhost temp]# docker network rm bridge2
[root@localhost temp]# docker network ls
NETWORK ID          NAME                DRIVER
1abfa4750ada        none                null
001c9c9047d9        host                host
201fbcb64b75        bridge              bridge
</code></pre>
指定network创建容器：
<pre><code>[root@localhost temp]# docker run -d --net=bridge2 busybox sleep 100
6e7146eb4afee6a47e55a76ead5f6decb98e650ffdb71dcdba544dcf72190b47
[root@localhost temp]# docker ps
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS               NAMES
6e7146eb4afe        busybox               "sleep 100"         4 seconds ago       Up 1 seconds                            sharp_fermi
4d4d37853115        mbrt/golang-vim-dev   "/bin/bash"         10 weeks ago        Up 23 hours                             vim
[root@localhost temp]# docker network inspect bridge2
[
    {
        "Name": "bridge2",
        "Id": "1920ba2095004a12e87cc9b5a6908766cc66b7d259603100b4860a43a203640b",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "192.168.1.0/24",
                    "IPRange": "192.168.1.0/24",
                    "Gateway": "192.168.1.1"
                }
            ]
        },
        "Containers": {
            "6e7146eb4afee6a47e55a76ead5f6decb98e650ffdb71dcdba544dcf72190b47": {
                "EndpointID": "5914c79d9b22663be43f4e279d799a1ce99a7b8eaa0beb15e6edd62a928822d4",
                "MacAddress": "02:42:c0:a8:01:02",
                "IPv4Address": "192.168.1.2/24",
                "IPv6Address": ""
            }
        },
        "Options": {}
    }
]
[root@localhost temp]# docker exec sharp_fermi ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
74: eth0@if75: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue
    link/ether 02:42:c0:a8:01:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:c0ff:fea8:102/64 scope link
       valid_lft forever preferred_lft forever
</code></pre>
### driver plugin机制与driver plugin实现  
#### 简介
Docker插件一览：  
http://docs.docker.com/engine/extend/plugins/
Docker的plugin采用了一种out-of-process的方式。  
这样有两个好处，便于扩展、动态增加删除；同时代码上与Docker完全解耦。 

Plugin是运行在Docker host上的一个进程，通过将一个文件放在plugin目录来向Docker注册，
从而被docker发现机制发现。

插件名字建议使用比较短的小写单词。插件可以在容器内或者容器外运行，建议在容器外。  
#### 插件目录  
插件目录下可以放三种文件：
<pre><code>
.sock files are UNIX domain sockets.
.spec files are text files containing a URL, such as unix:///other.sock.
.json files are text files containing a full json specification for the plugin.
</code></pre>
.sock文件一般放置在/run/docker/plugins下；.spec/.json文件一般放置在/etc/docker/plugins
或者/usr/lib/docker/plugins下。

json文件范例：  
<pre><code>{
  "Name": "plugin-example",
  "Addr": "https://example.com/docker/plugin",
  "TLSConfig": {
    "InsecureSkipVerify": false,
    "CAFile": "/usr/shared/docker/certs/example-ca.pem",
    "CertFile": "/usr/shared/docker/certs/example-cert.pem",
    "KeyFile": "/usr/shared/docker/certs/example-key.pem",
  }
}
</code></pre>
#### 插件的其他说明
Plugin需要在docker启动前启动；更新plugin时需要先停止docker daemon，更新后再启动docker daemon。  

插件在第一次使用时激活。docker会根据指定的插件名字，在插件目录下查找。（感觉docker应该增加一个接口，查询本机插件列表）   

Docker与plugin间使用，json格式基于Http的RPC消息，消息类型为post。 

握手消息：   
<pre><code>/Plugin.Activate
Request: empty body

Response:

{
    "Implements": ["VolumeDriver"]
}
</code></pre>

#### Plugin实现  
主要需要实现如下消息：
<pre><code>/Plugin.Activate
/NetworkDriver.GetCapabilities
/NetworkDriver.CreateNetwork
/NetworkDriver.DeleteNetwork
/NetworkDriver.CreateEndpoint
/NetworkDriver.EndpointOperInfo
/NetworkDriver.DeleteEndpoint
/NetworkDriver.Join
/NetworkDriver.Leave
/NetworkDriver.DiscoverNew 
/NetworkDriver.DiscoverDelete
</code></pre>

详细参考:
https://github.com/docker/libnetwork/blob/master/docs/remote.md

### libnetwork与docker  
#### 调用关系：   
docker daemon---->libnetwork----->network plugin   
#### CNM介绍  
https://github.com/docker/libnetwork/blob/master/docs/design.md
CNM全称Container Network Model。主要对libnetwork的网络模型进行了定义。  
#####主要有三个概念：  
* network
一组可以直接互相通信的endpoint构成。通常的实现手段linux bridge/ovs等。
* sandbox
sandbox包含了一个容器的网络栈。一般包含interface/route/dns设置等。一般通过namespace实现。 
一个sandbox可以包含多个属于不同network的endpoint。
* endpoint  
endpoint将一个sandbox与一个网络连通。  
通常可以使用linux bridge的veth pair或者ovs的internal port等技术。  

#####CNM的主要对象有
* NetworkController   
主要负责管理driver，提供创建network接口。   
* Driver   
提供network/sandbox/endpoint的实现。   
* Network   
* Endpoint    
* Sandbox   

#### 代码分析  
#####Docker daemon
* 在daemon初始化时创建了三个网络
daemon.go  函数NewDaemon：
<pre><code>func NewDaemon(config *Config, registryService *registry.Service) (daemon *Daemon, err error) {
...
d.netController, err = d.initNetworkController(config)
	if err != nil {
		return nil, fmt.Errorf("Error initializing network controller: %v", err)
	}
...
}
</code></pre>

* initNetworkController函数在daemon_unix.go/daemon_windows.go中，以unix为例：   
主要做了以下几件事情：
初始化controller, 初始化null/host/bridge三个内置网络。
<pre><code>
func (daemon *Daemon) initNetworkController(config *Config) (libnetwork.NetworkController, error) {
	netOptions, err := daemon.networkOptions(config)
	if err != nil {
		return nil, err
	}

	controller, err := libnetwork.New(netOptions...)
	if err != nil {
		return nil, fmt.Errorf("error obtaining controller instance: %v", err)
	}

	// Initialize default network on "null"
	if _, err := controller.NewNetwork("null", "none", libnetwork.NetworkOptionPersist(false)); err != nil {
		return nil, fmt.Errorf("Error creating default \"null\" network: %v", err)
	}

	// Initialize default network on "host"
	if _, err := controller.NewNetwork("host", "host", libnetwork.NetworkOptionPersist(false)); err != nil {
		return nil, fmt.Errorf("Error creating default \"host\" network: %v", err)
	}

	if !config.DisableBridge {
		// Initialize default driver "bridge"
		if err := initBridgeDriver(controller, config); err != nil {
			return nil, err
		}
	}

	return controller, nil
}
</code></pre>
* 启动容器时创建网络
<pre><code>
api/server/router/container/container.go
local.NewPostRoute("/containers/{name:.*}/start", r.postContainersStart),
postContainersStart中调用了containerStart函数
daemon/start.go
func (daemon *Daemon) containerStart(container *Container) (err error) {
	...
	if err := daemon.initializeNetworking(container); err != nil {
		return err
	}
        ...
}
daemon/container_unix.go
func (daemon *Daemon) initializeNetworking(container *Container) error {
...
	if err := daemon.allocateNetwork(container); err != nil {
		return err
	}
...
}
func (daemon *Daemon) allocateNetwork(container *Container) error {
...
	for n := range container.NetworkSettings.Networks {
		if err := daemon.connectToNetwork(container, n, updateSettings); err != nil {
			return err
		}
	}
...
}
func (daemon *Daemon) connectToNetwork(container *Container, idOrName string, updateSettings bool) (err error) {
...
// 创建endpoint
ep, err = n.CreateEndpoint(endpointName, createOptions...)
...
// 获得sandbox
sb := daemon.getNetworkSandbox(container)
...
// 接入sandbox
	if err := ep.Join(sb); err != nil {
		return err
	}
...
}
</code></pre>

* 创建Network  
实际上是调用libnetwork的NewNetwork接口。
network的所有API，都会被路由到daemon/network.go中的函数上
<pre><code>
daemon/network.go
// CreateNetwork creates a network with the given name, driver and other optional parameters
func (daemon *Daemon) CreateNetwork(name, driver string, ipam network.IPAM, options map[string]string) (libnetwork.Network, error) {
	c := daemon.netController
	if driver == "" {
		driver = c.Config().Daemon.DefaultDriver
	}

	nwOptions := []libnetwork.NetworkOption{}

	v4Conf, v6Conf, err := getIpamConfig(ipam.Config)
	if err != nil {
		return nil, err
	}

	nwOptions = append(nwOptions, libnetwork.NetworkOptionIpam(ipam.Driver, "", v4Conf, v6Conf))
	nwOptions = append(nwOptions, libnetwork.NetworkOptionDriverOpts(options))
	return c.NewNetwork(driver, name, nwOptions...)
}
</code></pre>
* Connect容器到Network  
我们再看看connect容器到Network。  
可以看到最后也是调用了libnetwork的接口。  
创建EP，加入sandbox。  
<pre><code>
// ConnectContainerToNetwork connects the given container to the given
// network. If either cannot be found, an err is returned. If the
// network cannot be set up, an err is returned.
func (daemon *Daemon) ConnectContainerToNetwork(containerName, networkName string) error {
	container, err := daemon.Get(containerName)
	if err != nil {
		return err
	}
	return daemon.ConnectToNetwork(container, networkName)
}
// ConnectToNetwork connects a container to a network
func (daemon *Daemon) ConnectToNetwork(container *Container, idOrName string) error {
	if !container.Running {
		return derr.ErrorCodeNotRunning.WithArgs(container.ID)
	}
	if err := daemon.connectToNetwork(container, idOrName, true); err != nil {
		return err
	}
	if err := container.toDiskLocking(); err != nil {
		return fmt.Errorf("Error saving container to disk: %v", err)
	}
	return nil
}
func (daemon *Daemon) connectToNetwork(container *Container, idOrName string, updateSettings bool) (err error) {
	if container.hostConfig.NetworkMode.IsContainer() {
		return runconfig.ErrConflictSharedNetwork
	}

	if runconfig.NetworkMode(idOrName).IsBridge() &&
		daemon.configStore.DisableBridge {
		container.Config.NetworkDisabled = true
		return nil
	}

	controller := daemon.netController

	n, err := daemon.FindNetwork(idOrName)
	if err != nil {
		return err
	}

	if updateSettings {
		if err := daemon.updateNetworkSettings(container, n); err != nil {
			return err
		}
	}

	ep, err := container.getEndpointInNetwork(n)
	if err == nil {
		return fmt.Errorf("container already connected to network %s", idOrName)
	}

	if _, ok := err.(libnetwork.ErrNoSuchEndpoint); !ok {
		return err
	}

	createOptions, err := container.buildCreateEndpointOptions(n)
	if err != nil {
		return err
	}

	endpointName := strings.TrimPrefix(container.Name, "/")
	ep, err = n.CreateEndpoint(endpointName, createOptions...)
	if err != nil {
		return err
	}
	defer func() {
		if err != nil {
			if e := ep.Delete(); e != nil {
				logrus.Warnf("Could not rollback container connection to network %s", idOrName)
			}
		}
	}()

	if err := daemon.updateEndpointNetworkSettings(container, n, ep); err != nil {
		return err
	}

	sb := daemon.getNetworkSandbox(container)
	if sb == nil {
		options, err := daemon.buildSandboxOptions(container, n)
		if err != nil {
			return err
		}
		sb, err = controller.NewSandbox(container.ID, options...)
		if err != nil {
			return err
		}

		container.updateSandboxNetworkSettings(sb)
	}

	if err := ep.Join(sb); err != nil {
		return err
	}

	if err := container.updateJoinInfo(n, ep); err != nil {
		return derr.ErrorCodeJoinInfo.WithArgs(err)
	}

	return nil
}
</code></pre>

其他功能和上述分析类似。
##### libnetwork 
Libnetwork代码中内置了若干个driver,分别是（bridge,null,host,overlay,remote,windows)，其中bridge,null,host是常用的local driver。    
overlay是docker新发布的multi-host network方案。remote则可以与第三方定制的driver plugin通信。     
参考libnetwork/dirvers/remote/driver.go的函数，最后是发送rest消息与plugin通信。    
<pre><code>...
func (d *driver) CreateNetwork(id string, options map[string]interface{}, ipV4Data, ipV6Data []driverapi.IPAMData) error {
	create := &api.CreateNetworkRequest{
		NetworkID: id,
		Options:   options,
		IPv4Data:  ipV4Data,
		IPv6Data:  ipV6Data,
	}
	return d.call("CreateNetwork", create, &api.CreateNetworkResponse{})
}

func (d *driver) DeleteNetwork(nid string) error {
	delete := &api.DeleteNetworkRequest{NetworkID: nid}
	return d.call("DeleteNetwork", delete, &api.DeleteNetworkResponse{})
}
...
</code></pre>


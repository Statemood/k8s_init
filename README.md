### Kubernetes高可用集群

Kubernetes是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能。

通过Kubernetes你可以：

- 快速部署应用
- 快速扩展应用
- 无缝对接新的应用功能
- 节省资源，优化硬件资源的使用



本工具使用ansible playbook安装k8s高可用集群，并可进行节点扩容。

版本说明：

| 名称       | 版本       |
| ---------- | ---------- |
| kubernetes | 1.10.4     |
| docker     | 18.03.1    |
| system     | CentOS 7.5 |



使用方法：

### 一、修改inventory文件

请按照inventory格式填写

```
#本组内填写etcd服务器及主机名
[etcd]
172.17.14.220    hostname=k8s-etcd-01

#本组内填写master服务器及主机名
[master]
172.17.14.223    hostname=k8s-master-01
172.17.14.224    hostname=k8s-master-02
172.17.14.225    hostname=k8s-master-03

#本组机器不会进行系统初始化等操作，仅用做安装kubectl命令行
[kubectl]
172.17.14.223    hostname=k8s-master-01

#本组机器不会进行系统初始化等操作，只是apiserver证书签发时使用
[k8s_service]
10.64.0.1        #service网段第一个IP
172.17.14.229    #apiserver 负载均衡IP

#本组填写执行playbook的机器IP，不会进行系统初始化等操作
[sslhost]
172.16.70.177

#本组内填写node服务器及主机名
[node]
172.17.14.226   hostname=k8s-node-01
172.17.14.227   hostname=k8s-node-02
172.17.14.228   hostname=k8s-node-03
172.17.14.231   hostname=k8s-node-04
```



### 二、修改相关配置

编辑group_vars/all文件

| 配置项                   | 说明                                                    |
| ------------------------ | ------------------------------------------------------- |
| zbx_server_ip            | 填写zabbix server的IP，系统初始化会自动安装zabbix-agent |
| gpgkey                   | 选择使用vpc内网软件源还是外网软件源                     |
| download_url             | k8s二进制文件下载地址                                   |
| ssl_dir                  | 签发ssl证书保存路径                                     |
| ssl_days                 | 签发ssl的有效期（单位：天）                             |
| apiserver_domain_name    | apiserver域名，签发证书和配置node节点连接master时会用到 |
| service_cluster_ip_range | 指定k8s集群service的网段                                |
| pod_cluster_cidr         | 指定k8s集群pod的网段                                    |
| cluster_dns              | 指定集群dns服务IP                                       |
| harbor                   | 指定harbor镜像仓库地址                                  |



###三、安装方式

1、系统初始化

```
ansible-playbook k8s.yml -i inventory -t init
```

2、签发证书

```
ansible-playbook k8s.yml -i inventory -t cert
```

3、安装etcd

```
ansible-playbook k8s.yml -i inventory -t install_etcd
```

4、安装master节点

```
ansible-playbook k8s.yml -i inventory -t install_master
```

5、安装node节点

```
ansible-playbook k8s.yml -i inventory -t install_node
```

6、全部安装

```
ansible-playbook k8s.yml -i inventory
```

7、扩容mater节点

```
ansible-playbook k8s.yml -i inventory -t init -l master
ansible-playbook k8s.yml -i inventory -t cert,install_master 
```

8、扩容node节点

```
ansible-playbook k8s.yml -i inventory -t init -l node
ansible-playbook k8s.yml -i inventory -t cert,install_node
```

9、替换证书

先删除ca、apiserver证书，然后执行以下步骤

```
ansible-playbook k8s.yml -i inventory -t cert
ansible-playbook k8s.yml -i inventory -t dis_certs,restart_flannel,restart_master,restart_node,restart_etcd
```



### kubernetes HA架构

![k8s](kubernetes.png)
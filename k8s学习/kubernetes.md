[TOC]

# 1. kubernetes介绍

## 1.1 kubernetes简介

![](kubernetes.assets/image-20220310192622236.png)

![image-20220310192749711](kubernetes.assets/image-20220310192749711.png)

## 1.2 组件介绍

一个k8s集群主要是由**控制节点**、**工作节点**构成，每个节点上都会安装不同的组件

![image-20220310193337228](kubernetes.assets/image-20220310193337228.png)

![image-20220310193422371](kubernetes.assets/image-20220310193422371.png)

## 1.3 kubernetes概念

- Master: 集群控制节点，每个集群需要至少一个master节点负责集群的管理。
- Node：工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行。
- Pod：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有一个或者多个容器。
- Controller：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等。
- Service：pod对外服务的统一入口，下面可以维护着同一类的多个pod。
- Label：标签、用于对pod进行分类，同一类pod会拥有相同的标签。
- NameSpace：命名空间，用来隔离pod的运行环境。

# 2. 集群环境搭建

## 2.1 环境规划

### 2.1.1 集群类型

kubernetes集群大体上分为两类：==一主多从和多住多从==。

- 一主多次：一台Master节点和多台Node节点，搭建简单，但是有单机故障风险，适合用于测试环境。
- 多主多从：多台Master节点和多台Node节点，搭建麻烦，安全性高，适合用于生产环境。

### 2.1.2 安装方式

kubernetes有多种部署方式，目前主流的方式有kubeadm、minikube、二进制包

- minikube：一个用于快速搭建单节点kubernetes的工具。
- kubeadm：一个用于快速搭建kubernetes集群的工具。
- 二进制包：从官网下载每个组件的二进制包，依次安装，此方式对于理解kubernetes组件更加有效

### 2.1.3 主机规划

![image-20220310200136833](kubernetes.assets/image-20220310200136833.png)

## 2.2 环境搭建

### 2.2.1 vmware安装

dns服务器可以使用谷歌DNS服务器

- 8.8.8.8
- 8.8.4.4

阿里DNS服务器

- 223.5.5.5
- 223.6.6.6

![image-20220314192541999](kubernetes.assets/image-20220314192541999.png)

> 子网掩码

![image-20220314192700143](kubernetes.assets/image-20220314192700143.png)

![image-20220314192739087](kubernetes.assets/image-20220314192739087.png)

> /etc/sysconfig/network-scripts 中的配置

![image-20220314200058932](kubernetes.assets/image-20220314200058932.png)

### 2.2.2 环境初始化

> 操作系统版本需要7.5以上

![image-20220314192858054](kubernetes.assets/image-20220314192858054.png)

![image-20220315193802388](kubernetes.assets/image-20220315193802388.png)

![image-20220315194137314](kubernetes.assets/image-20220315194137314.png)

![image-20220315194252739](kubernetes.assets/image-20220315194252739.png)

```shell
[root@master ~]# vim /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

[root@master ~]# sysctl -p
[root@master ~]# modprobe br_netfilter
[root@master ~]# lsmod | grep br_netfilter
```

![image-20220315194836872](kubernetes.assets/image-20220315194836872.png)

```shell
cat <<EOF > /etc/sysconfig/modules/ipvs.modules
#!bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

[root@node1 ~]# chmod +x /etc/sysconfig/modules/ipvs.modules
[root@node1 ~]# /bin/bash /etc/sysconfig/modules/ipvs.modules
[root@node1 ~]# lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

![image-20220315200315807](kubernetes.assets/image-20220315200315807.png)

### 2.2.3 安装docker

![image-20220316190925942](kubernetes.assets/image-20220316190925942.png)

![image-20220316192006150](kubernetes.assets/image-20220316192006150.png)



```shell
wget http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

yum list docker-ce --showduplicates

yum install --setopt=obsoletes=0 docker-ce-18.06.3.ce-3.el7 -y

mkdir /etc/docker
cat <<EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://kn0tbca.mirror.aliyuncs.com"]
}
EOF
```

### 2.2.4 安装kubernetes 组件

![image-20220316193709082](kubernetes.assets/image-20220316193709082.png)

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y --nogpgchec --setopt=obsoletes=0 kubeadm-1.17.4-0 kubelet-1.17.4-0 kubectl-1.17.4-0 

cat <<EOF > /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MOOE="ipvs"
EOF

systemctl enable kubelet
```

### 2.2.5 准备集群镜像

![image-20220316195818835](kubernetes.assets/image-20220316195818835.png)

```shell
images=(kube-apiserver:v1.17.17
kube-controller-manager:v1.17.17
kube-scheduler:v1.17.17
kube-proxy:v1.17.17
pause:3.1
etcd:3.4.3-0
coredns:1.6.5
)

for imageName in ${images[@]};do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```

### 2.2.6 集群初始化

![image-20220318171022779](kubernetes.assets/image-20220318171022779.png)

> 支持成功的标志已经node加入执行的命令

![image-20220318172141445](kubernetes.assets/image-20220318172141445.png)

```shell
kubeadm init --kubernetes-version=v1.17.17 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --apiserver-advertise-address=172.16.8.100

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```



![image-20220318171948417](kubernetes.assets/image-20220318171948417.png)

### 2.2.7 安装网络插件

![image-20220318172234739](kubernetes.assets/image-20220318172234739.png)

```shell
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

```yaml
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
  - configMap
  - secret
  - emptyDir
  - hostPath
  allowedHostPaths:
  - pathPrefix: "/etc/cni/net.d"
  - pathPrefix: "/etc/kube-flannel"
  - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN', 'NET_RAW']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unused in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
rules:
- apiGroups: ['extensions']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames: ['psp.flannel.unprivileged']
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni-plugin
       #image: flannelcni/flannel-cni-plugin:v1.0.1 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1
        command:
        - cp
        args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        volumeMounts:
        - name: cni-plugin
          mountPath: /opt/cni/bin
      - name: install-cni
       #image: flannelcni/flannel:v0.17.0 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel:v0.17.0
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
       #image: flannelcni/flannel:v0.17.0 for ppc64le and mips64le (dockerhub limitations may apply)
        image: rancher/mirrored-flannelcni-flannel:v0.17.0
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
        - name: xtables-lock
          mountPath: /run/xtables.lock
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni-plugin
        hostPath:
          path: /opt/cni/bin
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg
      - name: xtables-lock
        hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
```

## 2.3 服务部署

![image-20220318173852001](kubernetes.assets/image-20220318173852001.png)

```shell
kubectl create deployment nginx --image=nginx:1.14-alpine
kubectl expose deployment nginx --port=80 --type=NodePort
```

![image-20220318175829937](kubernetes.assets/image-20220318175829937.png)


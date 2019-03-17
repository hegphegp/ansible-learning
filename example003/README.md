## docker容器模拟客户端


```
# yum install -y git
mkdir -p /opt/soft/ansible && cd /opt/soft/ansible
git clone git clone https://github.com/hegphegp/ansible-learning.git

## 准备docker的离线安装包
mkdir -p /opt/soft/ansible && cd /opt/soft/ansible
docker run -it --rm -v  $(pwd)/ansible-learning/example003/roles/install-docker/files:/docker-packages centos:7.6.1810 sh
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install --downloadonly --downloaddir=/docker-packages docker-ce-18.09.3
exit
```

#### 制作镜像模拟虚拟机
```
##### 制作Dockerfile文件
tee Dockerfile <<-'EOF'
FROM centos:7.6.1810

ENV ROOT_PASSWORD root

RUN curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo ; \
    echo "export LC_ALL=en_US.UTF-8" >> /etc/profile ; \
    yum install -y openssh-server openssh-clients firewalld ; \
    ssh-keygen -A ; \
    echo "root:${ROOT_PASSWORD}" | chpasswd ; \
    sed -i 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config; \
    sed -i 's/#UsePAM no/UsePAM no/g' /etc/ssh/sshd_config; \
    sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config; \
    yum clean all

EXPOSE 22

CMD ["/usr/sbin/init"]
EOF
##### 制作Dockerfile文件结束

docker build -t centos-sshd:7.6.1810 .
```

#### 模拟虚拟机
```
docker pull williamyeh/ansible:alpine3
docker tag williamyeh/ansible:alpine3 ansible-2.5

docker stop web1
docker rm web1
docker stop web2
docker rm web2
docker stop ansible
docker rm ansible
docker network rm ansible-network
docker network create --subnet=10.10.58.0/24 ansible-network
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.100 --name ansible -v /opt/soft/ansible:/ansible ansible-2.5 sh
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.101 --name web1 centos-sshd:7.6.1810 /usr/sbin/init
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.102 --name web2 centos-sshd:7.6.1810 /usr/sbin/init
docker run --privileged -itd --restart always --net ansible-network --ip 10.10.58.103 --name web3 centos-sshd:7.6.1810 /usr/sbin/init
```
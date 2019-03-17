

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

```
yum install -y git

```
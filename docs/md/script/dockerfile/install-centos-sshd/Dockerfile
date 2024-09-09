FROM centos:7
RUN mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
COPY ./CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo
COPY ./epel.repo /etc/yum.repos.d/epel.repo
RUN yum makecache
RUN yum install -y openssh-server vim lrzsz wget gcc-c++ pcre pcre-devel zlib zlib-devel ruby openssl openssl-devel patch bash-completion zlib.i686 libstdc++.i686 lsof unzip zip
RUN yum install net-tools -y
#RUN systemctl status sshd
#RUN systemctl restart sshd
RUN whoami
RUN  echo root|passwd --stdin root
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]


FROM centos:7

RUN yum update -y && \
    yum -y install ncurses-6.1-7.20180224.el8.x86_64 vim vim-enhanced wget openssh-server openssh-clients coreutils-single-8.30-6.el8.x86_64 && \ 
    yum -y install epel-release wget && \
    yum -y install python3-pip

RUN ssh-keygen -f /etc/ssh/ssh_host_rsa_key -N '' -t rsa

RUN ssh-keygen -f /etc/ssh/ssh_host_ed25519_key -N '' -t ed25519

EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]

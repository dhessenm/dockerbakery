FROM ubuntu:14.04
MAINTAINER Dirk Hessenmueller

# Install Ansible from source (master)
RUN apt-get -y update && \
    apt-get install -y software-properties-common openssh-server sudo curl git python-pip python-dev libffi-dev libssl-dev && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN pip install awscli && \
    pip install boto pyyaml pytz jinja2

RUN pip install shade

# sshd
RUN mkdir /var/run/sshd

# create ansible user
RUN groupadd -g 1001 ansible && \
    mkdir /app && \
    useradd -u 1001 -g 1001 -s /bin/bash -c "ansible user" -d /app/ansible -m ansible && \
    echo "ansible:ansible" | chpasswd

# setup ssh directory
RUN mkdir /app/ansible/.ssh && \
    chown ansible:ansible /app/ansible/.ssh && \
    chmod 700 /app/ansible/.ssh

ADD ansible.cfg /app/ansible/.ansible.cfg
ADD hosts /app/ansible/hosts
ADD proxy /app/ansible/proxy

RUN chown ansible:ansible /app/ansible/.ansible.cfg && \
    chown ansible:ansible /app/ansible/hosts && \
    chown ansible:ansible /app/ansible/proxy

# Add AWS config file
RUN mkdir /app/ansible/.aws && \
    chown ansible:ansible /app/ansible/.aws && \
    chmod 700 /app/ansible/.aws

ADD config /app/ansible/.aws/config
RUN chown ansible:ansible /app/ansible/.aws/config

# set sudo for user ansible
RUN echo '%ansible ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# set timezone
ENV TZ=Europe/Berlin
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# install ansible devel
RUN git clone http://github.com/ansible/ansible.git --recursive /opt/ansible

ADD source_ansible /app/ansible/source_ansible 
RUN chown ansible:ansible /app/ansible/source_ansible && chgrp -R ansible /opt/ansible
RUN /bin/bash /app/ansible/source_ansible

RUN echo "http_proxy=http_proxy://zst-100v.linux.rz.db.de:8888" >> /etc/environment && \
    echo "https_proxy=http_proxy://zst-100v.linux.rz.db.de:8888" >> /etc/environment && \
    echo "HTTP_PROXY=http_proxy://zst-100v.linux.rz.db.de:8888" >> /etc/environment && \
    echo "HTTPS_PROXY=http_proxy://zst-100v.linux.rz.db.de:8888" >> /etc/environment && \
    echo "no_proxy=localhost,127.0.0.1,.db.de" >> /etc/environment && \
    echo "NO_PROXY=localhost,127.0.0.1,.db.de" >> /etc/environment

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]


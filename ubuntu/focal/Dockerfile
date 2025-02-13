# syntax = docker/dockerfile:1.13.0
FROM ubuntu:focal-20241011
LABEL maintainer="Alex Alonso"

ENV pip_packages "ansible"
ENV DEBIAN_FRONTEND "noninteractive"

ADD https://bootstrap.pypa.io/get-pip.py get-pip.py
# Install dependencies.
RUN <<EOR
  apt-get update
  apt-get install -y --no-install-recommends software-properties-common
  add-apt-repository -y ppa:deadsnakes/ppa
  apt-get update
  apt-get install -y --no-install-recommends \
    apt-utils \
    locales \
    python3.13 \
    rsyslog systemd systemd-cron sudo iproute2
  python3.13 get-pip.py
  apt-get upgrade -y
  rm -Rf /var/lib/apt/lists/*
  rm -Rf /usr/share/doc && rm -Rf /usr/share/man
  apt-get clean
EOR

RUN sed -i 's/^\($ModLoad imklog\)/#\1/' /etc/rsyslog.conf

# Fix potential UTF-8 errors with ansible-test.
RUN locale-gen en_US.UTF-8

# Install Ansible via Pip.
RUN python3.13 -m pip install --no-cache-dir $pip_packages

COPY initctl_faker .
RUN chmod +x initctl_faker && rm -fr /sbin/initctl && ln -s /initctl_faker /sbin/initctl

# Install Ansible inventory file.
RUN mkdir -p /etc/ansible
RUN printf "[local]\nlocalhost ansible_connection=local" > /etc/ansible/hosts

# Remove unnecessary getty and udev targets that result in high CPU usage when using
# multiple containers with Molecule (https://github.com/ansible/molecule/issues/1104)
RUN <<EOR
  rm -f /lib/systemd/system/systemd*udev*
  rm -f /lib/systemd/system/getty.target
EOR

# Create `ansible` user with sudo permissions
ENV ANSIBLE_USER=ansible SUDO_GROUP=sudo
RUN set -xe \
  && groupadd -r ${ANSIBLE_USER} \
  && useradd -m -g ${ANSIBLE_USER} ${ANSIBLE_USER} \
  && usermod -aG ${SUDO_GROUP} ${ANSIBLE_USER} \
  && sed -i "/^%${SUDO_GROUP}/s/ALL\$/NOPASSWD:ALL/g" /etc/sudoers

VOLUME ["/sys/fs/cgroup", "/tmp", "/run"]
CMD ["/lib/systemd/systemd"]
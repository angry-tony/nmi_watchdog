```
sudo sh -c 'systemctl kill -s SIGUSR1 dcos-mesos-master && systemctl stop dcos-mesos-master'; \
rm -f /var/lib/dcos/mesos-resources; \
systemctl start dcos-mesos-master

sudo sh -c 'systemctl kill -s SIGUSR1 dcos-mesos-slave && systemctl stop dcos-mesos-slave'; \
rm -rf /var/lib/mesos/slave/meta/slaves/latest/; \
systemctl restart dcos-mesos-slave

sudo sh -c 'systemctl kill -s SIGUSR1 dcos-mesos-slave-public && systemctl stop dcos-mesos-slave-public'; \
rm -rf /var/lib/mesos/slave/meta/slaves/latest/; \
systemctl restart dcos-mesos-slave-public
```

```
sudo tee /etc/docker/daemon.json << 'EOF'
{
    "storage-driver": "overlay",
    "insecure-registries": [ "ldap-nfs-pdr-01", "192.168.147.101" ],
    "dns": [ "16.110.135.51", "16.110.135.52" ]
}
EOF
```

```
tee /etc/mesosphere/docker_credentials << 'EOF'
{
        "auths": {
                "pdr-01:5000": {
                        "auth": "dGVzdDoxMjM="
                }
        }
}
EOF

/opt/mesosphere/etc/mesos-slave-common:
MESOS_DOCKER_REGISTRY=http://nas.ajway.kr:6000

MESOS_DOCKER_CONFIG=file:///etc/mesosphere/docker_credentials
```

```
for i in dcos-master-1 dcos-slave-1 dcos-pub-slave-01; do ssh $i "rm -rf /etc/pki/ca-trust/source/anchors/*"& done
for i in dcos-master-1 dcos-slave-1 dcos-pub-slave-01; do ssh $i "ls -l  /etc/pki/ca-trust/source/anchors/*"; done
for i in dcos-master-1 dcos-slave-1 dcos-pub-slave-01; do ssh $i "update-ca-trust force-enable && update-ca-trust enable && update-ca-trust extract"& done


for i in dcos-master-1 dcos-slave-1 dcos-pub-slave-01; do ssh $i "cp /opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem /opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem.BAK"; done

for i in dcos-master-1 dcos-slave-1 dcos-pub-slave-01; do ssh $i "rm -rf /opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem"; done
for i in dcos-master-1 dcos-slave-1 dcos-pub-slave-01; do ssh $i "cp /opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem.BAK /opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem"; done
for i in dcos-master-1 dcos-slave-1 dcos-pub-slave-01; do cat /etc/pki/ca-trust/source/anchors/portus-CA.crt | ssh $i "cat >> /opt/mesosphere/active/python-requests/lib/python3.5/site-packages/requests/cacert.pem"; done
```

...


SSH_SUDO
On first run the SSH user is created with a the sudo rule ALL=(ALL) ALL which allows the user to run all commands but a password is required. If you want to limit the access to specific commands or allow sudo without a password prompt SSH_SUDO can be used.
...
  --env "SSH_SUDO=ALL=(ALL) NOPASSWD:ALL" \
...


SSH_USER
On first run the SSH user is created with the default username of "app-admin". If you require an alternative username SSH_USER can be used when running the container.
...
  --env "SSH_USER=app-1" \
...
SSH_USER_PASSWORD
On first run the SSH user is created with a generated password. If you require a specific password SSH_USER_PASSWORD can be used when running the container. If set to an empty string then a password is auto-generated and, if SSH_SUDO is not set to allow no password for all commands, will be displayed in the docker logs.
...
  --env "SSH_USER_PASSWORD=Passw0rd!" \
...


git clone https://github.com/jdeathe/centos-ssh.git; cd centos-ssh/

mkdir -p ./ssh-key; ssh-keygen -t rsa -f ./ssh-key/id_rsa -q -P ""; ls ssh-key/
id_rsa  id_rsa.pub

[root@bootstrap-01 centos-ssh]# cp ssh-key/id_rsa.pub src/etc/services-config/ssh/authorized_keys
cp: overwrite ‘src/etc/services-config/ssh/authorized_keys’? y

tee src/etc/resolv.conf <<-EOF
nameserver 8.8.8.8
EOF


tee src/etc/services-config/supervisor/supervisord.d/nslcd.conf <<-EOF
[program:nslcd]
command=/usr/sbin/nslcd
autostart=true
autorestart=unexpected
startsecs=0
startretries=0
priority=1
redirect_stderr = true
stdout_logfile = /var/log/nslcd.log
stdout_events_enabled = false
EOF

[root@bootstrap-01 centos-ssh]# tee Dockerfile <<-'IMAGE_EOF'
# ========================================================================
# jdeathe/centos-ssh
#
# CentOS-7 7.3.1611 x86_64 - SCL/EPEL/IUS Repos. / Supervisor / OpenSSH.
#
# ========================================================================
FROM centos:7.3.1611

# -----------------------------------------------------------------------------
# Base Install + Import the RPM GPG keys for Repositories
# -----------------------------------------------------------------------------
RUN rpm --rebuilddb \
        && rpm --import \
                http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7 \
        && rpm --import \
                https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7 \
        && rpm --import \
                https://dl.iuscommunity.org/pub/ius/IUS-COMMUNITY-GPG-KEY \
        && yum -y install \
                        --setopt=tsflags=nodocs \
                        --disableplugin=fastestmirror \
                centos-release-scl \
                centos-release-scl-rh \
                epel-release \
                https://centos7.iuscommunity.org/ius-release.rpm \
                openssh-6.6.1p1-35.el7_3 \
                openssh-server-6.6.1p1-35.el7_3 \
                openssh-clients-6.6.1p1-35.el7_3 \
                openssl-1.0.1e-60.el7 \
                python-setuptools-0.9.8-4.el7 \
                sudo-1.8.6p7-23.el7_3 \
                vim-minimal-7.4.160-1.el7_3.1 \
                yum-plugin-versionlock-1.1.31-40.el7 \
                xz-5.2.2-1.el7 \
                net-tools \
                iproute \
                dstat \
                authconfig \
                openldap-clients \
                nss-pam-ldapd \
                pam_ldap \
                nc \
        && yum versionlock add \
                openssh \
                openssh-server \
                openssh-clients \
                python-setuptools \
                sudo \
                vim-minimal \
                yum-plugin-versionlock \
                xz \
        && yum clean all \
        && rm -rf /etc/ld.so.cache \
        && rm -rf /sbin/sln \
        && rm -rf /usr/{{lib,share}/locale,share/{man,doc,info,cracklib,i18n},{lib,lib64}/gconv,bin/localedef,sbin/build-locale-archive} \
        && rm -rf /{root,tmp,var/cache/{ldconfig,yum}}/* \
        && > /etc/sysconfig/i18n \
        && authconfig --enableforcelegacy --update \
        && authconfig --enableldap --enableldapauth --ldapserver="192.168.147.150:389" --ldapbasedn="dc=example,dc=com" --update \
        && rm -rf /var/run/nslcd/*

# /etc/nslcd.conf && /etc/openldap/ldap.conf


# -----------------------------------------------------------------------------
# Install supervisord (required to run more than a single process in a container)
# Note: EPEL package lacks /usr/bin/pidproxy
# We require supervisor-stdout to allow output of services started by
# supervisord to be easily inspected with "docker logs".
# -----------------------------------------------------------------------------
RUN easy_install \
                'supervisor == 3.3.3' \
                'supervisor-stdout == 0.1.1' \
        && mkdir -p \
                /var/log/supervisor/

# -----------------------------------------------------------------------------
# UTC Timezone & Networking
# -----------------------------------------------------------------------------
RUN ln -sf \
                /usr/share/zoneinfo/Asia/Seoul \
                /etc/localtime \
        && echo "NETWORKING=yes" > /etc/sysconfig/network

# -----------------------------------------------------------------------------
# Configure SSH for non-root public key authentication
# -----------------------------------------------------------------------------
RUN sed -i \
        -e 's~^PasswordAuthentication yes~PasswordAuthentication no~g' \
        -e 's~^#PermitRootLogin yes~PermitRootLogin no~g' \
        -e 's~^#UseDNS yes~UseDNS no~g' \
        -e 's~^\(.*\)/usr/libexec/openssh/sftp-server$~\1internal-sftp~g' \
        -e 's~^HostKey /etc/ssh/ssh_host_ecdsa_key$~#HostKey /etc/ssh/ssh_host_ecdsa_key~g' \
        -e 's~^HostKey /etc/ssh/ssh_host_ed25519_key$~#HostKey /etc/ssh/ssh_host_ed25519_key~g' \
        /etc/ssh/sshd_config

#         -e 's~^PasswordAuthentication no~PasswordAuthentication yes~g' \

# -----------------------------------------------------------------------------
# Enable the wheel sudoers group
# -----------------------------------------------------------------------------
RUN sed -i \
        -e 's~^# %wheel\tALL=(ALL)\tALL~%wheel\tALL=(ALL) ALL~g' \
        -e 's~\(.*\) requiretty$~#\1requiretty~' \
        /etc/sudoers

# -----------------------------------------------------------------------------
# Copy files into place
# -----------------------------------------------------------------------------
ADD src/usr/bin \
        /usr/bin/
ADD src/usr/sbin \
        /usr/sbin/
ADD src/opt/scmi \
        /opt/scmi/
ADD src/etc/systemd/system \
        /etc/systemd/system/
ADD src/etc/services-config/ssh/authorized_keys \
        src/etc/services-config/ssh/sshd-bootstrap.conf \
        src/etc/services-config/ssh/sshd-bootstrap.env \
        /etc/services-config/ssh/
ADD src/etc/services-config/supervisor/supervisord.conf \
        /etc/services-config/supervisor/
# add service-config-files which is what you wnat to run in container
ADD src/etc/services-config/supervisor/supervisord.d \
        /etc/services-config/supervisor/supervisord.d/
ADD src/etc/resolv.conf \
       /etc/

RUN mkdir -p \
                /etc/supervisord.d/ \
        && cp -pf \
                /etc/ssh/sshd_config \
                /etc/services-config/ssh/ \
        && ln -sf \
                /etc/services-config/ssh/sshd_config \
                /etc/ssh/sshd_config \
        && ln -sf \
                /etc/services-config/ssh/sshd-bootstrap.conf \
                /etc/sshd-bootstrap.conf \
        && ln -sf \
                /etc/services-config/ssh/sshd-bootstrap.env \
                /etc/sshd-bootstrap.env \
        && ln -sf \
                /etc/services-config/supervisor/supervisord.conf \
                /etc/supervisord.conf \
        && ln -sf \
                /etc/services-config/supervisor/supervisord.d/sshd-wrapper.conf \
                /etc/supervisord.d/sshd-wrapper.conf \
        && ln -sf \
                /etc/services-config/supervisor/supervisord.d/sshd-bootstrap.conf \
                /etc/supervisord.d/sshd-bootstrap.conf \
        && ln -sf \
                /etc/services-config/supervisor/supervisord.d/nslcd.conf \
                /etc/supervisord.d/nslcd.conf \
        && chmod 700 \
                /usr/{bin/healthcheck,sbin/{scmi,sshd-{bootstrap,wrapper}}}

EXPOSE 22

# -----------------------------------------------------------------------------
# Set default environment variables
# -----------------------------------------------------------------------------
ENV SSH_AUTHORIZED_KEYS="" \
        SSH_AUTOSTART_SSHD=true \
        SSH_AUTOSTART_SSHD_BOOTSTRAP=true \
        SSH_CHROOT_DIRECTORY="%h" \
        SSH_INHERIT_ENVIRONMENT=false \
        SSH_SUDO="ALL=(ALL) ALL" \
        SSH_USER="app-admin" \
        SSH_USER_FORCE_SFTP=false \
        SSH_USER_HOME="/home/%u" \
        SSH_USER_ID="500:500" \
        SSH_USER_PASSWORD="" \
        SSH_USER_PASSWORD_HASHED=false \
        SSH_USER_SHELL="/bin/bash"

# -----------------------------------------------------------------------------
# Set image metadata
# -----------------------------------------------------------------------------
ARG RELEASE_VERSION="2.2.3"
LABEL \
        maintainer="James Deathe <james.deathe@gmail.com>" \
        install="docker run \
--rm \
--privileged \
--volume /:/media/root \
jdeathe/centos-ssh:${RELEASE_VERSION} \
/usr/sbin/scmi install \
--chroot=/media/root \
--name=\${NAME} \
--tag=${RELEASE_VERSION} \
--setopt='--volume {{NAME}}.config-ssh:/etc/ssh'" \
        uninstall="docker run \
--rm \
--privileged \
--volume /:/media/root \
jdeathe/centos-ssh:${RELEASE_VERSION} \
/usr/sbin/scmi uninstall \
--chroot=/media/root \
--name=\${NAME} \
--tag=${RELEASE_VERSION} \
--setopt='--volume {{NAME}}.config-ssh:/etc/ssh'" \
        org.deathe.name="centos-ssh" \
        org.deathe.version="${RELEASE_VERSION}" \
        org.deathe.release="jdeathe/centos-ssh:${RELEASE_VERSION}" \
        org.deathe.license="MIT" \
        org.deathe.vendor="jdeathe" \
        org.deathe.url="https://github.com/jdeathe/centos-ssh" \
        org.deathe.description="CentOS-7 7.3.1611 x86_64 - SCL, EPEL and IUS Repositories / Supervisor / OpenSSH."

HEALTHCHECK \
        --interval=0.5s \
        --timeout=1s \
        --retries=5 \
        CMD ["/usr/bin/healthcheck"]

CMD ["/usr/bin/supervisord", "--configuration=/etc/supervisord.conf"]
IMAGE_EOF



docker build --tag centos-ssh-custom:v1.0 .

docker images
REPOSITORY              TAG              IMAGE ID                CREATED             SIZE
centos-ssh-custom        v1.0              2fe766e6c31c         5 seconds ago       263 MB
centos                           7.3.1611        67591570dd29        8 months ago        192 MB


mkdir -p /home/mozart

chown -R mozart:mozart /home/mozart/
chown: invalid group: ‘mozart:mozart’

chown -R mozart /home/mozart/


docker run -d -v /home/mozart:/home/mozart:rw --name ssh-test -p 2020:22 centos-ssh-custom:v1.0
a52a53d6747dd4255b1439ea00c91339f0483dc41eeda442af680c1719b60106

docker ps -a
CONTAINER ID        IMAGE                            COMMAND                  CREATED          STATUS                            PORTS                  NAMES
a52a53d6747d        centos-ssh-custom:v1.0   "/usr/bin/supervis..."     3 seconds ago       Up 2 seconds (health: starting)   0.0.0.0:2020->22/tcp   ssh-test


docker logs ssh-test
2017-09-06 14:53:48,483 CRIT Supervisor running as root (no user in config file)
2017-09-06 14:53:48,484 WARN No file matches via include "/etc/supervisord.d/*.ini"
2017-09-06 14:53:48,484 INFO Included extra file "/etc/supervisord.d/nslcd.conf" during parsing
2017-09-06 14:53:48,484 INFO Included extra file "/etc/supervisord.d/sshd-bootstrap.conf" during parsing
2017-09-06 14:53:48,484 INFO Included extra file "/etc/supervisord.d/sshd-wrapper.conf" during parsing
2017-09-06 14:53:48,487 INFO supervisord started with pid 1
2017-09-06 14:53:49,494 INFO spawned: 'supervisor_stdout' with pid 17
2017-09-06 14:53:49,497 INFO spawned: 'nslcd' with pid 18
2017-09-06 14:53:49,509 INFO spawned: 'sshd-bootstrap' with pid 19
2017-09-06 14:53:49,512 INFO spawned: 'sshd-wrapper' with pid 20
2017-09-06 14:53:49,624 INFO success: supervisor_stdout entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
2017-09-06 14:53:49,624 INFO success: nslcd entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
2017-09-06 14:53:49,625 INFO success: sshd-bootstrap entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
2017-09-06 14:53:49,625 INFO success: sshd-wrapper entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
2017-09-06 14:53:49,625 INFO exited: nslcd (exit status 0; expected)

sshd-bootstrap stdout | Initialising SSH.
sshd-bootstrap stdout |
==================================================================
SSH Details
--------------------------------------------------------------------------------
user : app-admin
password : fSYka4EJTHaU0Dck
id : 500:500
home : /home/app-admin
chroot path : N/A
shell : /bin/bash
sudo : ALL=(ALL) ALL
key fingerprints :
bb:02:51:c1:ee:54:c8:d2:ca:78:d4:c1:6e:d4:c9:46
rsa host key fingerprint :
d0:a5:63:4e:69:55:f0:12:d1:a5:87:6e:5d:f6:46:28
--------------------------------------------------------------------------------
4.40131

2017-09-06 13:04:43,178 INFO exited: sshd-bootstrap (exit status 0; expected)


docker port ssh-test 22
0.0.0.0:2020


ssh -p 2020 -i ./ssh-key/id_rsa app-admin@192.168.148.150 -o StrictHostKeyChecking=no
Warning: Permanently added '[192.168.148.150]:2020' (RSA) to the list of known hosts.
[app-admin@a52a53d6747d ~]$

[app-admin@a52a53d6747d ~]$ id
uid=500(app-admin) gid=500(app-admin) groups=500(app-admin),10(wheel),100(users)

[app-admin@a52a53d6747d ~]$ exit
Logout
Connection to 192.168.148.150 closed.

app-admin $] getent passwd |grep mozart
mozart:x:10003:10003:Wolfgang Amadeus Mozart:/home/mozart:/bin/bash

app-admin $] exit



## Host-node 에 autofs bind 설정 (GPFS 로 이미 마운트 되어 있음)
Host-node ~]# /etc/auto.master <<-EOF
/rhome  /etc/auto.misc --timeout=60     # for bind fs
EOF

Host-node ~]# /etc/auto.misc <<-EOF
*               -fstype=bind    :/nas/&   # for bind fs
EOF

-v /rhome:/home:shared



####################

Host-node ~]# /etc/auto.master <<-EOF
/home    /etc/auto.misc  --timeout=60  --ghost
#/rhome  /etc/auto.misc --timeout=60     # for bind fs
EOF

Host-node ~]# /etc/auto.misc <<-EOF
*   -rw,soft,intr    NFS_SERVER_IP_ADDR:/home/&
#*               -fstype=bind    :/nas/&   # for bind fs
EOF

	# docker run -d -v /rhome:/home:shared --name ssh-test -p 2020:22 centos-ssh-custom:v1.0

tee src/etc/services-config/supervisor/supervisord.d/autofs.conf <<-EOF
[program:autofs]
command=/usr/sbin/automount -t 0 -f /etc/auto.master
priority=2
numprocs=1
autostart=true
autorestart=true
startsecs=0
startretries=0
#redirect_stderr = true
#stdout_logfile=/var/log/supervisor/autofs.log
stdout_events_enabled = false
EOF

	# in Dockerfils
..
ADD src/etc/auto.master \
       /etc/
ADD src/etc/auto.misc \
       /etc/
..
        && ln -sf \
                /etc/services-config/supervisor/supervisord.d/autofs.conf \
                /etc/supervisord.d/autofs.conf \

## 중요
https://scriptthe.net/2015/02/05/autofs-in-docker-containers/

On all systems, you will need to run your container in --privileged mode!!! If you don't, you'll see the likes of:
/usr/sbin/automount: test mount forbidden or incorrect kernel protocol version, kernel protocol version 5.00 or above required.




# UCR
@ DCOS docker 정보
[root@master-01 ~]# cat /opt/mesosphere/etc/mesos-slave-common |grep MESOS_DOCKER_
MESOS_DOCKER_VOLUME_CHECKPOINT_DIR=/var/lib/mesos/isolators/docker/volume
MESOS_DOCKER_REMOVE_DELAY=1hrs
MESOS_DOCKER_STOP_TIMEOUT=20secs
MESOS_DOCKER_STORE_DIR=/var/lib/mesos/slave/store/docker
MESOS_DOCKER_REGISTRY=https://ldap-nfs-pdr-01.dcos
MESOS_DOCKER_CONFIG=file:///etc/mesosphere/docker_credentials

@ DCOS 기본 docker credential 계정 : test (이 계정으로 images pull 할 수 없음)
[root@master-01 ~]# cat /etc/mesosphere/docker_credentials  # test/a 계정임
{
        "auths": {
                "ldap-nfs-pdr-01.dcos": {
                        "auth": "dGVzdDoxMjM="
                }
        }
}

@ image 올린 계정 : mozart
[root@master-01 ~]# getent passwd |grep mozart
mozart:x:10003:10003:Wolfgang Amadeus Mozart:/home/mozart:/bin/bash

[root@bootstrap-01 centos-ssh]# cat ~/.docker/config.json
{
        "auths": {
                "ldap-nfs-pdr-01.dcos": {
                        "auth": "bW96YXJ0OnBhc3N3b3Jk"
                }
        }
}

[root@bootstrap-01 centos-ssh]# docker push ldap-nfs-pdr-01.dcos/mozart/centos-ssh:v1.0

@ mozart 계정 홈디렉토리 만들기 : 컨테이너가 private slave node 에 실행되므로 pri-slave-01 에서 아래 작업 수행
[root@pri-slave-01 ~]# mkdir -p -m 700 /home/mozart

[root@pri-slave-01 ~]# cp /etc/skel/.bash* /home/mozart/

[root@pri-slave-01 ~]# chown -R mozart:mozart /home/mozart/
chown: invalid group: ‘mozart:mozart’

[root@pri-slave-01 ~]# chown -R mozart /home/mozart/

@ mozart 로 컨테이너 띄울수 있게 permmision 주기 : native marathon 에서 수행하므로
Organization > Service Accounts > dcos_marathon
	dcos:mesos:master:task:user:mozart create

@ UCR 에 사용할 docker credendial 만들기
[ -d /usr/local/bin ] || sudo mkdir -p /usr/local/bin && 
curl https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.10/dcos -o dcos && 
sudo mv dcos /usr/local/bin && 
sudo chmod +x /usr/local/bin/dcos && 
dcos cluster setup https://192.168.147.102 && 
dcos

dcos config show core.ssl_verify
/root/.dcos/clusters/9983e313-1d80-4252-baa3-ab4b05d738cc/dcos_ca.crt

dcos package install --cli dcos-enterprise-cli

#curl -k -v $(dcos config show core.dcos_url)/ca/dcos-ca.crt -o /tmp/dcos-ca.crt
#dcos config set core.ssl_verify /tmp/dcos-ca.crt

docker login ldap-nfs-pdr-01.dcos
username : mozart
password : password

cat .docker/config.json
{
        "auths": {
                "ldap-nfs-pdr-01.dcos": {
                        "auth": "bW96YXJ0OnBhc3N3b3Jk"
                }
        }
}

dcos security secrets create --value-file=/root/.docker/config.json dcosdocker/mozart-pullConfig
	# https://docs.mesosphere.com/1.9/security/secrets/create-secrets/
	# https://docs.mesosphere.com/1.10/deploying-services/private-docker-registry/
	# secrets.0: Secret dcosdocker/mozart-pullConfig is not accessible
	# 에러가 남.. 퍼미션도 드가지가 않음







{
  "id": "/test",
  "backoffFactor": 1.15,
  "backoffSeconds": 1,
  "cmd": "env; id; ip -o addr; cat /proc/net/fib_trie; df -h; /usr/bin/supervisord --configuration=/etc/supervisord.conf; sleep 30000",
  "container": {
    "type": "MESOS",
    "volumes": [
      {
        "containerPath": "/home/mozart",
        "hostPath": "/home/mozart",
        "mode": "RW"
      }
    ],
    "docker": {
      "image": "mozart/centos-ssh:v1.0",
      "forcePullImage": false,
      "parameters": [],
      "pullConfig": {
        "secret": "secret0"
      }
    }
  },
  "cpus": 0.1,
  "disk": 0,
  "instances": 0,
  "maxLaunchDelaySeconds": 3600,
  "mem": 128,
  "gpus": 0,
  "networks": [
    {
      "name": "dcos",
      "mode": "container"
    }
  ],
  "requirePorts": false,
  "upgradeStrategy": {
    "maximumOverCapacity": 1,
    "minimumHealthCapacity": 1
  },
  "user": "mozart",
  "killSelection": "YOUNGEST_FIRST",
  "unreachableStrategy": {
    "inactiveAfterSeconds": 300,
    "expungeAfterSeconds": 600
  },
  "healthChecks": [],
  "fetch": [],
  "constraints": [],
  "secrets": {
    "secret0": {
      "source": "dcosdocker/mozart-pullConfig"
    }
  },
  "env": {
    "pullConfig": {
      "secret": "secret0"
    }
  }
}






@ secret 없이 아래 것으로 수행.
{
  "id": "/test",
  "instances": 1,
  "user": "mozart",
  "container": {
    "type": "MESOS",
    "volumes": [
      {
        "containerPath": "/home/mozart",
        "hostPath": "/home/mozart",
        "mode": "RW"
      }
    ],
    "docker": {
      "image": "ldap-nfs-pdr-01.dcos/mozart/centos-ssh:v1.0"
    }
  },
  "cpus": 0.1,
  "mem": 128,
  "requirePorts": true,
  "cmd": "env; ip -o addr; cat /proc/net/fib_trie; df -h; /usr/bin/supervisord --configuration=/etc/supervisord.conf; sleep 30000",
  "ipAddress": {
    "groups": [],
    "networkName": "dcos"
  },
  "portDefinitions": [],
  "networks": [],
  "healthChecks": [],
  "fetch": [],
  "constraints": []
}


## error
Traceback (most recent call last):
  File "/usr/lib64/python2.7/site.py", line 556, in <module>
    main()
  File "/usr/lib64/python2.7/site.py", line 538, in main
    known_paths = addusersitepackages(known_paths)
  File "/usr/lib64/python2.7/site.py", line 266, in addusersitepackages
    user_site = getusersitepackages()
  File "/usr/lib64/python2.7/site.py", line 241, in getusersitepackages
    user_base = getuserbase() # this will also set USER_BASE
  File "/usr/lib64/python2.7/site.py", line 231, in getuserbase
    USER_BASE = get_config_var('userbase')
  File "/usr/lib64/python2.7/sysconfig.py", line 516, in get_config_var
    return get_config_vars().get(name)
  File "/usr/lib64/python2.7/sysconfig.py", line 473, in get_config_vars
    _CONFIG_VARS['userbase'] = _getuserbase()
  File "/usr/lib64/python2.7/sysconfig.py", line 187, in _getuserbase
    return env_base if env_base else joinuser("~", ".local")
  File "/usr/lib64/python2.7/sysconfig.py", line 173, in joinuser
    return os.path.expanduser(os.path.join(*args))
  File "/usr/lib64/python2.7/posixpath.py", line 269, in expanduser
    userhome = pwd.getpwuid(os.getuid()).pw_dir
KeyError: 'getpwuid(): uid not found: 10003'

	계정을 passwd 파일에 주면 실행은 되나 파일퍼미션이 없음.
IOError: [Errno 13] Permission denied: '/var/log/supervisor/supervisord.log'


2017-09-14 00:43:36,487 WARN No file matches via include "/etc/supervisord.d/*.ini"
2017-09-14 00:43:36,487 INFO Included extra file "/etc/supervisord.d/nslcd.conf" during parsing
2017-09-14 00:43:36,487 INFO Included extra file "/etc/supervisord.d/sshd-bootstrap.conf" during parsing
2017-09-14 00:43:36,487 INFO Included extra file "/etc/supervisord.d/sshd-wrapper.conf" during parsing
2017-09-14 00:43:36,490 CRIT could not write pidfile /var/run/supervisord.pid
2017-09-14 00:43:37,492 INFO spawned: 'supervisor_stdout' with pid 24
2017-09-14 00:43:37,494 INFO spawnerr: unknown error making dispatchers for 'nslcd': EACCES
2017-09-14 00:43:37,494 INFO spawnerr: no permission to run command '/usr/sbin/sshd-bootstrap'
2017-09-14 00:43:37,494 INFO spawnerr: no permission to run command '/usr/sbin/sshd-wrapper'
2017-09-14 00:43:37,785 INFO success: supervisor_stdout entered RUNNING state, process has stayed up for > than 0 seconds (startsecs)
2017-09-14 00:43:37,785 INFO gave up: nslcd entered FATAL state, too many start retries too quickly
2017-09-14 00:43:37,786 INFO gave up: sshd-bootstrap entered FATAL state, too many start retries too quickly
2017-09-14 00:43:38,788 INFO spawnerr: no permission to run command '/usr/sbin/sshd-wrapper'
2017-09-14 00:43:40,791 INFO spawnerr: no permission to run command '/usr/sbin/sshd-wrapper'
2017-09-14 00:43:43,796 INFO spawnerr: no permission to run command '/usr/sbin/sshd-wrapper'
2017-09-14 00:43:43,796 INFO gave up: sshd-wrapper entered FATAL state, too many start retries too quickly

@ Dockerfile 에 계정 및 퍼미션 추가 및 마지막 에러에 있는 것들 수정해야함..
...
        && rm -rf /var/run/nslcd/* \
        && echo "mozart:x:10003:10003:Wolfgang Amadeus Mozart:/home/mozart:/bin/bash" >> /etc/passwd
...
...
        && mkdir -p \
                /var/log/supervisor/ \
        && chmod 777 /var/log/supervisor
...
...





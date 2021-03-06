# hadoop2 on docker-centos
# 
# docker build -t SPEC_IMAGE_NAME .

FROM centos:centos6
MAINTAINER Winse <fuqiuliu2006@qq.com>

RUN yum install -y which nc wget curl tar rsync openssh-clients openssh-server expect

RUN ssh-keygen -P '' -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN ssh-keygen -P '' -t dsa -f /etc/ssh/ssh_host_dsa_key

RUN su root -c "ssh-keygen -P '' -t rsa -f ~/.ssh/id_rsa"
# ssh-copy-id not work, sshd isnot started!
RUN su root -c "cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys"

RUN sed -i '/pam_loginuid.so/c session    optional     pam_loginuid.so'  /etc/pam.d/sshd

RUN echo 'root:hadoop' | chpasswd 

RUN groupadd hadoop \
  && useradd -g hadoop hadoop \
  && echo 'hadoop:hadoop' | chpasswd

RUN chown hadoop:hadoop /opt

# JDK

#-- RUN curl -LO 'http://download.oracle.com/otn-pub/java/jdk/7u67-b01/jdk-7u67-linux-x64.tar.gz' -H 'Cookie: oraclelicense=accept-securebackup-cookie' 
RUN curl -s http://172.17.42.1:8080/hadoop/jdk-7u67-linux-x64.tar.gz \
        | tar zx --exclude=src.zip --exclude=db/** --exclude=missioncontrol/** --exclude=visualvm/** --exclude=desktop/** --exclude=man/** -C /opt/

# HADOOP

#-- RUN wget http://apache.fayea.com/apache-mirror/hadoop/common/hadoop-2.5.1/hadoop-2.5.1.tar.gz 
RUN su hadoop -c \
        "curl -s http://172.17.42.1:8080/hadoop/hadoop-2.5.1.tar.gz | tar zx --exclude=sources/* --exclude=doc/** --exclude=*.cmd -C /opt/"
#--
RUN su hadoop -c \
	"curl -s http://172.17.42.1:8080/hadoop/spark-1.1.0-bin-2.5.1.tgz | tar zx --exclude=examples/* -C  /opt/"

RUN su hadoop -c "mkdir -p /home/hadoop/tmp/pid"

ENV HADOOP_PREFIX /opt/hadoop-2.5.1
ENV JAVA_HOME /opt/jdk1.7.0_67
ENV PATH $JAVE_HOME/bin:$PATH

ADD etc/hadoop/core-site.xml $HADOOP_PREFIX/etc/hadoop/core-site.xml
ADD etc/hadoop/hdfs-site.xml $HADOOP_PREFIX/etc/hadoop/hdfs-site.xml
ADD etc/hadoop/mapred-site.xml $HADOOP_PREFIX/etc/hadoop/mapred-site.xml
ADD etc/hadoop/yarn-site.xml $HADOOP_PREFIX/etc/hadoop/yarn-site.xml
ADD etc/hadoop/slaves $HADOOP_PREFIX/etc/hadoop/slaves

# 每个版本可能有所不同，仅对其进行修改
RUN sed -i '/export JAVA_HOME/c export JAVA_HOME=/opt/jdk1.7.0_67 \n export HADOOP_PID_DIR=/home/hadoop/tmp/pid' $HADOOP_PREFIX/etc/hadoop/hadoop-env.sh

ENV SPARK_HOME /opt/spark-1.1.0-bin-2.5.1
ADD conf/spark-env.sh $SPARK_HOME/conf/spark-env.sh
ADD etc/hadoop/slaves $SPARK_HOME/conf/slaves

RUN chown hadoop:hadoop $HADOOP_PREFIX/etc/hadoop/* $SPARK_HOME/conf/*

# SSH

RUN su hadoop -c "ssh-keygen -P '' -t rsa -f /home/hadoop/.ssh/id_rsa"
RUN su hadoop -c "cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys"

EXPOSE 22
CMD /usr/sbin/sshd -D

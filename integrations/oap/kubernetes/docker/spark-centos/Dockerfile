# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

FROM centos:centos7.6.1810 as builder

RUN yum install -y \
    wget \
    libdigest-sha-perl \
    bzip2

# Download miniconda 4.5.12, then upgrade it to 4.8.4
RUN wget --quiet --output-document miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-4.5.12-Linux-x86_64.sh \
    && (echo '866ae9dff53ad0874e1d1a60b1ad1ef8  miniconda.sh' | md5sum -c) \
    # Conda must be installed at /opt/home/conda
    && /bin/bash miniconda.sh -b -p /opt/home/conda \
    && rm miniconda.sh \
    && /opt/home/conda/bin/conda install --name base conda=4.8.4

FROM centos:centos7.6.1810

MAINTAINER The Optimized Analyitics Package for Spark Platform (OAP) Authors https://github.com/Intel-bigdata/OAP/

WORKDIR /opt/home

RUN yum install -y \
    curl \
    wget \
    unzip \
    maven \
    git \ 
  && rm -rf /tmp/* /var/tmp/*

#python & conda
COPY --from=builder /opt/home/conda /opt/home/conda

# Source conda.sh for all login shells.
RUN ln -s /opt/home/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh

# Conda recommends using strict channel priority speed up conda operations and reduce package incompatibility problems.
# Set always_yes to avoid needing -y flags, and improve conda experience in Databricks notebooks.
RUN /opt/home/conda/bin/conda config --system --set channel_priority strict \
    && /opt/home/conda/bin/conda config --system --set always_yes True

#java
ENV JAVA_HOME            /opt/home/jdk
ENV PATH                          ${JAVA_HOME}/bin:${PATH}

RUN wget https://enos.itcollege.ee/~jpoial/allalaadimised/jdk8/jdk-8u291-linux-x64.tar.gz  && \
    gunzip jdk-8u291-linux-x64.tar.gz && \
    tar -xf jdk-8u291-linux-x64.tar -C /opt/home && \
    rm jdk-8u291-linux-x64.tar && \
    ln -s /opt/home/jdk1.8.0_291 /opt/home/jdk
    
ARG SPARK_VERSION=3.1.1

ENV SPARK_VERSION                   ${SPARK_VERSION}
ENV SPARK_HOME          /opt/home/spark-${SPARK_VERSION}

#spark download
RUN wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop3.2.tgz && \
    tar -zxvf spark-${SPARK_VERSION}-bin-hadoop3.2.tgz && \
    mv spark-${SPARK_VERSION}-bin-hadoop3.2 spark-${SPARK_VERSION} && \
    rm spark-${SPARK_VERSION}-bin-hadoop3.2.tgz

RUN wget https://github.com/zhixingheyi-tian/Resource/releases/download/Cloud-Tools/tpcds-kit.tar.gz  && \
    mkdir -p /opt/home/tools && \
    tar -zxvf tpcds-kit.tar.gz -C /opt/home/tools && \
    rm tpcds-kit.tar.gz


#run
ARG spark_uid=185

ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /opt/home/tini
RUN chmod +x /opt/home/tini

RUN set -ex && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    rm -rf /var/cache/apt/* && \
    chmod g+w /opt/home && \
    mkdir -p /opt/home/logs && \
    chmod g+w /opt/home/logs

ENV HOME_DIR            /opt/home
ENV SPARK_LOG_DIR   /opt/home/logs
ENV SPARK_CONF_DIR /opt/home/conf

COPY  entrypoint.sh /opt/home/entrypoint.sh
COPY  entrypoint-nop.sh /opt/home/entrypoint-nop.sh

COPY  spark-thrift-server.sh /opt/home/spark-thrift-server.sh
COPY  spark-sql.sh /opt/home/spark-sql.sh
COPY  spark-shell.sh /opt/home/spark-shell.sh
COPY  spark-submit.sh /opt/home/spark-submit.sh

ADD https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.2.0/hadoop-aws-3.2.0.jar  $SPARK_HOME/jars/hadoop-aws-3.2.0.jar
ADD https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.11.375/aws-java-sdk-bundle-1.11.375.jar $SPARK_HOME/jars/aws-java-sdk-bundle-1.11.375.jar
ADD https://github.com/zhixingheyi-tian/Resource/releases/download/Cloud-Tools/spark-sql-perf_2.12-0.5.1-SNAPSHOT.jar  /opt/home/tools/spark-sql-perf_2.12-0.5.1-SNAPSHOT.jar

RUN chmod +r $SPARK_HOME/jars/hadoop-aws-3.2.0.jar
RUN chmod +r $SPARK_HOME/jars/aws-java-sdk-bundle-1.11.375.jar
RUN chmod +r /opt/home/tools/spark-sql-perf_2.12-0.5.1-SNAPSHOT.jar

ENTRYPOINT [ "/opt/home/entrypoint.sh" ]

# Specify the User that the actual main process will run as
USER ${spark_uid}

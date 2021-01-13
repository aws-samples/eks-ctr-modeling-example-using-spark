# Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
# SPDX-License-Identifier: MIT-0
FROM amazoncorretto:8 AS build

RUN yum -y update && yum install -y tar.x86_64 && yum install -y gzip && yum install -y wget

# Install Spark
RUN \
mkdir -p /spark/ && \
curl -fsL -o ./spark-2.4.4-bin-hadoop2.7.tgz https://archive.apache.org/dist/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz

RUN \
tar xfz spark-2.4.4-bin-hadoop2.7.tgz -C /spark/ && \
rm /spark/spark-2.4.4-bin-hadoop2.7/jars/kubernetes-*-4.1.2.jar && \
wget https://repo1.maven.org/maven2/io/fabric8/kubernetes-model-common/4.4.2/kubernetes-model-common-4.4.2.jar -P /spark/spark-2.4.4-bin-hadoop2.7/jars/ && \
wget https://repo1.maven.org/maven2/io/fabric8/kubernetes-client/4.4.2/kubernetes-client-4.4.2.jar -P /spark/spark-2.4.4-bin-hadoop2.7/jars/ && \
wget https://repo1.maven.org/maven2/io/fabric8/kubernetes-model/4.4.2/kubernetes-model-4.4.2.jar -P /spark/spark-2.4.4-bin-hadoop2.7/jars/

FROM amazoncorretto:8-alpine AS spark

ARG spark_home=/spark/spark-2.4.4-bin-hadoop2.7

RUN echo 'http://dl-cdn.alpinelinux.org/alpine/v3.9/main' >> /etc/apk/repositories

RUN set -ex && \
    apk upgrade --no-cache && \
    apk add --no-cache bash tini libc6-compat gcompat linux-pam nss && \
    mkdir -p /opt/spark && \
    mkdir -p /opt/spark/work-dir && \
    touch /opt/spark/RELEASE && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd

COPY --from=build ${spark_home}/jars /opt/spark/jars
COPY --from=build ${spark_home}/bin /opt/spark/bin
COPY --from=build ${spark_home}/sbin /opt/spark/sbin
COPY --from=build ${spark_home}/python /opt/spark/python
COPY --from=build ${spark_home}/kubernetes/dockerfiles/spark/entrypoint.sh /opt/

# Install C libs
RUN apk add make automake gcc g++ subversion

# Install Python
RUN apk add --no-cache python3-dev~=3.6.9 python3~=3.6.9 && \
    python3 -m ensurepip && \
    pip3 install --upgrade pip setuptools && \
    pip3 install numpy pandas && \
    rm -r /usr/lib/python*/ensurepip && \
    if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi && \
    if [[ ! -e /usr/bin/python ]]; then ln -sf /usr/bin/python3 /usr/bin/python; fi && \
    rm -r /root/.cache

FROM spark AS final

ENV SPARK_HOME /opt/spark

WORKDIR /opt/spark/work-dir

ENTRYPOINT [ "/opt/entrypoint.sh" ]
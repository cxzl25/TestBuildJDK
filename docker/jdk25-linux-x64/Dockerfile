#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

# Build JDK 25 on CentOS 7 to ensure glibc 2.17 compatibility.
# This avoids the "version `GLIBC_2.34' not found" error on older systems.

FROM centos:7

ARG JDK_REPO=https://github.com/openjdk/jdk.git
ARG JDK_REF=jdk-25+11
ARG BUILD_VARIANT=server

# Switch to vault mirrors since CentOS 7 is EOL
RUN sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-*.repo && \
    sed -i 's|^#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*.repo

# Install devtoolset-11 for a modern GCC (required for JDK 25)
RUN yum install -y centos-release-scl && \
    sed -i 's|^mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-SCLo-*.repo && \
    sed -i 's|^# *baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-SCLo-*.repo && \
    yum install -y \
      devtoolset-11-gcc \
      devtoolset-11-gcc-c++ \
      devtoolset-11-make \
      autoconf \
      automake \
      zip \
      unzip \
      which \
      git \
      file \
      fontconfig-devel \
      freetype-devel \
      libX11-devel \
      libXext-devel \
      libXrender-devel \
      libXrandr-devel \
      libXtst-devel \
      libXt-devel \
      cups-devel \
      alsa-lib-devel && \
    yum clean all

# Install Boot JDK (Temurin 25 EA) for bootstrapping
RUN curl -fsSL https://api.adoptium.net/v3/binary/latest/25/ea/linux/x64/jdk/hotspot/normal/eclipse \
      -o /tmp/boot-jdk.tar.gz && \
    mkdir -p /opt/boot-jdk && \
    tar -xzf /tmp/boot-jdk.tar.gz -C /opt/boot-jdk --strip-components=1 && \
    rm -f /tmp/boot-jdk.tar.gz

ENV BOOT_JDK=/opt/boot-jdk

WORKDIR /build

# Clone JDK source
RUN if git clone --depth 1 --branch "${JDK_REF}" "${JDK_REPO}" jdk25-source 2>/dev/null; then \
      echo "Cloned branch/tag: ${JDK_REF}"; \
    else \
      echo "Branch/tag not found, fetching commit SHA: ${JDK_REF}"; \
      git clone "${JDK_REPO}" jdk25-source && \
      cd jdk25-source && git checkout "${JDK_REF}"; \
    fi

# Save the commit SHA for use in the output filename
RUN echo "$(cd jdk25-source && git rev-parse --short HEAD)" > /build/commit_sha.txt

# Configure and build inside devtoolset-11
RUN cd jdk25-source && \
    scl enable devtoolset-11 -- bash configure \
      --with-boot-jdk=${BOOT_JDK} \
      --with-jvm-variants=${BUILD_VARIANT} \
      --with-vendor-name="Docker Build (CentOS 7)" \
      --with-vendor-url="https://github.com/cxzl25/TestBuildJDK" \
      --with-version-opt="$(git rev-parse --short HEAD)" \
      --with-native-debug-symbols=none \
      --disable-warnings-as-errors

RUN cd jdk25-source && \
    scl enable devtoolset-11 -- make images

# Package the built JDK as tar.gz with commit id in filename
RUN COMMIT_SHA=$(cat /build/commit_sha.txt) && \
    cd jdk25-source/build && \
    for dir in */; do \
      if [ -d "${dir}images/jdk" ]; then \
        cd "${dir}images" && \
        tar -czf "/build/jdk25-linux-x64-${COMMIT_SHA}.tar.gz" jdk/ && \
        break; \
      fi; \
    done

# Verify the built JDK works
RUN cd jdk25-source/build && \
    for dir in */; do \
      if [ -d "${dir}images/jdk" ]; then \
        "${dir}images/jdk/bin/java" -version; \
        break; \
      fi; \
    done

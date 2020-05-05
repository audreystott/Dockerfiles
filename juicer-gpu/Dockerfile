FROM ubuntu:18.04 as builder
MAINTAINER Audrey Stott <audrey.stott@pawsey.org.au>

ARG LC_ALL="C"
ENV DEBIAN_FRONTEND="noninteractive" \
    LC_ALL="${LC_ALL}"
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    ca-certificates \
    git \
    wget \
    gnupg2 \
    zlib1g-dev && \
    apt-get clean all && \
    apt-get purge && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

##### For alignment and creation of Hi-C file pairs 
    # Burrows-Wheeler Aligner (BWA) 0.7.17
##### For .hic file creation and Juicer tools analysis 
    # Java 1.8 JDK
    # Juicer Tools Jar 1.14.08
##### For peak calling
    # CUDA 10.1 (NVIDIA driver included)
    # JCuda libraries for CUDA 10.1

ARG JAVA_VER="13.0.1" 
ARG JUICER_TOOLS_VER="1.14.08" 
ARG CUDA_VER="10.1" 
ARG JCUDA_VER="10.1.0" 
RUN mkdir -p /tmp/build && \
    cd /tmp/build && \
    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_"${CUDA_VER}".105-1_amd64.deb && \
    mkdir /apps && \
    mv cuda-repo-ubuntu1804_"${CUDA_VER}".105-1_amd64.deb /apps/ && \
    cd /apps && \
    wget https://download.oracle.com/otn-pub/java/jdk/13.0.1+9/cec27d702aa74d5a8630c65ae61e4305/jdk-"${JAVA_VER}"_linux-x64_bin.tar.gz && \
    dpkg -i cuda-repo-ubuntu1804_"${CUDA_VER}".105-1_amd64.deb && \
    apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub && \
    apt-get update && apt-get install -y cuda && \
    wget http://www.jcuda.org/downloads/JCuda-All-"${JCUDA_VER}".zip && \
    rm -f cuda-repo-ubuntu1804_"${CUDA_VER}".105-1_amd64.deb && \
    rm -rf /tmp/build && \
    git clone https://github.com/aidenlab/juicer.git && \
    cd juicer && \
    wget https://s3.amazonaws.com/hicfiles.tc4ga.com/public/juicer/juicer_tools_"${JUICER_TOOLS_VER}".jar && \
    ln -s /apps/juicer/juicer_tools_*.jar /apps/juicer/CPU/common/juicer_tools.jar && \
    ln -s /apps/juicer/juicer_tools_*.jar /apps/juicer/AWS/scripts/juicer_tools.jar && \
    ln -s /apps/juicer/juicer_tools_*.jar /apps/juicer/LSF/scripts/juicer_tools.jar && \
    ln -s /apps/juicer/juicer_tools_*.jar /apps/juicer/PBS/scripts/juicer_tools.jar && \
    ln -s /apps/juicer/juicer_tools_*.jar /apps/juicer/SLURM/scripts/juicer_tools.jar && \
    ln -s /apps/juicer/juicer_tools_*.jar /apps/juicer/UGER/scripts/juicer_tools.jar && \
    mkdir references && mkdir restriction_sites && \
    chmod -R go+rwX .

ARG BWA_VER="0.7.17"
RUN mkdir /tmp/bwa-build && cd /tmp/bwa-build && \
    git clone https://github.com/lh3/bwa.git && \
    cd bwa && git checkout v"${BWA_VER}" && \
    make && cp bwa /usr/local/bin/ && \
    cd / && rm -r /tmp/bwa-build

FROM  ubuntu:18.04 as runner

MAINTAINER Audrey Stott <audrey.stott@pawsey.org.au>

ARG LC_ALL="C"
ENV DEBIAN_FRONTEND="noninteractive" \
    LC_ALL="${LC_ALL}"
RUN apt-get update && apt-get install -y --no-install-recommends \
    default-jre \
    perl \
    && apt-get clean all && \
    apt-get purge && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY --from=builder /usr/local/bin/bwa /usr/local/bin/
COPY --from=builder /apps /apps

WORKDIR /data
CMD ["/bin/bash"]
FROM phusion/baseimage:master

LABEL maintainer="Georgios Sokianos <walkero@gmail.com>"

# Use baseimage-docker's init system.
CMD ["/sbin/my_init"]

ARG VBCC_USER_ID=1000
ARG VBCC_GROUP_ID=1000

RUN apt-get update && apt-get -y install gcc \
    wget \
    curl \
    make \
    mc \
    git \
    xz-utils;

RUN mkdir -p /opt/vbcc/bin; \
    mkdir -p /opt/code; \
    mkdir -p /opt/sdk/morphos;

# Add vbcc user and group
RUN existing_group=$(getent group "${VBCC_GROUP_ID}" | cut -d: -f1); \
    if [[ -n "${existing_group}" ]]; then delgroup "${existing_group}"; fi; \
    existing_user=$(getent passwd "${VBCC_USER_ID}" | cut -d: -f1); \
    if [[ -n "${existing_user}" ]]; then deluser "${existing_user}"; fi; \
    addgroup --gid ${VBCC_GROUP_ID} vbcc; \
    adduser --system --uid ${VBCC_USER_ID} --disabled-password --shell /bin/bash --gid ${VBCC_GROUP_ID} vbcc; \
    sed -i '/^vbcc/s/!/*/' /etc/shadow;

# Install lha 1.14i
RUN curl -fSL "https://launchpad.net/ubuntu/+archive/primary/+files/lha_1.14i-10.3_amd64.deb" -o /tmp/lha.deb; \
    cd /tmp; \
    dpkg -i lha.deb;

# Install vasm
RUN curl -fSL "http://sun.hasenbraten.de/vasm/release/vasm.tar.gz" -o /tmp/vasm.tar.gz; \
    cd /tmp; \
    tar xvfz vasm.tar.gz; \
    cd /tmp/vasm; \
    make CPU=ppc SYNTAX=std; \
    cp ./vasmppc_std ./vobjdump /opt/vbcc/bin; 

ENV VBCC="/opt/vbcc"

# Install vlink
RUN curl -fSL "http://sun.hasenbraten.de/vlink/release/vlink.tar.gz" -o /tmp/vlink.tar.gz; \
    cd /tmp; \
    tar xvfz vlink.tar.gz; \
    cd /tmp/vlink; \
    make; \
    cp ./vlink /opt/vbcc/bin;

# Install vbcc
RUN curl -fSL "http://www.ibaug.de/vbcc/vbcc.tar.gz" -o /tmp/vbcc.tar.gz; \
    cd /tmp; \
    tar xvfz vbcc.tar.gz; \
    cd /tmp/vbcc; \
    mkdir bin; \
    yes '\
' | make TARGET=ppc; \
    make TARGET=ppc bin/vscppc; \
    cp ./bin/vbcc* ./bin/vscppc ./bin/vc ./bin/vprof /opt/vbcc/bin;

RUN curl -fSL "https://server.owl.de/~frank/vbcc/2019-10-04/vbcc_target_ppc-morphos.lha" -o /tmp/vbcc_target_ppc-morphos.lha; \
    curl -fSL "https://server.owl.de/~frank/vbcc/2019-10-04/vbcc_unix_config.tar.gz" -o /tmp/vbcc_unix_config.tar.gz; \
    cd /tmp; \
    lha -xfq2 vbcc_target_ppc-morphos.lha; \
    tar xvfz vbcc_unix_config.tar.gz; \
    mv ./config $VBCC/; \
    mv ./vbcc_target_ppc-morphos/targets $VBCC/;

ENV PATH="$VBCC/bin:$PATH"

# Install MorphOS SDK
RUN curl -fkSL "https://www.morphos-team.net/files/sdk-20200422.lha" -o /tmp/MorphOS-SDK.lha; \
    cd /tmp; \
    lha -xfq2 MorphOS-SDK.lha; \
    cd morphossdk; \
    tar -xJf sdk.pack; \
    cd Development/gg; \
    mv ./include /opt/sdk/morphos/include; \
    mv ./includestd /opt/sdk/morphos/includestd; \
    mv ./lib /opt/sdk/morphos/lib; \
    mv ./os-include /opt/sdk/morphos/os-include; \
    mv ./ppc-morphos /opt/sdk/morphos/ppc-morphos;

ENV MOS_SDK_INC="/opt/sdk/morphos/include"
ENV MOS_SDK_STD="/opt/sdk/morphos/includestd"
ENV MOS_OS_INC="/opt/sdk/morphos/os-include"

# Install MUI 5.0 dev
RUN curl -fSL "http://muidev.de/download/MUI%205.0%20-%20Release/MUI-5.0-2019R4-os4.lha" -o /tmp/MUI-5.0.lha; \
    curl -fSL "http://muidev.de/download/MUI%205.0%20-%20Release/MUI-5.0-2019R4-os4-contrib.lha" -o /tmp/MUI-5.0-contrib.lha; \
    cd /tmp; \
    lha -xfq2 MUI-5.0.lha; \
    lha -xfq2 MUI-5.0-contrib.lha; \
    mv ./SDK/MUI /opt/sdk/MUI_5.0;

ENV MUI50_INC="/opt/sdk/MUI_5.0/C/include"

# Install MCC_GuiGfx
RUN curl -fSL "http://aminet.net/dev/mui/MCC_Guigfx.lha" -o /tmp/MCC_Guigfx.lha; \
    cd /tmp; \
    lha -xfq2 MCC_Guigfx.lha; \
    cp ./MCC_Guigfx/Developer/C/Include/MUI/* /opt/sdk/MUI_5.0/C/include/mui;

# Install AMISSL SDK
RUN curl -fSL "https://github.com/jens-maus/amissl/releases/download/4.5/AmiSSL-4.5.lha" -o /tmp/AmiSSL.lha; \
    cd /tmp; \
    lha -xfq2 AmiSSL.lha; \
    mv ./AmiSSL/Developer /opt/sdk/AmiSSL;

ENV AMISSL_INC="/opt/sdk/AmiSSL/include"

# Install FlexCat
RUN curl -fSL "https://github.com/adtools/flexcat/releases/download/2.18/FlexCat-2.18.lha" -o /tmp/FlexCat.lha; \
    cd /tmp; \
    lha -xfq2 FlexCat.lha; \
    cp ./FlexCat/Linux-i386/flexcat /usr/bin/;

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# USER vbcc
WORKDIR /opt/code
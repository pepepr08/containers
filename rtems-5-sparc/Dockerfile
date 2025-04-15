FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive

# Basic dependencies
RUN apt-get update && apt-get install -y git make

# rtems dependencies
RUN apt-get install -y flex bison texinfo python3-dev g++ unzip libgmp-dev libmpfr-dev bzip2 xz-utils

# Make python3 the default python
RUN ln -s /usr/bin/python3 /usr/bin/python

# cFS files needed for building for LEON3/SPARC
COPY rtems-5-sparc/bsp_rtems_cfg.h /root/bsp_rtems_cfg.h
COPY rtems-5-sparc/psp.diff /root/psp.diff
COPY rtems-5-sparc/toolchain-leon3-rtems5.cmake /root/toolchain-leon3-rtems5.cmake

# setup and compile RTEMS BSP
RUN mkdir -p ${HOME}/rtems-5-sparc \
  && cd ${HOME} \
  && git clone -b 5 https://gitlab.rtems.org/rtems/tools/rtems-source-builder.git \
  && cd rtems-source-builder/rtems \
  && ../source-builder/sb-set-builder --prefix=$HOME/rtems-5-sparc 5/rtems-sparc
# clone and bootstrap RTEMS source tree
RUN cd ${HOME} \
  && git clone -b 5 https://gitlab.rtems.org/rtems/rtos/rtems.git \
  && export PATH=$HOME/rtems-5-sparc/bin:$PATH \
  && cd rtems && ./bootstrap
# Build and install RTEMS LEON3 BSP
RUN cd ${HOME} \
  && export PATH=$HOME/rtems-5-sparc/bin:$PATH \
  && mkdir b-sparc \
  && cd b-sparc \
  && ../rtems/configure --target=sparc-rtems5 \
    --enable-rtemsbsp=leon3 \
    --prefix=${HOME}/rtems-5-sparc \
    --enable-networking \
    --enable-cxx \
    --enable-posix \
    --disable-deprecated \
  && make -j2 \
  && make install

# install git from source for updated version
RUN apt-get install -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext cmake
RUN curl -o git.tar.gz https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.26.2.tar.gz
RUN tar -zxf git.tar.gz && rm git.tar.gz
RUN cd git-* && make prefix=/usr/local && make prefix=/usr/local install

# Manually install qemu-system-sparc version 7.1.0
RUN curl https://download.qemu.org/qemu-7.1.0.tar.xz -O \
  && apt-get update && apt-get install -y ninja-build pkg-config libglib2.0-dev python3-pip python3-setuptools python3-wheel libpixman-1-dev && pip3 install meson \
  && tar xvJf qemu-7.1.0.tar.xz \
  && cd qemu-7.1.0 \
  && ./configure --target-list=sparc-softmmu --enable-debug \
  && make install \
  && cd ../ \
  && rm qemu-7.1.0.tar.xz \
  && rm -r qemu-7.1.0 \
  && qemu-system-sparc --version

# Delete unnecessary directories now that rtems is built
RUN cd ${HOME} \
  && rm -rf rtems-source-builder b-sparc rtems

# Additional dependencies
RUN apt-get install -y -f mtools parted udev bsdmainutils

# Install dosfstools from source release
RUN curl -o dosfstools.tar.gz http://deb.debian.org/debian/pool/main/d/dosfstools/dosfstools_4.2.orig.tar.gz
RUN tar -zxf dosfstools.tar.gz && rm dosfstools.tar.gz
RUN cd dosfstools-* && ./configure && make && make prefix=/usr/local install

# Remove unecessary dependencies now that rtems is built
RUN apt-get purge -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip git flex bison texinfo curl \
  libgmp-dev libmpfr-dev bzip2 xz-utils ninja-build pkg-config 
RUN apt-get autoremove -y

FROM ubuntu:16.04
FROM ubuntu:16.04
MAINTAINER Tofunmi Ajayi <tofunmi@gmx.ch>
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update --fix-missing
RUN apt-get -yq dist-upgrade
RUN apt-get update && apt-get install -y --no-install-recommends apt-utils #should fix 'debconf' problem
RUN apt-get -yq install autoconf build-essential libusb-1.0-0-dev cmake wget pkg-config libboost-all-dev python-dev python-cheetah git subversion python-software-properties

# Dependencies for UHD image downloader script
RUN apt-get -yq install python-mako python-requests

# Fetching the uhd 3.010.001 driver for our USRP SDR card
RUN wget http://files.ettus.com/binaries/uhd/uhd_003.010.001.001-release/uhd-3.10.1.1.tar.gz && tar xvzf uhd-3.10.1.1.tar.gz && cd UHD_3.10.1.1_release && mkdir build && cd build && cmake ../ && make && make install && ldconfig && python /usr/local/lib/uhd/utils/uhd_images_downloader.py

# Dependencies for OpenAirInterface software
RUN apt-get -yq install autoconf  \
        automake  \
        cmake \
        ethtool \
        gdb  \
        git \
        iperf \
        iproute \
        iptables \
        python  

# Fetching the develop repository
RUN git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
RUN cd openairinterface5g && git checkout master

# Compile
WORKDIR /root/openairinterface5g
RUN cd cmake_targets && mkdir -p lte_build_oai/build


RUN echo $WORKDIR
#RUN /bin/bash -c '/root/openairinterface5g/cmake_targets/.build_oai -I'
CMD ["./build_oai -I"]
WORKDIR /root/openairinterface5g/cmake_targets/lte_build_oai




# CmakeLists generation
RUN echo "cmake_minimum_required(VERSION 2.8)"                                          > CMakeLists.txt
RUN echo "set ( CMAKE_BUILD_TYPE \"\" )"                                                        >> CMakeLists.txt
RUN echo "set ( CFLAGS_PROCESSOR_USER \"\" )"                                           >> CMakeLists.txt
#RUN echo "set ( XFORMS \"True\" )"                                                                     >> CMakeLists.txt
RUN echo "set ( RRC_ASN1_VERSION \"Rel14\")"                                            >> CMakeLists.txt
RUN echo "set ( ENABLE_VCD_FIFO \"False\")"                                             >> CMakeLists.txt
RUN echo "set ( RF_BOARD \"OAI_USRP\")"                                                         >> CMakeLists.txt
RUN echo "set ( TRANSP_PRO \"ETHERNET\")"                                                           >> CMakeLists.txt
RUN echo "set(PACKAGE_NAME \"lte-softmodem\")"                                          >> CMakeLists.txt
RUN echo "set (DEADLINE_SCHEDULER \"False\" )"                                          >> CMakeLists.txt
RUN echo "set (CPU_AFFINITY \"False\" )"                                                        >> CMakeLists.txt
RUN echo "set ( T_TRACER \"False\" )"                                                   >> CMakeLists.txt
RUN echo "set (UE_AUTOTEST_TRACE \"False\")"                                            >> CMakeLists.txt
RUN echo "set (UE_DEBUG_TRACE \"False\")"                                               >> CMakeLists.txt
RUN echo "set (UE_TIMING_TRACE \"False\")"                                                      >> CMakeLists.txt
RUN echo "set (DISABLE_LOG_X \"False\")"                                                        >> CMakeLists.txt
RUN echo 'include(${CMAKE_CURRENT_SOURCE_DIR}/../CMakeLists.txt)'   >> CMakeLists.txt

WORKDIR /root/openairinterface5g/cmake_targets/lte_build_oai/build
RUN OPENAIR_HOME=/root/openairinterface5g OPENAIR_DIR=$OPENAIR_HOME OPENAIR1_DIR=$OPENAIR_HOME/openair1 OPENAIR2_DIR=$OPENAIR_HOME/openair2 OPENAIR3_DIR=$OPENAIR_HOME/openair3 OPENAIR_TARGETS=$OPENAIR_HOME/targets cmake ../
RUN make -j`nproc` lte-softmodem
RUN make -j`nproc` oai_usrpdevif
RUN make -j`nproc` params_libconfig

#RUN ln -sf liboai_usrpdevif.so liboai_device.so

# Run directly the eNodeB code
ENTRYPOINT ["/root/openairinterface5g/cmake_targets/lte_build_oai/build/lte-softmodem", "-O", "enb.conf"]

ENV DEBIAN_FRONTEND teletype #possible defconf fix



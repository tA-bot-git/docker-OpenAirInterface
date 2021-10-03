FROM ubuntu:16.04
MAINTAINER Jesutofunmi Ajayi <tofunmi@gmx.ch>
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update
RUN apt-get -yq dist-upgrade
# Dependencies for the UHD driver for the USRP hardware
RUN apt-get -yq install apt-utils autoconf build-essential libusb-1.0-0-dev cmake wget pkg-config libboost-all-dev python3 python3-dev python-cheetah git subversion python3-software-properties

# Dependencies for UHD image downloader script
RUN apt-get -yq install python-mako python-requests python-pip
RUN pip install requests

# Fetching the uhd 3.013.001.000 driver for our USRP SDR card
RUN pwd
RUN wget https://files.ettus.com/binaries/uhd/uhd_003.010.001.001-release/uhd-3.10.1.1.tar.gz
RUN tar xvzf uhd-3.10.1.1.tar.gz 
RUN cd UHD_3.10.1.1_release
RUN pwd
RUN ls
RUN pwd
RUN mkdir build 
RUN cd build 
WORKDIR /UHD_3.10.1.1_release/build/
RUN ls
RUN pwd
RUN cmake ../ 
RUN make 
RUN make install 
RUN ldconfig 
RUN python /usr/local/lib/uhd/utils/uhd_images_downloader.py

RUN mv /sbin/dhclient /usr/sbin/dhclient

# Dependencies for OpenAirInterface software
RUN apt update
RUN apt-get -yq install git vim-common iproute2 iputils-ping nano sudo net-tools psmisc lsb-release libcap-dev libnet1-dev libnl-3-dev dh-autoreconf libgps-dev liborc-0.4-dev libudev-dev python-mako cmake libgps-dev liborc-0.4-dev libudev-dev python-mako libgps-dev liborc-0.4-dev libudev-dev python-mako dh-autoreconf libcap-dev libnet1-dev libnl-3-dev libconfig8-dev libconfig-dev isc-dhcp-server isc-dhcp-client	

# ASN1 compiler with Eurecom fixes
RUN pwd
WORKDIR /root
RUN git clone https://gitlab.eurecom.fr/oai/asn1c.git
RUN cd asn1c && ./configure && make -j`nproc` && make install

# Fetching the develop repository
RUN git clone https://gitlab.eurecom.fr/TofunmiA/oai5g_new.git
RUN ls /root/oai5g_new
RUN echo "ls -l `which sh`"
RUN sudo ln -s bash /bin/sh.bash
RUN sudo mv /bin/sh.bash /bin/sh
RUN ls -l `which sh`

WORKDIR /root/oai5g_new/
RUN . ./oaienv #Compiles

#Compile
WORKDIR /root/oai5g_new/
RUN cd cmake_targets #&& mkdir -p lte_build_oai/build/
RUN ls /root/oai5g_new/cmake_targets/
WORKDIR /root/oai5g_new/cmake_targets/
RUN sed -i '34 c\source /root/oai5g_new/cmake_targets/tools/build_helper' build_oai
WORKDIR /root/oai5g_new/cmake_targets/tools/
RUN sed -i '214 c\tar -xzvf protobuf-cpp-3.3.0.tar.gz --owner root --group $(groups | cut -d" " -f1) --no-same-owner' build_helper
WORKDIR /root/oai5g_new/cmake_targets/lte_build_oai/build/
RUN sed -i '2 c\# For build in directory: /root/oai5g_new/cmake_targets/lte_build_oai/build' CMakeCache.txt
RUN sed -i '383 c\OpenAirInterface_BINARY_DIR:STATIC=/root/oai5g_new/cmake_targets/lte_build_oai/build' CMakeCache.txt
RUN sed -i '386 c\OpenAirInterface_SOURCE_DIR:STATIC=/root/oai5g_new/cmake_targets/lte_build_oai' CMakeCache.txt
RUN sed -i '439 c\Project_BINARY_DIR:STATIC=/root/oai5g_new/cmake_targets/lte_build_oai/build' CMakeCache.txt
RUN sed -i '442 c\Project_SOURCE_DIR:STATIC=/root/oai5g_new/cmake_targets/lte_build_oai' CMakeCache.txt
RUN sed -i '657 c\CMAKE_CACHEFILE_DIR:INTERNAL=/root/oai5g_new/cmake_targets/lte_build_oai/build' CMakeCache.txt
RUN sed -i '722 c\CMAKE_HOME_DIRECTORY:INTERNAL=/root/oai5g_new/cmake_targets/lte_build_oai' CMakeCache.txt

WORKDIR /root/oai5g_new/cmake_targets/
RUN . ./build_oai -I
WORKDIR /root/oai5g_new/cmake_targets/
RUN . ./build_oai --eNB -w USRP -t ETHERNET
RUN ls /root/oai5g_new/cmake_targets/lte_build_oai/
WORKDIR /root/oai5g_new/cmake_targets/lte_build_oai

# CmakeLists generation
RUN echo "cmake_minimum_required(VERSION 2.8)"                                          > CMakeLists.txt
RUN echo "set ( CMAKE_BUILD_TYPE \"\" )"                                                        >> CMakeLists.txt
RUN echo "set ( CFLAGS_PROCESSOR_USER \"\" )"                                           >> CMakeLists.txt
RUN echo "set ( RRC_ASN1_VERSION \"Rel14\")"                                            >> CMakeLists.txt
RUN echo "set ( ENABLE_VCD_FIFO \"False\")"                                             >> CMakeLists.txt
RUN echo "set ( RF_BOARD \"None\")"                                                         >> CMakeLists.txt
RUN echo "set ( TRANSP_PRO \"ETHERNET\")"                                                           >> CMakeLists.txt
RUN echo "set(PACKAGE_NAME \"lte-softmodem\")"                                          >> CMakeLists.txt
RUN echo "set (DEADLINE_SCHEDULER \"False\" )"                                          >> CMakeLists.txt
RUN echo "set (CPU_AFFINITY \"False\" )"                                                        >> CMakeLists.txt
RUN echo "set ( T_TRACER \"False\" )"                                                   >> CMakeLists.txt
RUN echo "set (UE_AUTOTEST_TRACE \"False\")"                                            >> CMakeLists.txt
RUN echo "set (UE_DEBUG_TRACE \"False\")"                                               >> CMakeLists.txt
RUN echo "set (UE_TIMING_TRACE \"False\")"                                                      >> CMakeLists.txt
RUN echo "set (DISABLE_LOG_X \"False\")"                                                        >> CMakeLists.txt
#RUN echo 'include(${CMAKE_CURRENT_SOURCE_DIR}/CMakeLists.txt)'   >> CMakeLists.txt

WORKDIR /root/oai5g_new/cmake_targets/lte_build_oai/build
RUN OPENAIR_HOME=/root/oai5g_new OPENAIR_DIR=$OPENAIR_HOME OPENAIR1_DIR=$OPENAIR_HOME/openair1 OPENAIR2_DIR=$OPENAIR_HOME/openair2 OPENAIR3_DIR=$OPENAIR_HOME/openair3 OPENAIR_TARGETS=$OPENAIR_HOME/targets cmake ../

WORKDIR /root/oai5g_new/targets/PROJECTS/GENERIC-LTE-EPC/CONF/
#eNB Configuration details
RUN sed -i '184 c\        ENB_INTERFACE_NAME_FOR_S1_MME            = "eth0";' enb.band7.tm1.50PRB.usrpb210.conf
RUN sed -i '186 c\        ENB_INTERFACE_NAME_FOR_S1U            = "eth0";' enb.band7.tm1.50PRB.usrpb210.conf

#RCC Configuration details
RUN sed -i '144 c\    mme_ip_address      = ( { ipv4       = "130.92.70.161";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '158 c\        ENB_INTERFACE_NAME_FOR_S1_MME            = "eth0";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '159 c\        ENB_IPV4_ADDRESS_FOR_S1_MME              = "130.92.65.69/24";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '160 c\        ENB_INTERFACE_NAME_FOR_S1U               = "eth0";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '161 c\        ENB_IPV4_ADDRESS_FOR_S1U                 = "130.92.65.69/24";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '162 c\        ENB_PORT_FOR_S1U                         = 2152; # Spec 2152' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '164 c\        ENB_IPV4_ADDRESS_FOR_X2C                 = "130.92.65.69/24";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '165 c\        ENB_PORT_FOR_X2C                         = 36422; # Spec 36422' rcc.band7.tm1.if4p5.50PRB.conf

RUN sed -i '187 c\      local_if_name  = "eth1";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '188 c\     	remote_address = "10.0.5.4";' rcc.band7.tm1.if4p5.50PRB.conf
RUN sed -i '189 c\    	local_address  = "10.0.5.2";' rcc.band7.tm1.if4p5.50PRB.conf



WORKDIR /root/oai5g_new/cmake_targets/lte_build_oai/build
RUN make -j`nproc` lte-softmodem
WORKDIR /root/oai5g_new/cmake_targets/lte_build_oai/build
RUN ls

RUN pwd

RUN ln -sf liboai_usrpdevif.so liboai_device.so

EXPOSE 2152/udp 36412/udp 36422/udp

RUN ifconfig

# Run directly the eNodeB code
RUN ls /root/oai5g_new/targets/PROJECTS/GENERIC-LTE-EPC/CONF/

RUN pwd

RUN ls

#Set up container to DNS
#RUN dhclient eth0

WORKDIR /root/oai5g_new/cmake_targets/lte_build_oai/build/

#ENTRYPOINT ["/root/oai5g_new/cmake_targets/lte_build_oai/build/lte-softmodem", "-O", "/root/oai5g_new/targets/PROJECTS/GENERIC-LTE-EPC/CONF/enb.band7.tm1.50PRB.usrpb210.conf"]
ENTRYPOINT ["/root/oai5g_new/cmake_targets/lte_build_oai/build/lte-softmodem", "-O", "/root/oai5g_new/targets/PROJECTS/GENERIC-LTE-EPC/CONF/rcc.band7.tm1.if4p5.50PRB.conf"]

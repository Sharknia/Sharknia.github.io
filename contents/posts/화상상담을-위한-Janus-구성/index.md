---
title: "화상상담을 위한 Janus 구성"
description: "화상상담을 위한 Janus 구성"
date: 2023-01-03
update: 2023-08-20
tags:
  - BE
---
<aside>
👨‍💻 Janus를 우분투에 설치하면서 사용한 명령어 정리
Stun 서버와 Turn 서버를 위한 Coturn Server도 함께 설치
</aside>

## 환경구성

- Ubuntu 18.04 bionic
- Python 3.7

## 설치 명령어 정리

1. 파이썬 및 기본 소프트웨어 설치
2. Janus 설치
    
    [How to install Janus Gateway in Ubuntu Server 18.04](https://ourcodeworld.com/articles/read/1197/how-to-install-janus-gateway-in-ubuntu-server-18-04)
    
    - Janus 설치
        
        ```bash
        packagelist=( 
        git 
        libmicrohttpd-dev 
        libjansson-dev 
        libssl-dev 
        libsrtp-dev 
        libsofia-sip-ua-dev 
        libglib2.0-dev 
        libopus-dev 
        libogg-dev 
        libcurl4-openssl-dev 
        liblua5.3-dev 
        libconfig-dev 
        pkg-config 
        gengetopt 
        libtool 
        automake 
        gtk-doc-tools 
        cmake 
        ) 
        apt-get install ${packagelist[@]}
        ```
        
    - libnice 설치
        
        *※ 최소 파이썬 3.7을 요구한다.* 
        
        ```bash
        pip3 install meson==0.61.5 
        ln -s /usr/local/bin/meson /usr/bin/ 
        wget https://github.com/ninja-build/ninja/releases/download/v1.10.1/ninja-linux.zip 
        unzip ninja-linux.zip 
        cp ninja /usr/bin/ 
        git clone https://gitlab.freedesktop.org/libnice/libnice.git
        cd libnice 
        meson --prefix=/usr build 
        ninja -C build 
        ninja -C build install
        ```
        
    - libstrp 설치
        
        ```bash
        wget https://github.com/cisco/libsrtp/archive/v2.2.0.tar.gz 
        tar xfv v2.2.0.tar.gz 
        cd libsrtp-2.2.0 
        ./configure —prefix=/usr —enable-openssl 
        make shared_library && make install
        ```
        
    - usrctp 설치
        
        ```bash
        git clone https://github.com/sctplab/usrsctp 
        cd usrsctp 
        ./bootstrap 
        ./configure --prefix=/usr && make && make install
        ```
        
    - libwebsockets 설치
        
        ```bash
        git clone https://github.com/warmcat/libwebsockets.git 
        cd libwebsockets 
        mkdir build 
        cd build 
        cmake -DLWS_MAX_SMP=1 -LWS_IPV6=ON -DCMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_C_FLAGS="-fpic" .. 
        make && make install
        ```
        
    - mqtt 설치
        
        ```bash
        git clone https://github.com/eclipse/paho.mqtt.c.git 
        cd paho.mqtt.c 
        prefix=/usr make install
        ```
        
    - NanoMSG 설치
        
        ```bash
        apt-get install libnanomsg-dev -y
        ```
        
    - RabbitMQ C AMQP 설치
        
        ```bash
        git clone https://github.com/alanxz/rabbitmq-c 
        cd rabbitmq-c 
        git submodule init 
        git submodule update 
        mkdir build && cd build 
        cmake -DCMAKE_INSTALL_PREFIX=/usr ..
        make && make install
        ```
        
    - janus 컴파일링
        
        ```bash
        git clone https://github.com/meetecho/janus-gateway.git
        cd janus-gateway
        sh autogen.sh
        ./configure —prefix=/opt/janus
        make && make install
        make configs
        ```
        
3. Janus 설정
    
    설정 파일 위치 : /opt/janus/etc/janus
    
    - janus.jcfg
        
        ```bash
        - log_to_file : 주석해제 
        - admin_secret : 변경 "PW" 
        - rtp_port_range : 주석해제 및 변경 "20000-60000" 
        - stun_server : 변경 "Stun Server Domain" 
        - stun_port : 주석해제 
        - nat_1_1_mapping : 변경 "Server IP" 
        - turn_server : 변경 "Turn Server Domain" 
        - turn_port : 주석해제 
        - turn_type : 주석해제 
        - turn_user : 주석해제 및 변경 "Turn ID" 
        - turn_pwd : 주석해제 및 변경 " Turn PW" 
        - ipv6 : 주석해제 및 변경 "true"
        ```
        
    - janus.plugin.videoroom.jcfg
        
        ```bash
        general{ 
        admin_key : 주석해제 및 변경 "비밀번호" 
        publishers : 생성 값 "10" 
        }
        ```
        
    - janus.transport.http.jcfg
        
        ```bash
        https : 값 변경 "true" 
        secure_port : 주석 해제 
        admin_https : 주석처리 값 변경 "true" 
        cert_pem : 값 변경 "crt.pem 위치" 
        cert_key : 값 변경 "key.pem 위치" 
        cert_pwd : 값 변경 "인증서 비밀번호"
        ```
        
4. Coturn 설치
    
    [Corturn](http://john-home.iptime.org:8085/xe/index.php?mid=board_sKSz42&document_srl=1546)
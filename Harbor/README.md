## Harbor Installation and Configuartion Guid

### Table of Contents
1. [Offline installer](#1)
2. [Prerequisites for the target host](#2)
3. [Installation Steps](#3)
     * 3.1. [Download the installer ](#3.1)
     * 3.2. [인증서를 생성](#3.2)
     * 3.3. [Configure harbor.cfg](#3.3)
     * 3.4. [Run install.sh to install and start Harbor;  ](#3.4)


# <div id='1'/> 1. Offline installer
Harbor 설치 방법은 online 과 offline 으로 나뉜다.<br>
 - online installer  : installer가 Docker hub를 다운로드하기 때문에 installer 크기는 매우 작다.
 - offline installer : 호스트가 인터넷에 연결되어 있지 않을 때 이 방법을 사용한다. 설치 프로그램에는 미리 빌드 된 이미지가 포함되어 크기가 크다.
 
현재 설치된 Harbor Release 버전은 v1.6.2를 기준으로 가이드를 작성하였다.<br>
 
# <div id='2'/> 2. Prerequisites for the target host

Harbor는 여러 Docker 컨테이너로 배포되므로 Docker를 지원하는 모든 Linux 배포에 배포 할 수 있습니다. 대상 호스트는 Python, Docker 및 Docker Compose가 설치되어 있어야합니다.

### Hardware
<table>
  <tr>
    <th>Resource</th>
    <th>Capacity</th>
    <th>Descrpition</th>
  </tr>
  <tr>
    <td>CPU</td>
    <td>minimal 2 CPU</td>
    <td>4 CPU is preferred</td>
  </tr>
  <tr>
    <td>Mem</td>
    <td>minimal 4GB</td>
    <td>8GB is preferred</td>
  </tr>
  <tr>
    <td>Disk</td>
    <td>minimal 40GB</td>
    <td>160GB is preferred</td>
  </tr>
</table>

### Software
<table>
  <tr>
    <th>Software</th>
    <th>Version</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Python</td>
    <td>version 2.7 or higher</td>
    <td>기본적으로 파이썬 인터프리터가 설치되지 않은 리눅스 배포판 (Gentoo, Arch)에 파이썬을 설치해야 할 수도 있습니다</td>
  </tr>
  <tr>
    <td>Docker engine</td>
    <td>version 1.10 or higher</td>
    <td>설치 지침은 다음을 참조하십시오. https://docs.docker.com/engine/installation/</td>
  </tr>
  <tr>
    <td>Docker Compose</td>
    <td>version 1.6.0 or higher</td>
    <td>설치 지침은 다음을 참조하십시오. https://docs.docker.com/compose/install/</td>
  </tr>
  <tr>
    <td>Openssl</td>
    <td>latest is preferred</td>
    <td>Harbor에 대한 인증서 및 키 생성</td>
  </tr>
</table>

### Network ports
<table>
  <tr>
    <th>Port</th>
    <th>Protocol</th>
    <th>Descrpition</th>
  </tr>
  <tr>
    <td>443</td>
    <td>HTTPS</td>
    <td>Harbor UI 및 API는이 포트에서 https 프로토콜에 대한 요청을 수락합니다.</td>
  </tr>
  <tr>
    <td>4443</td>
    <td>HTTPS</td>
    <td>Harbord에 대한 Docker Content Trust 서비스 연결, Notary가 활성화 된 경우에만 필요.</td>
  </tr>
  <tr>
    <td>80</td>
    <td>HTTP</td>
    <td>Harbor UI 및 API는이 포트에서 http 프로토콜에 대한 요청을 수락합니다.</td>
  </tr>
</table>

# <div id='3'/> 3. Installation Steps

설치 단계는 다음과 같이 요약됩니다.

SSL 키의 경로. 프로토콜이 https로 설정된 경우에만 적용됩니다.
 
 # <div id='3.1'/>1. Download the installer 
 
 -  offline installer 바이너리 파일([release](https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.2.tgz))을 다운로드 한다.
    
 -  패키지를 추출하려면 tar 명령을 사용하십시오.
      
        $ tar xvf harbor-offline-installer-1.6.2.tgz
 
 # <div id='3.2'/> 2. https를 사용 할 경우 인증서를 생성한다.(optional) 
  
 - cert 디렉토리 생성
        
          $ mkdir ~/cert
          $ cd ~/cert
   
 - Authority 인증서 생성
        
         $ openssl genrsa -out ca.key 4096
         $ openssl req -x509 -new -nodes -sha512 -days 3650 \
               -subj "/C=KR/ST=Seoul/L=Seoul/O=crossent/OU=paasxpert/CN={yourdomain}" \
               -key ca.key \
               -out ca.crt
             
 - Server 인증서 생성
   
   - Create your own Private Key:
          
            $ openssl genrsa -out yourdomain.com.key 4096
      
   - Generate a Certificate Signing Request:
         
            $ openssl req -sha512 -new \
                -subj "/C=KR/ST=Seoul/L=Seoul/O=crossent/OU=paasxpert/CN={yourdomain}" \
                -key yourdomain.com.key \
                -out yourdomain.com.csr 
                
  - 구성 및 설치
   
    - Harbor에 대한 서버 인증서 및 키 구성
       
            $ cp yourdomain.com.crt /data/cert/
            $ cp yourdomain.com.key /data/cert/ 

    - Docker에 대한 서버 인증서, 키 및 CA 구성
            
            $ openssl x509 -inform PEM -in yourdomain.com.crt -out yourdomain.com.cert
      
    - Docker 용 yourdomain.com.cert, yourdomain.com.key 및 ca.crt를 배포합니다.
      
            $ cp yourdomain.com.cert /etc/docker/certs.d/yourdomain.com/
            $ cp yourdomain.com.key /etc/docker/certs.d/yourdomain.com/
            $ cp ca.crt /etc/docker/certs.d/yourdomain.com/
      
   - 다음은 생성된 인증서 구성을 보여줍니다.
            
            /etc/docker/certs.d/
                └── yourdomain.com:port   
                   ├── yourdomain.com.cert  <-- Server certificate signed by CA
                   ├── yourdomain.com.key   <-- Server key signed by CA
                   └── ca.crt               <-- Certificate authority that signed the registry certificate
 
 # <div id='3.3'/> 3. Configure harbor.cfg      

 - harbor.cfg 파일을 편집하고 호스트 이름과 프로토콜을 업데이트하고 속성 ssl_cert 및 ssl_cert_key를 업데이트합니다.
     
              #set hostname
              hostname = yourdomain.com:port
              #set ui_url_protocol
              ui_url_protocol = https
              ......
              #The path of cert and key files for nginx, they are applied only the protocol is set to https 
              ssl_cert = /data/cert/yourdomain.com.crt
              ssl_cert_key = /data/cert/yourdomain.com.key  
   
 - <b>hostname</b> : UI 및 레지스트리 서비스에 액세스하는 데 사용되는 대상 호스트의 호스트 이름입니다. 대상 컴퓨터의 IP 주소 또는 FQDN (정규화 된 도메인 이름)이어야합니다 (예 : 192.168.1.10 또는 reg.yourdomain.com). 호스트 이름에 localhost 또는 127.0.0.1을 사용하지 마십시오. 레지스트리 서비스를 외부 클라이언트가 액세스 할 수 있어야합니다.
 - <b>ui_url_protocol</b> : 
(http 또는 https, 기본값은 http) UI 및 토큰 / 알림 서비스에 액세스하는 데 사용되는 프로토콜입니다. 인증서를 사용하는 경우 매개 변수는 https 여야합니다. 기본적으로는 http입니다. https 프로토콜을 설정하려면 [Configuring Harbor with HTTPS Access](https://github.com/goharbor/harbor/blob/master/docs/configure_https.md)을 참조하십시오.
   - <b>ssl_cert</b> : SSL 인증서의 경로. 프로토콜이 https로 설정된 경우에만 적용됩니다.
   - <b>ssl_cert_key</b> : SSL 키의 경로. 프로토콜이 https로 설정된 경우에만 적용됩니다.
  
  
 # <div id='3.4'/> 4. Run install.sh to install and start Harbor;  
 - Generate configuration files for Harbor:
    
             $ ./prepare
   
 - If Harbor is already running, stop and remove the existing instance. Your image data remain in the file system
     
             $ docker-compose down -v
   
 - Finally, restart Harbor:
       
             $ docker-compose up -d
         
             $ docker login yourdomain.com